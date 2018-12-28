---
title: linux notes
date: 2018-12-28 16:30:58
categories: memo
tags:
- memo
- linux
---

记录常用的 linux 操作。

## server 配置 ssh service

- `useradd -m [username]`, `-m` 表示自动建立用户的登陆目录
- `passwd [username]`, 配置密码
- `usermod -s /bin/bash [username]`, 更改默认的 login shell
    - `cat /etc/passwd` 查看所有用户
    - `cat /etc/shells` 查看所有 shell
- 回到 client，`scp id_rsa.pub [username]@[server address]:/home/[username]` 传输公钥到 server
- 登陆回 server，`mv id_rsa.pub .ssh/authorized_keys` 就好了