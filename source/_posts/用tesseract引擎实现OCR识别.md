---
title: 用tesseract引擎实现OCR识别
date: 2018-05-07 21:38:17
categories: tech
tags:
- ocr
- javascript
---

因为需要一个 OCR 图像识别功能，网上所提供的大部分 API 都是收费的，看到一个 Google 的一个开源项目 [tesseract](https://github.com/tesseract-ocr/tesseract)，很强大。不过是一个开源项目，没有现成的 API 接口可以直接用，干脆就自己直接上手配置撸了。

部署到了 [https://www.pwxcoo.com/tesseract](https://www.pwxcoo.com/tesseract) 上。

## 使用 tesseract
这里有一个地方比较鸡掰，就是我本地机器是 windows，服务器是 linux，所以存在一些地方有些不同。

### 下载
从 github 上下载对应的版本。可以选择源代码版本或者编译完成可执行文件。我当然是选择可执行文件。[github上下载地址](https://github.com/tesseract-ocr/tesseract/wiki)。(噢对，github上下载release文件都是要科学上网的)。

### 环境变量（仅windows)
ubuntu 用 apt 直接会配置好环境变量的。

1. PATH 增加  tesseract 安装目录
2. 新增系统变量 TESSDATA_PREFIX='安装目录文件夹下\tessdata'

### 下载 tessdata
要用 tesseract 就要用数据集训练。训练方法的网上有很多，我没训练过 (我看大多是用 LSTM 做训练)。我是直接下载别人训练好的训练集。

我下了 [tesseract-ocr/tessdata_best](https://github.com/tesseract-ocr/tessdata_best) 的训练后模型 (这个名字说这个训练模型是最好的。。我就信了)，放到 TESSDATA_PREFIX 环境变量路径下就好了。

### 在 node 的 runtime 里调用
在 node 的 runtime 里调用 tesseract，这个已经有人写好了。

[desmondmorris/node-tesseract](https://github.com/desmondmorris/node-tesseract)。用这个库和官方的 demo 就用 node 跑 tesseract 了。

### 部署到自己的 node + koa 应用上
主要需要两个功能，需要 OCR 识别本地图片和网络图片。

除了 [desmondmorris/node-tesseract](https://github.com/desmondmorris/node-tesseract) 这个库，还用到了 koa-multer（上传图片），superagent（下载网络图片）这两个库。

具体可以看我 ["[ADD]添加OCR识别功能" 这次 commit](https://github.com/pwxcoo/BlackMagic/commit/31500a2adaa9b6015e56480050e4d2ca5f5cff24)。

实现后的结果 [https://www.pwxcoo.com/tesseract](https://www.pwxcoo.com/tesseract)。

## 参考资料
- [https://www.pwxcoo.com/tesseract](https://www.pwxcoo.com/tesseract)。实现结果。
- [tesseract](https://github.com/tesseract-ocr/tesseract) Google 的 tesseract 开源项目
- [desmondmorris/node-tesseract](https://github.com/desmondmorris/node-tesseract) A simple wrapper for the Tesseract OCR package
- [Node-tesseract OCR Windows使用小结](https://blog.csdn.net/zm_zhl/article/details/78582831)
- [tesseract-ocr/tessdata_best](https://github.com/tesseract-ocr/tessdata_best) tessdata 训练后模型
- [tesseract-ocr](https://github.com/tesseract-ocr/) tesseract-ocr 的 project 社区
- [基于tesseract-orc的koa2 OCR Web小应用](https://blog.csdn.net/u013810234/article/details/78508311) 