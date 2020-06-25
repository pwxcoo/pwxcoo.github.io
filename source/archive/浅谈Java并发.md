---
title: 浅谈Java并发
date: 2018-09-07 13:18:47
categories: base
tags:
- java
- concurrent
---

## Java线程状态

![concurrent.png](/image/concurrent.png)

**Java 中的线程概念是操作系统线程概念的一个 wrapper。**

- 操作系统中线程状态
    - New
    - Ready
    - Blocked
    - Running
    - Terminated
- Java中的线程状态 (Java 中的线程是对操作系统线程的再次封装)
    - New
    - Runnable (Thread.start())
    - Blocked (synchronized, Lock)
    - Time Waiting (Thread.sleep())
    - Waiting (Object.wait(), Object.notify())
    - Terminated (Exception)

### wait() 和 sleep() 和 yield() ?

- wait() 方法会释放 CPU 执行权和占有的锁。**(三者中只有用 wait() 会释放锁)**
- sleep() 方法仅释放 CPU 使用权，锁仍然占用；线程被放入超时等待队列，与 yield() 相比，它会使线程较长时间得不到运行。
- yield() 方法仅释放 CPU 执行权，锁仍然占用，线程会被放入就绪队列，会在短时间内再次执行。
- wait() 和 notify() 必须配套使用，即必须使用同一把锁调用；
- wait() 和 notify() 必须放在一个同步块中调用
- wait() 和 notify() 的对象必须是他们所处同步块的锁对象。

## 使用线程

- Runnable
    ```java
    public interface Runnable {
        public abstract void run();
    }
    ```
- Callable
    ```java
    public interface Callable<V> {
        V call() throws Exception;
    }
    ```
- Thread

----

Callable 可以用 Future 来获得返回值。

```java
public interface Future<V> {

    boolean cancel(boolean mayInterruptIfRunning);

    boolean isCancelled();

    boolean isDone();

    V get() throws InterruptedException, ExecutionException;

    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

Future 是一个接口，Future 提供了三种功能:

- 判断任务是否完成；
- 能够中断任务；
- 能够获取任务执行结果。

同时还有一个 FutureTask 的类，FutureTask 的父类是 RunnableFuture，而 RunnableFuture 继承了 Runnbale 和 Futrue 这两个接口。

```java
public class FutureTask<V> implements RunnableFuture<V>
```

```java
public interface RunnableFuture<V> extends Runnable, Future<V>
```

FutureTask实现了两个接口，Runnable和Future，所以它既可以作为Runnable被线程执行，又可以作为Future得到Callable的返回值。

## 同步问题

**跟线程一样，Java 中的线程是对操作系统的线程概念的 wrapper，而 Java 中的同步问题也是操作系统的同步问题的 wrapper。**

### 操作系统中的同步问题

- 进程同步
    - 信号量
    - 管程 (monitor)
    - 消息传递
- 线程同步
    - 临界区 (Critical Section)
    - 互斥量 (Mutex)
    - 信号量 (Semaphore)
    - 事件 (Event)

### Java中线程同步问题

- synchronized
    - Java中每个对象都有一个内置锁 (修饰静态方法的时候，会锁住整个类)
- 重入锁 (java.util.concurrent) ，ReentrantLock
    - 实现是一个自旋锁，循环调用 CAS 操作来实现加锁
- ThreadLocal
    - 如果使用 ThreadLocal 管理变量，则每一个使用该变量的线程都获得该变量的副本
- 使用阻塞队列实现线程同步
    - java.util.concurrent 的 LinkedBlockingQueue<E>
- 使用原子变量实现线程同步
    - java.util.concurrent.atomic，比如其中的 AtomicInteger
- wait() 和 notify()
    - 示例代码，
    ```java
    public class Main {
        public static void main(String[] args) {
            SyncStack ss = new SyncStack();

            Producer p = new Producer(ss);
            Consumer c = new Consumer(ss);

    //        new Thread(p).start();
    //        new Thread(p).start();
            new Thread(p).start();
            new Thread(c).start();
        }
    }

    class Goods {
        int id;

        Goods(int id) {
            this.id = id;
        }

        public String toString() {
            return "Goods : " + id;
        }
    }


    class SyncStack {
        int index = 0;
        Goods[] arrWT = new Goods[6];

        public synchronized void push(Goods wt) {
            while (index == arrWT.length) {
                try {
                    System.out.println("满了");
                    this.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

            this.notifyAll();
            arrWT[index] = wt;
            index++;
        }

        public synchronized Goods pop() {
            while (index == 0) {
                try {
                    System.out.println("不够");
                    this.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

            this.notifyAll();
            index--;
            return arrWT[index];
        }
    }


    class Producer implements Runnable {
        SyncStack ss = null;

        Producer(SyncStack ss) {
            this.ss = ss;
        }

        public void run() {
            for (int i = 0; i < 20; i++) {
                Goods goods = new Goods(i);
                ss.push(goods);
                System.out.println("生产了: " + goods);

                try {
                    Thread.sleep((int) (Math.random() * 200));
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }


    class Consumer implements Runnable {
        SyncStack ss = null;

        Consumer(SyncStack ss) {
            this.ss = ss;
        }

        public void run() {
            for (int i = 0; i < 20; i++) {
                Goods goods = ss.pop();
                System.out.println("消费了: " + goods);
                try {
                    Thread.sleep((int) (Math.random() * 1000));
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    ```
    - Condition 是在 java 1.5 中才出现的，它用来替代传统的 Object 的 wait()、notify() 实现线程间的协作，相比使用 Object 的 wait()、notify()，使用 Condition 的await()、signal() 这种方式实现线程间协作更加安全和高效。

### ReenTrantLock 和 synchronized 的区别?

- ReenTrantLock 可以指定是公平锁还是非公平锁。而 synchronized 只能是非公平锁。
- ReenTrantLock 提供了一个 Condition (条件) 类，用来实现分组唤醒需要唤醒的线程们，而不是像 synchronized 要么随机唤醒一个线程要么唤醒全部线程。
- ReenTrantLock 提供了一种能够中断等待锁的线程的机制，通过 lock.lockInterruptibly() 来实现这个机制。




## References

- [Java并发之Runnable、Callable、Future、FutureTask](https://www.jianshu.com/p/cf12d4244171)
- [进程同步的几种方式](https://www.cnblogs.com/Allen-rg/p/7172958.html)
- [线程同步的几种方式](https://www.cnblogs.com/Allen-rg/p/7172970.html)
- [ReenTrantLock可重入锁 (和synchronized的区别) 总结](https://blog.csdn.net/qq838642798/article/details/65441415)
- [CS-Notes/notes/Java 并发.md](https://github.com/CyC2018/CS-Notes/blob/a015b110387eb4a183fac7dc9526de6cd9e316b3/notes/Java%20%E5%B9%B6%E5%8F%91.md)
- [java对象结构](https://blog.csdn.net/zqz_zqz/article/details/70246212)
- [锁原理: 偏向锁、轻量锁、重量锁](https://www.cnblogs.com/wewill/p/8058292.html)
- [java 中的锁 -- 偏向锁、轻量级锁、自旋锁、重量级锁](https://blog.csdn.net/zqz_zqz/article/details/70233767)