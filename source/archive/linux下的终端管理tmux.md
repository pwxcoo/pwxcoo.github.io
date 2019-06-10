---
title: linux下的终端管理tmux
date: 2018-11-15 23:00:01
categories: memo
tags:
- memo
- linux
- tmux
---

linux 下的一个终端管理工具 —— tmux 。(Windows 下也有一个很方便的终端管理工具 —— Console Emulator)

## Introduction

一个 tmux 对应多个 session，一个 tmux 对应多个 window，一个 window 对应多个 window pane。

## Installation

```sh
sudo apt install tmux
```

## Commands

tmux 命令都是由 prefix key + command key 触发的，即先按下 prefix key，然后进入命令模式，然后按 command key。跟 vim 里的 `:` 异曲同工。默认的 prefix key 为 `Ctrl + b`。


### 操作窗格

- `%` 左右分割
- `"` 上下分割
- `<arrow key>` 窗口导航
- `Ctrl + <arrow key>` 调整大小
- `z` 放大/缩小(zoom)

## 操作窗口

- `c` 新建(create)
- `p` 切换到上一个窗口(pre)
- `n` 切换到下一个窗口(next)
- `<number>` 切换到指定的窗口
- `,` 重命名当前窗口

## 操作会话

- `d` 回到 bash(detach)
- `tmux ls` 查看当前 tmux sessions
- `tmux attach -t <number>` 连接到指定的 session
- `tmux new -s {name}` 创建会话
- `tmux rename-session -t <number> {name}` 重命名
- `tmux attach -t {name}` 连接到指定的 session


## References

- [Ubuntu下tmux的安装和使用](https://blog.csdn.net/yucicheung/article/details/80058338)
