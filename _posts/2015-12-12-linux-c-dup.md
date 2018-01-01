---
layout: post
title: Linux下开发－dup和dup2
categories:  C/C++
description: 
keywords: 
---


dup和dup2都可用来复制一个现存的文件描述符，**使两个文件描述符指向同一个file结构体**。如果两个文件描述符指向同一个file结构体，**File Status Flag和读写位置只保存一份在file结构体中，并且file结构体的引用计数是2**。

如果**两次open同一文件得到两个文件描述符，则每个描述符对应一个不同的file结构体**，可以有不同的FileStatus Flag和读写位置。

![](/images/posts/2015-12-12-linux-c-dup.md/1.png)

```c
#include <unistd.h>
int dup(int oldfd);
int dup2(int oldfd, int newfd);
```

如果调用成功，这两个函数都返回新分配或指定的文件描述符，如果出错则返回-1。

**dup返回的新文件描述符，是该进程未使用的最小文件描述符，这一点和open类似**。

**dup2可以用newfd参数指定新描述符的数值**。如果newfd当前已经打开，则先将其关闭再做dup2操作，如果oldfd等于newfd，则dup2直接返回newfd而不用先关闭newfd再复制。
 
```c
int  dup(int oldfd);
```
dup()用来复制参数oldfd所指的文件描述词，并将它返回。此新的文件描述词和参数oldfd指的是同一个文件，共享所有的锁定、读写位置和各项权限或旗标。例如，**当利用lseek()对某个文件描述词作用时，另一个文件描述词的读写位置也会随着改变**。不过，文件描述词之间并不共享close-on-exec旗标（<http://www.tuicool.com/articles/FvIjMv>）。

当复制成功时，则返回最小及尚未使用的文件描述词。若有错误则返回-1，errno会存放错误代码。

```c
int  dup2(int oldfd, int newfd);
```
dup2()用来**复制参数oldfd所指的文件描述词，并将它拷贝至参数newfd**后一块返回。若参数newfd为一个已打开的文件描述词，则newfd所指的文件会先被关闭。dup2()所复制的文件描述词，与原来的文件描述词共享各种文件状态。

当复制成功时，则返回最小及尚未使用的文件描述词。若有错误则返回-1，errno会存放错误代码。





