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

## Redis

redis 内部使用的是文件事件处理器（file event handler），这个文件时间处理器是单线程的，所以说 redis 叫单线程模型。

### Redis为什么单线程效率也这么高：

1. 纯内存操作
2. 核心是基于非阻塞的 IO 多路复用
3. 单线程避免了一些场景下频繁上下文切换的问题，预防了多线程可能产生的竞争问题

### Redis过期策略

定期删除（部分） + 惰性删除

### Redis内存淘汰机制

- noeviction
- **allkeys-lru**
- allkeys-random
- volatile-lru (volatile 表示从设置了过期时间的建空间里操作)
- volatile-random
- volatile-ttl

### Redis实现高并发

主从架构

### Redis实现高可用

哨兵，实现主备切换

### Redis实现持久化

- RDB：周期性持久化
- AOF：对每条写入命令作为日志，以 append-only 的模式写入一个日志文件中

如果同时使用 RDB 和 AOF 两种持久化机制，那么在 redis 重启的时候，会使用 AOF 来重新构建数据，因为 AOF 中的数据更加完整。

### Redis集群

- redis cluster 可以自动将数据切片，每个 master 上放一部分数据，节点之间通信用 gossip 协议（每个节点持有一份元数据）
- 分布式寻址（一致性哈希算法）
- 主从切换
    - 判断节点宕机（超过半数）
    - 过滤 slave 节点（和 master 节点断开时间超过了某时间，不配变成 master）
    - 从节点选举（超过半数）

### 缓存雪崩

事前：redis 高可用，主从+哨兵，redis cluster，避免全盘崩溃。
事中：本地 ehcache 缓存 + hystrix 限流&降级，避免 MySQL 被打死。
事后：redis 持久化，一旦重启，自动从磁盘上加载数据，快速恢复缓存数据。

### 缓存穿透

非正常请求也存一个空值到缓存里，并设置过期时间。

### 缓存击穿

热点数据突然失效。

1. 热点不过期
2. redis 或 zookeeper 实现互斥锁，等待第一个请求建完缓存后在释放锁



## Reference

- [Java 进阶扫盲](https://doocs.github.io/advanced-java/#/)