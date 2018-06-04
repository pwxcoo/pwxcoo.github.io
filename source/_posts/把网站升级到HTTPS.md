---
title: 把网站升级到HTTPS
date: 2018-05-03 20:46:02
categories: 备忘
tags:
- 备忘
---

## 起因

之前 [pwxcoo.com](http://pwxcoo.com) 这个玩具网站一直是 HTTP 的，虽然说全世界都在提倡 HTTPS，但是感觉这个玩具网站没有一定要 HTTPS 的理由。。索性也没理了。

因为突然遇到一个鸡掰的问题，就是有些东西移动端无法处理，我需要在服务器这边处理一些数据，然后再传给移动端。

然后这个移动端是微信。。然后呢，微信这边处理的时序图是这样的，微信的给开发者一个接口，然后微信把数据给开发者服务器，开发者服务器处理好后，再给微信服务器，然后微信服务器再传给微信，遇到的问题就是微信这边要求开发者这边提供的接口要是 HTTPS 的。。所以我没得选。。

于是正好升级成 HTTPS 好了。

## Certbot 升级

我找到一个很傻瓜的解决方案。用的话肯定是 LET'S ENCRYPT，然后我的服务器这边是 nginx + ubuntu16.04。用的是 [Certbot](https://github.com/certbot/certbot) ，github 上 letsencrypt star 最多的一个 repository。

Certbot 真的很舒服，不用去 Let ’ s Encrypt 注册账号（它会自动帮你注册），不用手动修改配置服务器配置，一行命令搞定。

```
$ sudo apt update
$ sudo apt install software-properties-common
$ sudo add-apt-repository ppa:certbot/certbot
$ sudo apt update
$ sudo apt install python-certbot-nginx 
```

安装好之后

```
$ sudo certbot --nginx
```

然后就好了，证书和私钥位置在

```
/etc/letsencrypt/live/www.example.com/privkey.pem
```

升级完成~

![https](https://i.loli.net/2018/05/03/5aeb0c44e371a.png)

要注意看接下来他问你的几个问题，比如说要不要把原来的 HTTP 重定向到 HTTPS 阿什么的，不看也没关系，到时候重新改一下 nginx 的配置文件就好了。证书是 90 天的，运行

```
$ sudo certbot renew --dry-run
```

Certbot 会帮你启动一个定时任务，在证书过期时自动更新。

PS：云服务器上的安全组规则里记得把 443 端口开了。

## 图床
因为网站之前用了七牛云的图床，然后因为七牛云免费图床是 HTTP 的，所以在 Chrome 里虽然是 HTTPS，但是没有那个绿色的小锁。

我搞了半天，他的 HTTPS 外链的图床是要收费的(?)，而且我也没找到升级 HTTPS 图床的入口(?)。。

于是换了一个图床 [SM.MS](https://sm.ms/)。因为网站不大，几张图片，人肉迁徙过去了，然后开了日志文件用来记录迁徙记录，要是以后这个图床倒闭，到时候有这个日志文件起码不会太混乱。。

![migrate](https://i.loli.net/2018/05/07/5af057358135a.png)

## 通配符匹配域名
上面这样是匹配单独域名，如果想要 `*.pwxcoo.com` 这样通配符匹配域名的话，需要另外一种设置。

好像也是 lets encrypt 去年刚出的功能。配置流程稍微复杂一点。

具体操作如下可以看这篇博文。[获取 Let's Encrypt 免费通配符证书实现Https](https://www.cnblogs.com/stulzq/p/8628163.html)。

完成之后在 nginx 里可以设置一下 `pwxcoo.com` 301 跳转到 `www.pwxcoo.com`。因为 `*.pwxcoo.com` 的设置不包括 `pwxcoo.com`。


## 参考资料
- [用 Certbot 一键升级你的网站为 Https](https://www.v2ex.com/t/383032)
- [Certbot](https://github.com/certbot/certbot)
- [SM.MS图床](https://sm.ms/)
- [pwxcoo.com](https://www.pwxcoo.com/)
- [获取 Let's Encrypt 免费通配符证书实现Https](https://www.cnblogs.com/stulzq/p/8628163.html)