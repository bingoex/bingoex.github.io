---
layout: post
title: 深入理解Java虚拟机读书笔记－java对象模型和内存参数
categories: Java 读书笔记
description: 入理解Java虚拟机读书笔记－java对象模型和内存参数
keywords: 
---

java对象模型和内存参数

# 对象访问

![](/images/posts/2015-09-13-jvm-book-module-params.md/1.png)

reference中存储的是稳定的句柄地址，在对象被移动（垃圾收集时移动对象是非常普遍的行为）时只会改变句柄中的实例数据指针，而reference本身不需要修改。
 
![](/images/posts/2015-09-13-jvm-book-module-params.md/2.png)
 
直接指针访问方式的最大好处就是速度更快，它节省了一次指针定位的时间开销。
 
 
堆的最小值-Xms参数和最大值-Xmx参数
 
-XX：+HeapDumpOnOutOfMemoryError可以让虚拟机在出现内存溢出异常时Dump出当前的内存堆转储快照以便事后进行分析
 
栈容量由-Xss参数设定
 
如果线程请求的栈深度大于虚拟机所允许的最大深度，将抛出StackOverflowError异常。

如果虚拟机在扩展栈时无法申请到足够的内存空间，则抛出OutOfMemoryError异常。（如果是建立过多线程导致的内存溢出，在不能减少线程数或者更换64位虚拟机的情况下，就只能通过减少最大堆和减少栈容量来换取更多的线程）
 
运行时常量池是方法区的一部分。由于常量池分配在永久代内，我们可以通过-XX：PermSize和-XX：MaxPermSize限制方法区大小，从而间接限制其中常量池的容量
 
DirectMemory容量可通过-XX：MaxDirectMemorySize指定（默认与Java堆最大值-Xmx指定的一样）
 
