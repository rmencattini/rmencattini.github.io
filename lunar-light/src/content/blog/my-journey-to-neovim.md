---
author: Romain Mencattini 
pubDatetime: 2023-08-10T16:20:00Z
title: My journey to Neovim
postSlug: my-journey-to-neovim
featured: false
draft: false
tags:
  - neovim
  - ide
ogImage: ""
description:
    My journey to (re)discover (Neo)vim. My approach and my config.
---

> Thank you Bram Moolenarr for all what you did for the software community\
> RIP 


## Backstory

I had a "_not so long but still_" story with (Neo)Vim. But first:
> For the purpose of clarity and simplicity, I will just refer to Neovim all along even if I used vim at some point.

I looked back my secret Github student account and found trace of Neovim config back to March 2016. This runs up to January
2017. Then I started some kind of cycle:
* moving to Sublime Text
* moving to Intellij (for the end of my studies and beginning of my professional career basically due to Java/Kotlin)
* moving to Vscode (for frontend and because my computer was not able to handle Idea + Docker + JVM)
* mixing Vscode and Intellij (for frontend and backend because I upgraded my setup)

and then here we are.
But how did I decide to give another try to Neovim ?

## ThePrimeagen

I did not know him before a few weeks ago (with article date as reference). I just get randomly one of
his video recommended to me (thx Youtube).

It should be this one (it does not matter at all):

[![IMAGE ALT TEXT HERE](https://img.youtube.com/vi/U16RnpV48KQ/0.jpg)](https://www.youtube.com/watch?v=U16RnpV48KQ)

Then I like his style and started watching a few videos I ended up hearing him talking about Vim/Vim motion/Neovim/Lsp.

The way he presented vim motions hits me and I decided to give a try to Vim plugin in Vscode/Intellij. Then I quickly decided 
to test again Neovim.

I need to confess I was always impressed by the Vim guru and I thought, naively:
> I may be a better programmer with Vim as IDE

How wrong I was...

So I give a try, for the bad reason, but I was surprise how easy it was to configure Neovim/Lsp/shortcuts and so on... with
one exception we will come back.

I enjoyed the time I spent to configure as I got quick results and quick wins.


## How it is going

So once I got my Neovim configured, I tried my best to code exclusively through it. The few muscle memory I had from previous
experiences came quickly back and I felt Vim motions becoming more and easier and natural.

With the peak when I used for an afternoon Vscode.\
Damn it felt **soooooo slow**, it was like coding with a 1s ping and I realized I had no pleasure to code compared to Neovim.
(even with the Vim motion plugins)

## LSP

I think without the technology I would have not kept Neovim. It feels so simple once you got `mason`, `mason-lsp` and 
`lsp-zero` adding lsp support is just two lines:

```lua
lsp.ensure_installed({
    'rust_analyzer',
    -- ....
})

require('lspconfig').rust.setup({})
```

_Such a beauty_.

It tooks me 30s to add `Astro` support when I started this blog.

LSP is such a complete lightweight technology:
* auto-complete
* go to definition
* documentation on hover
* ...

(_cf [Microsoft documentation](https://microsoft.github.io/language-server-protocol/)_)

For such no performance/resources overhead. That is a huge win.

Moreover, due to my curiosity, when I test new languages and new technologies, I need a quick support without having to
dig into documentation.


## Configuration

I think it is important to address two things.

At first the initial configuration.
It certainly takes some time the very first time to configure it. But if you are motivated enough to go through it (or 
you simply use an existing distro such as [NvChad](https://nvchad.com/) or [Lazyvim](https://www.lazyvim.org/)) you can quickly
end up to something working.

But then here come the real deal. Once you have something working you may want to continuously improve it, change it, etc.
You ended up spending more time configuring your IDE rather than coding.
I am this type of person: constantly think I could get something better.

To get ride of this, or at least with Neovim configuration, I just follow these two simple rules:
* Don't fix what is not broken
* Just add what you need **when** you need it

So after the initial config writing, I never spent more than a few minutes every week to add the thing that really **bothers** 
me when coding:
* no easy folding ? I just wait a few days to see if it is really necessary
* need Vuejs LSP ? I add it just to make it work and don't spend hours configuring it to get Eslint formatting as long it is
not required

## Jdtls

I spent a good amount of my professional time coding in Java. So I needed Java lsp.
I don't want to write technical stuff in this section because I just copied some working configuration over the web and
never read it.

This section is just to highlight it was a big pain and I want to set it for the record.


## Outro

As most of reasonable people on the web I would say that Neovim is not for everyone. It may even not fit for me.
I don't know what I will be using in a few weeks/months/years but at this point I'm going to continue my journey with Neovim
as I enjoy coding with it and, **that may be naive**, it is the most important thing.

I cannot be good if I don't like what I am doing.
I cannot be curious if my job is painful.

Neovim is currently making my job fun and pain-free.


> [My Neovim config repo link](https://github.com/rmencattini/nvim)











