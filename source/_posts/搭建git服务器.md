---
title: 搭建git服务器
date: 2018-09-24 11:05:54
categories: memo
tags:
- git
- linux
---

因为一些原因要搭一个 git 服务器，因为 linux 下文件管理权限的问题，在 ssh 这里踩了一点坑。

## Steps0: add user

```bash
$ sudo adduser git
$ su git
$ cd
$ mkdir .ssh && chmod 700 .ssh
$ touch .ssh/authorized_keys && chmod 600 .ssh/authorized_keys
```

## Step1: add public key

把公钥传到服务器，然后加到 authorized_keys。

```bash
$ cat /tmp/id_rsa.pwxcoo.pub >> ~/.ssh/authorized_keys
```

多个公钥用换行符 separate。

## Step2: initialize repository

```bash
$ cd /srv/git
$ sudo git init --bare sample.git
```

然后再自己的电脑上

```bash
$ git clone git@gitserver:/srv/git/project.git
```

到这里差不多基本的功能都 ok 了。

## Step3: authority management

然后要设置一下 git 用户的权限，不能让 git 用户用 ssh 登录 shell，让 git 用户默认用 git-shell 登录。

打开 `/etc/passwd`，修改 git 用户的默认 shell。

```
git:x:1001:1001:,,,:/home/git:/usr/bin/git-shell
```

## Problems

### ssh failed?

我用 ssh 的时候，一直失败。。然后 `ssh -Tvvv git@gitserver` 命令调试的时候也没有找到有用的信息。。

后来找到了这个 log：

```log
debug3: receive packet: type 51
```

server 对 ssh 验证的时候，response 了 type 51 的 packet，然后发现是 linux 下文件权限的问题，server 端一直读不到 authorized_keys 。。


```
$ sudo chown yourusername:yourusername /home/yourusername/ -R
$ sudo chmod o-rwx /home/yourusername/ -R
```

### stricter authority management?

权限管理这里有人做过了工具，可以去找一下，我之后可能要用到。



## References

- [搭建Git服务器](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/00137583770360579bc4b458f044ce7afed3df579123eca000)
- [A PRIVATE GIT SERVER CRASH COURSE](https://hackaday.com/2018/06/27/keep-it-close-a-private-git-server-crash-course/)
- [Git on the Server - Setting Up the Server](https://git-scm.com/book/en/v2/Git-on-the-Server-Setting-Up-the-Server)
- [Git Push Error: insufficient permission for adding an object to repository database](https://stackoverflow.com/questions/6448242/git-push-error-insufficient-permission-for-adding-an-object-to-repository-datab)
- [SSH-Key authentication fails](https://superuser.com/questions/1137438/ssh-key-authentication-fails)