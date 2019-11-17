---
title: '[Source Code]DelayQueue'
date: 2019-11-18 00:34:27
categories: source
tags:
- java
- concurrent
---

An unbound blocking queue of Delay elements, in which an element can only be taken when its delay has expired. The head of queue is that Delayed element whose delay expired furthest in the past. If no delay has expired there is no head and poll wil return null. Expiration occurs when an element's getDelay(TimeUnit.NANOSECONDS) method return a value less than or equal to zero. Even though unexpired elements cannot be removed using take or poll, they are otherwise treated as normal elements. For example, the size method returns the count of both expired and unexpired elements. This queue does not permit null element.

**In short, the queue is a Priority Queue. Using leader-follower mode to reduce followers' dispatch.**

## private variable

```java
private final transient ReentrantLock lock = new ReentrantLock();
private final PriorityQueue<E> q = new PriorityQueue<E>();
private Thread leader = null;
/**
* Condition signalled when a newer element becomes available
* at the head of the queue or a new thread may need to
* become leader.
*/
private final Condition available = lock.newCondition();
```

## offer()

```java
    /**
     * Inserts the specified element into this delay queue.
     *
     * @param e the element to add
     * @return {@code true}
     * @throws NullPointerException if the specified element is null
     */
    public boolean offer(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            q.offer(e);
            if (q.peek() == e) {
                // 如果新增的元素需要的延迟更小，leader置为null，唤醒线程来抢锁
                leader = null;
                available.signal();
            }
            return true;
        } finally {
            lock.unlock();
        }
    }
```

## take()

```java
    /**
     * Retrieves and removes the head of this queue, waiting if necessary
     * until an element with an expired delay is available on this queue.
     *
     * @return the head of this queue
     * @throws InterruptedException {@inheritDoc}
     */
    public E take() throws InterruptedException {
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            for (;;) {
                E first = q.peek();
                if (first == null)
                    // 队列是空的，就等着offer之后插进元素来唤醒
                    available.await();
                else {
                    long delay = first.getDelay(NANOSECONDS);
                    if (delay <= 0)
                        return q.poll();
                    // 防止因为这个线程一直被阻塞，但其实这个对象可能已经被take掉了，却一直拿着对象的引用而无法被垃圾回收。这个点太细了
                    first = null; // don't retain ref while waiting
                    if (leader != null)
                        // 有leader，follower就一直等着就好了
                        available.await();
                    else {
                        // 没有leader，该线程变成leader
                        Thread thisThread = Thread.currentThread();
                        leader = thisThread;
                        try {
                            // 定时阻塞
                            available.awaitNanos(delay);
                        } finally {
                            // 什么情况下 leader != thisThread 呢？offer之后重新搞了leader，然后唤醒了别的线程抢到了leader。
                            if (leader == thisThread)
                                leader = null;
                        }
                    }
                }
            }
        } finally {
            if (leader == null && q.peek() != null)
                // 如果没有leader，队列不为空的情况下，需要找个线程来当leader
                available.signal();
            lock.unlock();
        }
    }

```

















## references

- [oracle-docs: Class DelayQueue<E extends Delayed>](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/DelayQueue.html)
- [source code: DelayQueue](https://github.com/Himansu-Nayak/java7-sourcecode/blob/master/java/util/concurrent/DelayQueue.java)
