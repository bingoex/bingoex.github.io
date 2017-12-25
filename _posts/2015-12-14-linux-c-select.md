---
layout: post
title: Linux下开发－IO复用
categories: C/C++ Linux
description: 
keywords: 
---

本文对select、pselect、poll、epoll作简单的介绍

# select、pselect
```c
int select(int n, fd_set *readfds,fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
int pselect(int n, fd_set *readfds,fd_set *writefds, fd_set *exceptfds, const struct timespec *timeout, constsigset_t *sigmask);
```
select和pselect都是等待一系列的文件描述符（fd）的状态发生变化。

这两个函数基本上是一致，但是有三个区别：
1. select函数用的timeout参数是timeval的结构体（包含秒和微秒），而pselect用的是timespec结构体（包含秒和纳秒），更精确。
2. select函数可能会为了指示还剩多长时间而更新timeout参数，然而pselect不会改变timeout参数。
3. select函数没有sigmask参数，当pselect的sigmask参数为null时，两者行为时一致的。有sigmask的时候，pselect相当于如下的select()函数，在进入select()函数之前手动将信号的掩码改变，并保存之前的掩码值；select()函数执行之后，再恢复为之前的信号掩码值。

 
缺点：
1. 同时监听的fd数量有限（1024）
2. 函数返回后，需要全量遍历所有的fd，询问是否有事件发生，在fd较多的时候性能较差。



# poll
跟select差不多，只是API优化了。且没有fd监听数量的限制。

缺点：
1. 函数返回后，需要全量遍历所有的fd，询问是否有事件发生，在fd较多的时候性能较差。



# epoll

在poll的基础上优化，函数直接返还发生事件的fd。
 
## LT和ET

LT（level triggered 水平触发）是缺省的工作方式，并且同时支持block和no-blocksocket。在这种做法中，内核告诉你一个文件描述符是否就绪了，然后你可以对这个就绪的fd进行IO操作。如果你不作任何操作，内核还是会继续通知你的，所以，这种模式编程出错误可能性要小一点。传统的select/poll都是这种模型的代表。
 
ET （edge-triggered 边缘触发）是高速工作方式，只支持no-block socket。在这种模式下，当描述符从未就绪变为就绪时，内核通过epoll告诉你。然后它会假设你知道文件描述符已经就绪，并且不会再为那个文件描述符发送更多的就绪通知

# EPOLLOUT事件

一、EPOLLOUT事件在连接时触发一次，表示可写。

二、其他时候想要触发，那你要先准备好下面条件：
1. 某次write，写满了发送缓冲区，返回错误码为EAGAIN。
2. 对端读取了一些数据，又重新可写了，此时会触发EPOLLOUT。
 
如果你真的想强制触发一次，也是有办法的，直接调用epoll_ctl重新设置一下event就可以了，event跟原来的设置一模一样都行（但必须包含EPOLLOUT），关键是重新设置，就会马上触发一次EPOLLOUT事件。
 
 
 
 
 

