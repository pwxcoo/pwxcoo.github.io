---
title: a collection of common operations in vim
date: 2018-12-14 16:44:25
categories: memo
tags:
- memo
- linux
- vim
---

自己使用 Vim 中的一些常见问题。

我的 vimrc: [pwxcoo/vimrc](https://github.com/pwxcoo/vimrc)



## Update vim on ubuntu

[How can I get a newer version of Vim on Ubuntu](https://vi.stackexchange.com/questions/10817/how-can-i-get-a-newer-version-of-vim-on-ubuntu)

- `sudo add-apt-repository ppa:jonathonf/vim`
- `sudo apt update`
- `sudo apt install vim`


## Use System Clipboard

- `sudo apt install vim-gnome`
- `vim --version | grep "clipboard"` 查看 clipboard 前面是否是 `+`

然后

- `"+y` 复制到系统剪切板
- `"+p` 粘贴

## replace one by one

[Vim -> 边确认边查找替换](https://blog.csdn.net/feelang/article/details/38408875)

- `/{which}` 开始查找
- `cw {what} [ESC]` 删除并替换
- `n` 下一个单词
- `.` 重复上一个操作 (删除并替换) 

## operate multiple rows

块操作

- `C-v` 开始块操作
- `C-d` 向下移动
- 操作
    - `shift + i` 插入
    - `J` 连结所有行 (Join) 
    - `>` / `<` 左右缩进
    - `=` 自动缩进
- `[ESC]` 生效

## Plugins keyboard shortcut

### NERDTree

- `shift + i` 显示隐藏文件
- `m` 一系列操作
    - `a` 创造新文件

### Tabbar

- `:bd` 关闭 buffer  (**Recommend**)
- `:bw` 关闭并不保存

### NERDCommenter

- `<leader> + c + <space>` 快速注释