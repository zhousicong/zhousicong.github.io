---
title: Work Environment setup
description: 用于工作环境的搭建
categories:
  - 博客
abbrlink: 27b637df
---
### Mac App
- InsomniaX for Mac (Disable the sleep mode on your Mac)
- Karabiner-Elements (改建利器)
- Alfred
- Spectacle (分屏软件)
- dozer (系统栏图标隐藏工具)
- Itsycal (日历)
- Hammerspoon
- PicGo
- kubectx

### vim.conf
```
" 语法高亮
syntax enable
syntax on
" 编码设置
set encoding=utf-8
" 显示匹配
set showmatch
" 显示标尺
set ruler


set backspace=indent,eol,start

call plug#begin('~/.vim/plugged')
Plug 'Yggdroot/indentLine'
Plug 'ayu-theme/ayu-vim'
call plug#end()


" indentLine {{
let g:indentLine_char = '┆'
" }}
" ayu-vim {{
set termguicolors
let ayucolor="dark"
colorscheme ayu
" }}
```
