---
title: "WSL Clipboard Support in Neovim"
date: 2023-02-12T18:51:03-06:00
draft: false
toc: false
categories:
  - neovim
tags:
  - Neovim
---

Windows Subsystem for Linux allows one to run a Linux environment within Windows, without a separate virtual machine. For people who are require to use Windows machines, WSL can be a godsend.

One problem for neovim users who want to use WSL, there is no clipboard support. Most people on Reddit or Stack Overflow recommend win32yank.exe.

There is another way that does not require downloading a third party binary. if you have access to powershell, you can use clip.exe to copy and a powershell script to paste.

The first example is written in Lua for neovim:

``` lua
if vim.fn.has("wsl") == 1 then
  vim.g.clipboard = {
    name = "WslClipboard",
    copy = {
      ["+"] = "clip.exe",
      ["*"] = "clip.exe",
    },
    paste = {
      ["+"] = 'powershell.exe -c [Console]::Out.Write($(Get-Clipboard -Raw).tostring().replace("`r", ""))',
      ["*"] = 'powershell.exe -c [Console]::Out.Write($(Get-Clipboard -Raw).tostring().replace("`r", ""))',
    },
    cache_enabled = 1,
  }
end
```

for viml:

``` vim
if has('wsl')
  let g:clipboard = {
    \   'name': 'wslclipboard',
    \   'copy': {
    \      '+': 'clip.exe',
    \      '*': 'clip.exe',
    \    },
    \   'paste': {
    \      '+': 'powershell.exe -c [Console]::Out.Write($(Get-Clipboard -Raw).tostring().replace("`r", ""))',
    \      '*': 'powershell.exe -c [Console]::Out.Write($(Get-Clipboard -Raw).tostring().replace("`r", ""))',
    \   },
    \   'cache_enabled': 1,
  }
endif
```
