---
title: volatile
date: 2020-01-18 15:18:26
categories: note
tags:
- java
- concurrent
---

## Lock

Locks offer two primary features: **mutual exclusion** and **visibility**.

- **Mutual exclusion** means that only one thread at a time may hold a given lock, and this property can be used to implement protocols for coordinating access to shared data such that only one thread at a time will be using the shard data.
- **Visibility** is more subtle and has to do with ensuring that change made to shared data prior to releasing a lock are made visible to another thread that subsequently acquires that lock -- without the visibility guarantees provides by synchronization, thread could see stale or inconsistent values for shared variables, which could cause a host of serious problems.


## Volatile

Volatile variables in the Java language can be thought of as "`synchronized` lite"; they require less coding to use than `synchronized` blocks and often have less runtime overhead, but they can only be used to do a subset of the things that `synchronized` can. In cases where reads greatly out number writes, volatile variables can often reduce the performance cost of synchronization compared to locking.

Volatile provides two guarantees:

- **visibility**. Volatile variables share the visibility features of synchronized, but none of atomicity features. This mean that threads will automatically see the most up-to-date value for volatile variables.
- **establish a happen-before relationship**. (In Java5 or later)

### Condition for correct use of volatile

You can use volatile variables instead of locks only under a restricted set of circumstances. Both of the following criteria must be met for volatile variables to provide the desired thread-safety:

- Writes to the variables do not depend on its current value.
- The variable does not participate in invariants with other variables.

### Implement volatile

`Memory Barrier`. More details see [The JSR-133 Cookbook for Compiler Writers](http://gee.cs.oswego.edu/dl/jmm/cookbook.html).


## References

- [Managing volatility](https://www.ibm.com/developerworks/library/j-jtp06197/index.html)
- [Fixing the Java Memory Model, Part 1](https://www.ibm.com/developerworks/library/j-jtp02244/index.html)
- [Fixing the Java Memory Model, Part 2](https://www.ibm.com/developerworks/library/j-jtp03304/)
- [The Java Memory Model](http://www.cs.umd.edu/~pugh/java/memoryModel/)