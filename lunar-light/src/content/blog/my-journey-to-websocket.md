---
author: Romain Mencattini 
pubDatetime: 2023-08-05T15:22:00Z
title: My journey to Websocket
postSlug: my-journey-to-Websocket
featured: true
draft: false
tags:
  - Websocket
  - backend
  - frontend
  - spring-boot
  - vuejs
  - keycloak
  - authentication
ogImage: ""
description:
    My journey creating a project with backend and frontend interacting each other over Websocket with full
    authentication provided by keycloak.
---

> For those who can't wait here is the link to technical solution part: [Technical solution](#technical-solution)\
> and [the Github link to the POC](https://github.com/rmencattini/websocket-example-spring-boot-vuejs)

## Backstory

A bit of context: my company is providing an application on premise to different banks. It is composed of a backend and frontend apps.
It relies heavily on REST calls but due to the nature of the data (clients, portfolios, positions, risk metrics, etc.) it can be pretty slow to be computed.
Moreover, some features such as refresh or notifications are currently handle by REST calls.

We thought it would be worth investigating some other communication channel between the apps to tackle these problems. One of the solutions was Websockets.
The backend app is a beast, and it requires minutes to start, so the development cycle is pretty slow. After discussion with colleagues we ended up creating a
minimal POC project to:

1. Test the technology
2. Have quick feedback loop

## Tech stack

As tech stack, I mimicked the one we used, modulo small changes, in order to have a good carry over effect. It means:

* `Java` with `Spring-boot` framework.
* `VueJs`
* `Keycloak` for `OAuth2` provider

Most of the tutorials I found did not correspond to my requirements:

* The frontend was directly into the spring-boot application (in the `resources/` folder) and it was on the same URL.
* No Keycloak authentication
* No mix between Websocket authentication and REST authentication

## High-level solution

As `Stomp` is a high-level protocol and most resources focusing on it, it will be our main tool.
Here is a workflow chart on which we will rely:

![uml diagram workflow](../../my-journey-to-websocket-uml-diagram.jpg)


The frontend authenticates to `Keycloak` and receives a `Jwt` token.\
The token is used in different way to authenticate the caller (either for REST or Websocket).

Then Client does a connection request to the Server (aka. backend), once he got the acknowledgement, he can subscribe to some channels.

When Client sends a message, the Websocket channel forwards it to all subscribers. Backend run a hook before redirecting message to the proper controller.
The hook verifies the token validity by calling the authentication server. Once the token is validated, it sends the message to the controller.

In our example, the controller gets the message and sends another message to the Websocket channel which ultimatly is forwaded to Client 
(due to his previous subscription)


## Technical solution

The solution explained as plain words
> The Websocket is configured so that each time a message arrives, it goes through an interceptor which will:
> 
> * verify the message contains authorization token in its headers 
> * check the validity of token
> * decode the token
> * set the authenticated user to the context of _current message_ 

Here is the Websocket configuration:
```java
@Configuration
@EnableWebSocketMessageBroker // 1)
@Order(Ordered.HIGHEST_PRECEDENCE + 99) // 2)
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {
    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        config.enableSimpleBroker("/topic", "/alert"); // 3)
        config.setApplicationDestinationPrefixes("/app"); // 4)
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry
            .addEndpoint("/Websocket") // 5)
            .setAllowedOriginPatterns("*"); // 6)
    }


    @Override
    public void configureClientInboundChannel(ChannelRegistration registration) {
        registration.interceptors(new Interceptor()); // 7)
    }

    public class Interceptor implements ChannelInterceptor {

        @Override
        public Message<?> preSend(org.springframework.messaging.Message<?> message, // 8)
                MessageChannel channel) {
            StompHeaderAccessor accessor = MessageHeaderAccessor.getAccessor(message, StompHeaderAccessor.class);
            if (StompCommand.SEND == accessor.getCommand() || StompCommand.SEND == accessor.getCommand()) { // 9)
                JwtDecoder jwtDecoder = jwtDecoder(); // 10)
                String authorizationToken = accessor.getFirstNativeHeader("Authorization");
                if (authorizationToken != null) {
                    Jwt jwt = jwtDecoder.decode(token);
                    Principal principal = User.builder().build(); // 11)
                    accessor.setUser(principal);
                } else {
                    throw new OAuth2AuthenticationException(
                            new OAuth2Error("invalid_token", "Missing access token", null));
                }
            }
            return message;
        }
    }


```
1. The annotation to mark this configuration for the Websocket.
1. The order needs to be higher than spring security, otherwise the hooks won't work.
1. Create an in-memory message brook with some destinations for message exchanging.
2. Set a prefix for Spring to determine which message should be rooted to `@MessageMapping`.
3. The stomp endpoint clients will connect to
4. As we have different url/port, we need to allow some origin to avoid CORS issue.
5. Add an interceptor to channel message.
6. The interceptor will work on incoming channel message before dispatch it to controllers.
7. Check which type of message (could be `CONNECT`, `HEARTBEAT`,...)
8. Init a jwt decoder (implementation depends on your authentication server)
9. Once you will have decoded the jwt, you can build a `User` object with all the property you want. The `User` class should implement the `Principal` interface.
10. Set for the context *of the current message*, the authenticated user.

## Outro

The technical implementation can be found here: [the Github link to the POC](https://github.com/rmencattini/websocket-example-spring-boot-vuejs)\
It contains a `README` more focused on the implementation as well as the instruction to run the project.
