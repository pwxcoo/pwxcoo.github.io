---
title: 【Advanced Java】缓存
date: 2019-06-03 19:31:46
categories: base
tags:
- advanced-java
- cache
---


## 缓存优缺点

优点：

1. 高性能
2. 高并发

缺点，因为缓存可能出现的问题：

1. 缓存和数据库双写不一致
2. 缓存雪崩，缓存穿透
3. 缓存并发竞争