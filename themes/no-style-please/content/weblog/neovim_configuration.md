---
title: "Neovim Configuration Mk. II"
date: 2026-03-31
tags: [linux, arch, vim, config]
draft: true
description: ""
---

I have been using a `vim`-like editor for a few years now.
My fingers understand what they have to do and mouse-based text editors feel _SO_ clunky and slow.
I enjoy ricing in-so-far-as it improves the ergonomics of my system, but I am not really into the graphical sugar.
I like text.
I want things to be simple.
I don't need fancy animations and transparency.
I frequently adopt the default configurations for many of the applications I use.
This means that I used a default install of `vim` for most of the last few years.
It does the job and the defaults are sensical.
A few minor changes to indentation behaviour was enough for me.
However, I was always interested in the features that `neovim` provides.
Specifically, it felt like a better integration with `LSP`s might be useful for my workflow.
I write a lot of `rust` code, and `rust-analyzer` is a great tool.
I have found that working directly with the `LSP` in my editor drastically improves my iteration time.

Anyways, I think this post will serve as a personal reference for my configuration.
Its helpful to have something to document the configuration as it grows.

I started my `neovim` adventure with [kickstart](https://github.com/nvim-lua/kickstart.nvim).
This was a great way to avoid the giant headache that is bootstrapping a configuration.
Again, it comes with sensible defaults and those work fine for most of my problems.
With a few small tweaks, I have been running with this for a while now.

Lazy Plugin Manager:
- blink.cmp
- Comment.nvim
- conform.nvim
- fidget.nvim
- gitsigns.nvim
- guess-indent.nvim 
- lazy.nvim
- lazydev.nvim 
- LuaSnip
- mini.nvim
- nvim-lspconfig 
- nvim-treesitter
- plenary.nvim 
- telescope-fzf-native.nvim
- telescope-ui-select.nvim
- telescope.nvim
- todo-comments.nvim
- tokyonight.nvim
- which-key.nvim
- mason-lspconfig.nvim
- mason-tool-installer.nvim 
- mason.nvim
  - \*`black` (python) 
  - \*`rust-analyzer` (rust)
- \*oil.nvim
- \*typst-preview.nvim
- \*vimtex
- \*markdown-preview.nvim

As I have added more functionality, my limited understanding of the Lua configuration is beginning to show.
I have marked the plugins I've added with a \*.
I run into small problems that I am not sure how to fix.
More and more, I am avoiding touching my configuration because it is a bit of a mess.
Even when I do get something working, I am often not certain why or how.
This makes it challenging to debug problems down the line.
For example, I work with `vimtex` for editing `Latex` files locally.
I really like the hot reloading of the compiled document.
Its a must-have for me when writing `Latex`.
I also want to have auto-complete for citation keys read from the included `.bib` files.
I can't seem to get this working...

I would like to start fresh with my configuration.
I will bring over the things that I use often, and scrap the stuff I don't.
I want my new setup to be enjoyable to edit.

```
nvim
├── init.lua
└── lua
    ├── ben
    │   ├── init.lua
    │   ├── init_lazy.lua
    │   ├── lazy
    │   │   └── init.lua
    │   ├── remap.lua
    │   └── set.lua
    └── init.lua
```

## Options

## Plugins

First I setup the `Lazy` plugin manager.

