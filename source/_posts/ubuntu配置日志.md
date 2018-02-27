---
title: ubuntu配置日志
date: 2018-01-04 18:22:00
categories: 备忘
tags: 
- linux
---


## 安装
用u盘重新一遍 ubuntu 。安装了 ubuntu16.04 版本的。

## firefox
- `sudo apt-get update`
- `sudo apt-get install firefox`

## chrome
- 添加软件源 `sudo wget http://www.linuxidc.com/files/repo/google-chrome.list -P /etc/apt/sources.list.d/`
- 导入谷歌软件公钥 `wget -q -O - https://dl.google.com/linux/linux_signing_key.pub  | sudo apt-key add -`
- `sudo apt-get update`
- `/usr/bin/google-chrome-stable`

## vim
- `sudo apt-get install vim`
- `sudo vim /etc/vim/vimrc`
- 取消`syntax on`的注释
- 最后一行配置：

```
set nu          //在左侧显示行号码
set tabstop=4   //tab 长度设为4
set nobackup    //覆盖文件不备份
set cursorline  //突出显示当前行
set ruler       //在右下角显示光标位置打状态行
set autoindent  //自动缩进
```

## git
- `sudo apt-get install git`
- `git config --global user.name "your name"`
- `git config --globa user.email "your email.com"`
- `ssh-keygen -t rsa -b 4096 -C "your_email@example.com"`
- 复制公钥到github上


## vscode
- `sudo add-apt-repository ppa:ubuntu-desktop/ubuntu-make`
- `sudo apt-get update`
- `sudo apt-get install ubuntu-make`
- `sudo umake web visual-studio-code`

## anaconda(python)
- 官网下载
- `sudo bash Anaconda3-5.0.1-Linux-x86_64.sh` 然后选yes
- 添加环境路径
- `sudo chmod 777 -R anaconda3/` 给权限
- jupyter的皮肤配置: jupyterthemes配置 `jt -t gruvboxd -f ubuntu -fs 12 -tfs 12 -ofs 11`, 我发现ubuntu的字体已经比window好太多，所以就没配置。

## java(服务器端下载jdk8)
一件很奇怪的事情，就是不知道为什么我用apt的ppa源下载，一直连不上oracle官网。。一直404失败。所以就徒手配置了。
- `apt-get -u dist-upgrade`强制升级一下版本
- `sudo apt-get remove --purge openjdk* `伤处openjdk
- `sudo mkdir /usr/local/java` 创建安装目录 `cd /usr/local/java` 进入安装目录
- 因为服务器连不上oracle官网。。所以我用了rz把本地的jdk传上去了。。
- `tar -zxvf jdk-8u161-linux-x64.tar.gz` 解压
- `vim ~/.bashrc` 环境变量配置
```
export JAVA_HOME=/usr/local/java/jdk1.8.0_161
export JRE_HOME={JAVA_HOME}/jre
export CLASSPATH=.:{JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=$JAVA_HOME/bin:$PATH
```
- `source ~/.bashrc` 重新加载环境变量生效


## node
- `sudo apt install nodejs-legacy nodejs` 直接下载
- `sudo apt install npm` 安装npm
- `sudo npm install -g n` 下载node版本控制库
- `sudo n stable` 升级到最新的LTS


## codeblocks
- `sudo add-apt-repository ppa:damien-moore/codeblocks-stable` 添加ppa源
- `sudo apt-get update` 更新
- `sudo apt-get install codeblocks` 下载安装


## coursera 添加hosts解析
- `ping coursera.org` 得到ip
- 浏览器里看network，未正常响应的都添加到hosts里。
- `vim /etc/hosts` 修改
- `sudo /etc/init.d/networking restart`重启网络服务


## 截图软件 / shutter
- `apt install shutter` 

## 电脑主题
- `sudo  apt-get  install unity-tweak-tool` 下载 unity-tweak-tool
- `sudo add-apt-repository ppa:noobslab/themes` 添加源
- `sudo apt-get update` 更新
- `sudo apt-get install flatabulous-theme` 下载 flatabulous-themes
- `sudo apt-get install ultra-flat-icons` 下载 ultra-flat的图标

## zsh终端
- `sudo apt-get install zsh` 下载
- `chsh -s $(which zsh)` 设置
- 重启计算机生效