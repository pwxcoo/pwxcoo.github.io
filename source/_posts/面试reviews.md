---
title: 面试reviews
date: 2018-02-27 22:16:36
categories: 备忘
tags:
- 基础
- Java
- 计算机网络
---

记录一下自己为了 18 年春招面试特地复习的一些知识点。因为想投的是研发实习，一般都是 Java 的，所以内容基本就是 Java 的研发知识点。

另外我发现一件很屁股的事情。就是我对分布式一些东西没接触过，所以看到别人写的一些东西太概念了。

## TODO
- [ ] 手撸一下[红黑树](https://zhuanlan.zhihu.com/p/24367771)。
- [x] 手撸快排，堆排序

## Java的数据结构
```
java.util.Collection [I]
    +--java.util.List [I]
       +--java.util.ArrayList [C]    
       +--java.util.LinkedList [C]  
       +--java.util.Vector [C]    //线程安全
          +--java.util.Stack [C]  //线程安全
    +--java.util.Set [I]                   
       +--java.util.HashSet [C]      
       +--java.util.SortedSet [I]    
          +--java.util.TreeSet [C]    
    +--Java.util.Queue[I]
        +--java.util.Deque[I]   
        +--java.util.PriorityQueue[C]  
java.util.Map [I]
    +--java.util.SortedMap [I]
       +--java.util.TreeMap [C]
    +--java.util.Hashtable [C]   //线程安全
    +--java.util.HashMap [C]
    +--java.util.LinkedHashMap [C]
    +--java.util.WeakHashMap [C]

[I]：接口
[C]：类
```
Java 集合框架的基本接口/类层次结构大概就是这样的。看一下[Java数据结构概览](https://segmentfault.com/a/1190000009797159)，C++ 有一个 STL 源码剖析，但是 Java 好像没有 JCF(Java Collections Framework) 的书籍。Github 上有一个讲解 JCF 的仓库[Java Collections Framework Internals](https://github.com/CarpenterLee/JCFInternals)，是结合 JDK1.7 讲的，JDK1.8 有一些变化（如 HashMap 从数组+链表变成了数组+链表+红黑树），JDK1.8 比 JDK1.7 源码要复杂一些，先看 JDK1.7 然后在看 1.8 和 1.7 有什么进步就可以了。

## 浅谈 JDK1.8 中的 HashMap
[Java 8系列之重新认识HashMap](https://zhuanlan.zhihu.com/p/21673805)

## JVM
- [类的加载过程](http://blog.csdn.net/ns_code/article/details/17881581)
- [GC 和 内存管理](http://www.importnew.com/13504.html)

## 计算机网络 
- get/post 区别
- TCP/IP
- TCP 三次握手，四次挥手
- TCP 滑动窗口

## Mysql
[MySQL tutorial book](https://github.com/jaywcjlove/mysql-tutorial)
- [数据库存储引擎](https://github.com/jaywcjlove/mysql-tutorial/blob/master/chapter3/3.5.md)
- 乐观锁，悲观锁
- [分布式集群](http://www.infoq.com/cn/articles/exploration-of-distributed-mysql-cluster-scheme)

## 框架
- [SpringMvc](https://www.zhihu.com/question/38696452/answer/267119037)
- [Mybatis](https://github.com/tuguangquan/mybatis)

## 分布式锁
[分布式事物与分布式锁](https://www.jianshu.com/p/c2b4aa7a12f1)

