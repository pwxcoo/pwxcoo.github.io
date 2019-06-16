---
title: 'IO Multiplexing: The select and poll Function'
date: 2019-06-10 17:16:12
categories: base
tags:
- unp
- io
---

## Introduction

When the TCP client is handling two inputs at the same time: standard input and a TCP socket, we encounter a problem when the client was blocked in a call to `fgets` (on standard input) and the server process was killed. The server TCP correctly sent a FIN to the client TCP, but since the client process was blocked reading from standard input, it never saw the EOF until it read from the socket (possibly much later).

We want to be notified if one or more IO condition are ready (i.e., input is ready to be read, or the descriptor is capable of taking more output). The capability is called **IO multiplexing** and is provided by the `select` and `poll` functions, as well as a newer POSIX variation of the former, called `pselect`.

IO multiplexing is typically used in networking application in many scenarios. And IO multiplexing is not limited to networking programming, many nontrivial applications find a need for these techniques.


## IO Models

We firstly examine the basic differences in the five IO models that are available to us under Unix:

- blocking IO
- nonblocking IO
- IO multiplexing (select and poll)
- signal driven IO
- asynchronous IO (the POSIX aio_ functions)

There are normally two distinct phases for an input operations:

1. Waiting for data to be ready. This involves waiting for data to arrive on the network. When the packet arrives, it is copied into a buffer within the kernel.
2. Copying the data from the kernel to the process. This means copying the (ready) data from the kernel's buffer into our application buffer.


### Blocking IO Model

The most prevalent model for IO is the blocking IO model. By default, all socket are blocking. The scenario is shown in the figure below:

![blocking IO.gif](https://ws1.sinaimg.cn/large/8a79c363ly1g439nq3yodg20dw07qt8t.gif)

We use UDP for this example instead of TCP because with UDP, the concept of data being "ready" to read is simple: either an entire datagram has been received or it has not. With TCP it gets more complicated, as additional variables such as the socket's low-water mark come into play.

We also refer to `recvfrom` as a system call to differentiating between our application and the kernel, regardless of how `recvfrom` is implemented (system call on BSD and function that invokes `getmsg` system call on System V). There is normally a switch from running in application to running in the kernel, followed at some time later by a return to the application.

In the figure above, the process calls `recvfrom` and the system call does not return until the datagram arrives and is copied into our application buffer, or an error occurs. The most common error is the system call being interrupted by signal. We say that the process is blocked the entire time from when it calls `recvfrom` until it returns. When `recvfrom` return successfully, our application process the datagram.

### Nonblocking IO Model

When a socket is set to be nonblocking, we are telling the kernel "When an IO operation that I request cannot be completed without putting the process to sleep, do not put the process to sleep, but return an error instead". The figure is below:

![nonblocking-io.gif](https://ws4.sinaimg.cn/large/8a79c363ly1g43bp0b7gng20dw07q74m.gif)

- For the first three `recvfrom`, there is no data to return and the kernel immediately returns an error of `EWOULDBLOCK`.
- For the forth time we call `recvfrom`, a datagram is ready, it is copied into our application buffer, and `recvfrom` returns successfully. We then process the data.

When an application sits in a loop calling `recvfrom` on a nonblocking descriptor like this, it is called **polling**. The application is continually polling the kernel to see if some operation is ready. This is often a waste of CPU time, but this model is occasionally encountered, normally on systems dedicated to one function.

### IO Multiplexing Model

With **IO multiplexing**, we call `select` or `poll` and block in one of these two system calls, instead of blocking in the actual IO system call. The figure is a summary of the IO multiplexing model:

![io-multiplexing.gif](https://wx2.sinaimg.cn/large/8a79c363ly1g43c3o4uizg20dw07kmxf.gif)

We block in a call to `select`, waiting for the datagram socket to be readable. When `select` returns that the socket is readable, we then call `recvfrom` to copy the datagram into our application buffer.

Comparing blocking model:

- Disadvantage: using `select` requires two system call (`select` and `recvfrom`) instead of one
- Advantage: we can wait for more than one descriptor to be ready

Another closely related IO model is to use multithreading with blocking IO. That model very closely resembles the model described above, except that instead of using `select` to block on multiple file descriptors, the program uses multiple threads (one per file descriptor), and each thread is then free to call blocking system calls like `recvfrom`.

### Signal-Driven IO Model

The **signal-driven IO model** uses signals, telling the kernel to notify us with the `SIGIO` signal when descriptor is ready. The figure is below:

![signal-driven-io.gif](https://ws3.sinaimg.cn/large/8a79c363ly1g43cg44s7lg20dw07mmxd.gif)

- We first enable the socket for signal-driven IO and install a signal handler using the `sigaction` system call. The return from this system is immediate and our process continues; it is not blocked.
- When the datagram is ready to be read, the `SIGIO` signal is generated for our process. We can either:
    - read the datagram from the signal handler by calling `recvfrom` and then notify the main loop that data is ready to be process.
    - notify the main loop and let it read the datagram.

The advantage to this model is that we are not blocked while waiting for the datagram to arrive. The main loop can continue executing and just wait to be notified by the signal handler that either the data is ready to process or the datagram is ready to be read.

### Asynchronous IO model

**Asynchronous IO** is defined by the POSIX specification, and various differences in the read-time function that appeared in the various standards which came together to form the current POSIX specification have been reconciled.

These functions work by telling the kernel to start the operation and to notify us when then entire operation (including the copy of the data from the kernel to our buffer) is complete. **The main difference between this model and the signal-driven IO model is that with signal-driven IO, the kernel tells us when IO operation can be initiated, but asynchronous IO, the kernel tells us when an IO operation is complete.** See the figure below for example:

![asynchronous-io.gif](https://ws2.sinaimg.cn/large/8a79c363ly1g43d32olhtg20dw07s0su.gif)

- We call `aio_read` (the POSIX asynchronous IO function begin with `aio_` or `lio_`, **This system call returns immediately and our process is not blocked while waiting for the IO to complete**) and pass the kernel the following:
    - descriptor, buffer pointer, buffer size (the same three arguments for `read`)
    - file offset (similar to `lseek`)
    - and how to notify us when the entire operation is complete
    >
- We assume in this example that we ask the kernel to generate some signal when the operation is complete. This signal is not generated until the data has been copied into our application buffer, which is different from the signal-driven IO model.

## Comparison of the IO models

The figure below is a comparison of the five different IO models:

![comparison-io.gif](https://ws1.sinaimg.cn/large/8a79c363ly1g43dpda7cvg20dw07mt95.gif)

The main difference between the first four models is the first phase, as the second phase in the first four models is the same: the process is blocked in a call to `recvfrom` while the data is copied from the kernel to the caller's buffer. Asynchronous IO, however, handles both phases and is different from the first four.

## Synchronous IO versus Asynchronous IO

POSIX defines these two terms as follows:

- A synchronous IO operation causes the requesting process to be blocked until that IO operation completes.
- An asynchronous IO operation does not cause the requesting process to be blocked.

Using these definitions, the first four IO models (blocking, nonblocking, IO complexing and signal-driven IO) are all synchronous because the actual IO operation (`recvfrom`) blocks the process. Only the asynchronous IO model matches the asynchronous IO definition.












## References

- [Chapter 6. I/O Multiplexing: The select and poll Functions](https://notes.shichao.io/unp/ch6/)
- [UNP - 6.2 I/O Models](http://www.masterraghu.com/subjects/np/introduction/unix_network_programming_v1.3/ch06lev1sec2.html)
