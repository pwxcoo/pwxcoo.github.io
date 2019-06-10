---
title: Manage a Series of SSH Keys
date: 2018-07-15 22:18:47
categories: record
tags:
- ssh
- git
---

因为 `ssh` 用起来太爽了，很多的时候生成了一堆 `ssh` 私钥，如何在一台设备上管理多个 `ssh` 私钥。

起因是想在公司电脑上管理两个 git account。一个公司的，一个自己的，然后配置了半天。。。主要还是因为 CSDN 那几篇博客有点坑 (说 config 里 Host 是可以随便设置，却没提到 Host 是 ssh 开连接找路由的入口。。) 。。每次都 authentication 失败。。不过后来在 Stackoverflow 上找到原因了。。感觉 zz。

## Preparation
删除 global config 设置:(不过好像不删也没什么关系？因为讲道理 local config 是可以覆盖 global config 的)
```bash
$ git config --global --unset user.name
$ git config --global --unset user.email
```

然后给每个 repository 设置自己的 local config:
```bash
git config user.email "example@email.com"
git config user.name "your_name"
```

## Generation
生成 `ssh`
```bash
$ ssh-keygen -t rsa -C "example@email.com"
```

会提示要不要覆盖原来的 `id_rsa`，当然是不要，重新开一个名字，比如 `id_rsa_github` 这样子。

然后在对应的 git server 上加上自己的公钥。

## Configuration
在 `~/.ssh/` 目录下新建一个 config 文件，然后写配置文件。

很多博客没提到 Host 是路由，而是说 HostName 要写上目标网站，于是我一开始就以为 ssh 是通过 HostName 找 server 的。。其实 Host 是 ssh 找连接的路由，两个都是 ssh 连接时要用到的。

我的配置文件: 
```config
Host github.com
HostName github.com
User git
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa_github

Host vs-ssh.visualstudio.com
HostName vs-ssh.visualstudio.com
User t-xiwu
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa
```

## Debug
用 `ssh` 连一下就知道有没有成功了
```bash
$ ssh -T git@github.com
```

具体信息可以加个 `v` 的选项就可以了，就是 `verbose` ，可以用来调试，看看哪里出错了。
```bash
$ ssh -Tv git@github.com
```

更具体的可以用 `vvv`，更详细的信息。
```bash
$ ssh -vvv git@github.com
```

我最后就是这样子，然后带着错误信息在 Stackoverflow 找到了 Solution。


