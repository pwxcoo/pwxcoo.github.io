---
title: 浅谈Redis和缓存
date: 2018-09-06 12:00:54
categories: base
tags:
- redis
- cache
---

## 缓存的淘汰策略

- FIFO(Fisrt In First Out)
- LRU(Least Recently Used)

## 缓存位置

- 浏览器
- ISP
- 反向代理
- 服务器本地缓存
- 分布式缓存
- 数据库缓存

## CDN

CDN(Content distribution network)。利用更靠近用户的服务器从而更快更可靠地将内容分发给用户。

> For that CDN will stores the data in multiple servers so once a data request arises from a visitor the CDN will take the data from the nearest server located near to the visitor and display it to the visitor.

## 缓存穿透

对不存在的数据进行请求，该请求会穿透缓存到达数据库。

- 为不存在的数据缓存一个空数据
- 对这类请求过滤

## 缓存雪崩

数据没 load 到缓存中，或者缓存在同一个时间大面积失效，或者缓存服务器宕机，导致大量请求都到达数据库。

- 观察用户行为，合理设置缓存时间 (或者给每个缓存加上一个随机值，避免集体失效) 
- 分布式缓存，每个节点只缓存部分的数据，某个节点宕机的时候保证其他节点仍然可用
- 缓存预热，防止系统刚启动时还未将大量数据进行缓存而导致的雪崩  

## 缓存一致性

- 数据更新，立刻更新缓存
- 读取的时候判断是否是最新，不是再更新 (惰性) 

保证缓存一致性需要付出很大的代价，缓存数据适合那些一致性要求不高的数据，允许缓存数据存在一些脏数据。

## 分布式缓存的数据分布

- 哈希分布
- 顺序分布

## 一致性哈希

Distributed Hash Table(DHT)。

将哈希空间 $[0, 2^{n}-1]$ 看成一个哈希环，每个服务器节点都配置到哈希环上。每个数据对象通过哈希取模得到哈希值之后，存放到哈希环中顺时针方向第一个大于等于该哈希值的节点上。

## Redis简介

Redis(Remote Dictionary Server)。

Key-Value 类型的数据库，纯内存操作。

## 数据类型

- STRING, 字符串 + 整型 + 浮点数
    - 字符串操作
    - 数字加减
- LIST
    - 列表操作
    - **可以用来做消息队列**
- SET
    - 添加，删除，检查
    - 交集，并集，差集
    - **可以计算共同喜好，全部的喜好，自己独有的喜好等功能。**
- HASH
    - 添加，删除，检查
    - 遍历
    - **可以用来做模拟 session**
- ZSET
    - 根据分值范围或者成员来获取元素
    - 添加，删除，检查
    - 计算排名
    - **可以做做排行榜**


## 内存淘汰策略

定期删除 + 惰性删除。

在 redis.conf 中配置内存淘汰策略。

```conf
# maxmemory-policy volatile-lru
```

- noeviction: throw error
- allkeys-lru: LRU for all keys
- allkeys-random: random for all keys
- volatile-lru: LRU for expired keys
- volatile-random: random for expired keys
- volatile-ttl: remove the latest key for expired keys


## References

- [redis commands](https://redis.io/commands)
- [redis-tutorial](http://www.runoob.com/redis/redis-tutorial.html)
- **[Redis的几道面试题](https://baijiahao.baidu.com/s?id=1594341157941741587&wfr=spider&for=pc)**
- [Redis 总结精讲](https://blog.csdn.net/hjm4702192/article/details/80518856)
- [CS-Notes/notes/Redis.md](https://github.com/CyC2018/CS-Notes/blob/a015b110387eb4a183fac7dc9526de6cd9e316b3/notes/Redis.md#%E8%B7%B3%E8%B7%83%E8%A1%A8)