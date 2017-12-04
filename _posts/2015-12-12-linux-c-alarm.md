---
layout: post
title:  Linux下开发－alarm
categories: Linux C/C++
description: 
keywords: 
---

![](/images/posts//.png)

```c
#include<unistd.h>
unsigned int alarm（unsigned int seconds);
```

alarm也称为闹钟函数，它可以在进程中设置一个定时器，当定时器指定的时间到时，它向进程发送SIGALRM信号。如果忽略或者不捕获此信号，则其默认动作是终止调用该alarm函数的进程。

一个进程只能有一个闹钟时间，如果在调用alarm之前已设置过闹钟时间，则任何以前的闹钟时间都被新值所代替。需要注意的是，经过指定的秒数后，信号由内核产生，由于进程调度的延迟，所以进程得到控制从而能够处理该信号还需要一些时间。

如果有以前为进程登记的尚未超时的闹钟时钟，而且本次调用的seconds值是0，则取消以前的闹钟时钟，其余留值仍作为alarm函数的返回值。


成功：如果调用此alarm（）前，进程已经设置了闹钟时间，则返回上一个闹钟时间的剩余时间，否则返回0

出错：-1





