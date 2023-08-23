---
author: Romain Mencattini
pubDatetime: 2023-08-23T16:20:00Z
title: My journey to cmp lsp luasnip
postSlug: my-journey-to-cmp-lsp-luasnip
featured: false
draft: false
tags:
  - neovim
  - ide
  - cmp
  - lsp
  - luasnip
ogImage: ""
description:
    My journey to add completion on neovim with all lsp and snippet together. 
---

# Introduction

While rediscovering Neovim, I am trying to make it look like Visual Studio Code. It is a bad idea...\
Yes but as a poor little developer I need my comfort while coding.

So what was my objective:
> Having function as well as snippet completion while coding.

Something like this in Visual Studio Code:


![visual studio code completion menu](../../my-journey-to-cmp-lsp-snippet/visual-studio-code-cmp.png)

I will provide a more or less minimal configuration (tested ✅) to get this in Neovim.

# Plugins

All the plugins which will be used. Note that I use `lazy.vim` as package manager, so the config will reflect it
but, the following list is agnostic of the technology used.

* neovim/nvim-lspconfig (1)
* williamboman/mason.nvim (~1)
* williamboman/mason-lspconfig.nvim (1)
* hrsh7th/nvim-cmp (2)
* saadparwaiz1/cmp_luasnip (2)
* hrsh7th/cmp-nvim-lsp (2)
* hrsh7th/cmp-nvim-lsp-signature-help (2)
* L3MON4D3/LuaSnip (3) 
* rafamadriz/friendly-snippets (3)

This can be breakdown to three categories:

1. lsp
1. completion
1. snippet

The lsps will provide and configure the server for a given language.\
Completion is everything related to the popup menu while coding.\
Snippet is just an engine and a set of snippets which can be used.

# Configuration

Now let's check how everything is configured to work together.

## LSP

We will use `mason` and `mason-lspconfig` to get lsp servers and install them automatically when starting Neovim (
in this example it is lua lsp):

```lua
require("mason").setup()
require("mason-lspconfig").setup({
    ensure_installed = {
        'lua_ls',      -- To configure neovim
    },
    automatic_installation = true
})
```

Then we create function than will be run when the lsp is attached to a buffer.
It is somehow a hook, and you can do what you want inside: log stuff or in this case use it to define
custom bindings:

```lua
local on_attach = (function(_, bufnr)
    -- Set the completion shortcut + go back
    local opts = { buffer = bufnr }

    -- Define your own keymaps
    vim.keymap.set("n", "gd", function() vim.lsp.buf.definition() end, opts)
    vim.keymap.set("n", "K", function() vim.lsp.buf.hover() end, opts)
end)
```

Then create the capabilities, i.e. what can be displayed in the completion menu:
```lua
local capabilities = vim.lsp.protocol.make_client_capabilities()
capabilities = require('cmp_nvim_lsp').default_capabilities(capabilities)
```

At the end we configure the lsp:

```lua
-- Lua
require('lspconfig').lua_ls.setup({ on_attach = on_attach, capabilities = capabilities })

```

## Cmp

At this point if you open your Neovim nothing fancy will appear due to missing configuration. Indeed, we need all
the `cmp-` plugins to be configured in order to get some nice display.

This small code will do the work:

```lua
cmp.setup({
    mapping = {
        -- Define your completion keybindings
        ['<CR>'] = cmp.mapping.confirm({ select = false }),
    },
    sources = cmp.config.sources({
        { name = "nvim_lsp",               keyword_length = 1 },
        { name = "nvim_lsp_signature_help" },
    })
})
```
A nice completion menu with the stuff coming from LSP server.
But the snippets are still missing.

## Snippets

We need to configure `LuaSnip` like this:

```lua
require('luasnip.loaders.from_vscode').lazy_load()
luasnip.config.setup({})
```
Then adding it to the completion menu:

```lua
cmp.setup({
    snippet = {
        expand = function(args)
            luasnip.lsp_expand(args.body)
        end,
    },
    -- ...
    sources = cmp.config.sources({
        { name = "nvim_lsp",               keyword_length = 1 },
        { name = "nvim_lsp_signature_help" },
        { name = "luasnip" }
    })
```

And voilà, we have everything working.

## Complete solution

Here is a complete working example with `lazy.vim`

```lua
local lazypath = vim.fn.stdpath 'data' .. '/lazy/lazy.nvim'
if not vim.loop.fs_stat(lazypath) then
  vim.fn.system {
    'git',
    'clone',
    '--filter=blob:none',
    'https://github.com/folke/lazy.nvim.git',
    '--branch=stable', -- latest stable release
    lazypath,
  }
end
vim.opt.rtp:prepend(lazypath)

-- NOTE: Here is where you install your plugins.
--  You can configure plugins using the `config` key.
--
--  You can also configure plugins after the setup call,
--    as they will be available in your neovim runtime.
require('lazy').setup( {
    {
        "neovim/nvim-lspconfig",
        dependencies = {
            {
                'williamboman/mason.nvim',
                build = function()
                    pcall(vim.api.nvim_command, 'MasonUpdate')
                end,
            },
            'williamboman/mason-lspconfig.nvim', -- Lspconfig with mason

            -- Useful status updates for LSP
            { 'j-hui/fidget.nvim', tag = 'legacy' },

            -- Additional lua configuration, makes nvim stuff amazing!
            "folke/neodev.nvim",
        }
    },

    {
        -- Autocompletion
        'hrsh7th/nvim-cmp',
        dependencies = {
            -- Snippet Engine & its associated nvim-cmp source
            'L3MON4D3/LuaSnip',
            'saadparwaiz1/cmp_luasnip',

            -- Adds LSP completion capabilities
            'hrsh7th/cmp-nvim-lsp',
            'hrsh7th/cmp-nvim-lsp-signature-help',

            -- Adds a number of user-friendly snippets
            'rafamadriz/friendly-snippets',
        },
    },
})

local util = require('lspconfig/util')
local luasnip = require("luasnip")
local cmp = require('cmp')

require("mason").setup()
require("mason-lspconfig").setup({
    ensure_installed = {
        'lua_ls',      -- To configure neovim
    },
    automatic_installation = true
})

local on_attach = (function(_, bufnr)
    -- Set the completion shortcut + go back
    local opts = { buffer = bufnr }

    vim.keymap.set("n", "gd", function() vim.lsp.buf.definition() end, opts)
    vim.keymap.set("n", "K", function() vim.lsp.buf.hover() end, opts)
end)

local capabilities = vim.lsp.protocol.make_client_capabilities()
capabilities = require('cmp_nvim_lsp').default_capabilities(capabilities)

-- Lua
require('lspconfig').lua_ls.setup({ on_attach = on_attach, capabilities = capabilities })

require('luasnip.loaders.from_vscode').lazy_load()
luasnip.config.setup({})

cmp.setup({
    snippet = {
        expand = function(args)
            luasnip.lsp_expand(args.body)
        end,
    },
    mapping = {
        -- `Enter` key to confirm completion
        ['<CR>'] = cmp.mapping.confirm({ select = false }),
      },
    sources = cmp.config.sources({
        { name = "nvim_lsp",               keyword_length = 1 },
        { name = "nvim_lsp_signature_help" },
        { name = "luasnip" }
    })
  }
)
```

Here is a quick demo, _with a bit more of configuration_ :

<video width="fit-content" controls>
  <source src="../../my-journey-to-cmp-lsp-snippet/nvim-cmp.mp4" type="video/mp4">
</video>
