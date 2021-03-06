---
layout: post
title: Linux到底有没有线程
categories: Linux
description: 
keywords: 
---


有的人说Linux没有线程只有进程，有的人说Linux当然有线程，没有线程pthread库是干吗的？NPTL又是干嘛用？既然能用pthread库来创建线程，以及可以处理线程间的通信，当然可以认为在Linux中线程肯定是存在的。

从目前Linux内核实现的角度来看，Linux当前应该还是采用的一对一模型，创建一个用户态空间的线程，在内核中会通过一个轻量级的进程来管理，线程调度相当于进程调度。当前Linux内核中，**无论是创建一个线程还是一个进程都是会调用clone()系统调用**，只不过是参数不同，创建线程的话clone()的参数是(**CLONE_VM** \| CLONE_FS \| CLONE_FILES \| CLONE_SIGHAND)，其中CLONE_VM也指定了核心态空间中，此轻量级进程（即线程）和父进程共享地址空间，因此这个线程才可以访问父进程的地址空间。

因此，Linux和Windows的线程虽然实现不一样，但是应该可以大胆的说Linux是存在线程的，只是Linux的实现方式和Windows的不一样。至于说Linux的线程就是轻量级进程那只是从核心态空间的层面来看。


线程和进程的区别

- 进程是资源分配的最小单位，线程是程序执行的最小单位。
- 线程间共享进程所拥有的全部资源。
- 进程有独立的地址空间，线程没有单独的地址空间（同一进程内的线程共享进程的地址空间，共享大部分数据。线程间通信方便）
- 启动一个线程所花费的资源远远小于启动一个进程所花费的资源。
- 线程，它们彼此之间使用相同的地址空间。





