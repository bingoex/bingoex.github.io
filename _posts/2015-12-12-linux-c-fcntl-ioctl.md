---
layout: post
title:  Linux下开发－fcntl和ioctl的区别
categories:  C/C++
description: 
keywords: 
---


fcntl函数**可以改变一个已打开的文件的属性**，可以重新设置读、写、追加、非阻塞等标志（这些标志称为File Status Flag），而不必重新open文件。

通过fcntl设置的都是**当前进程如何访问设备或文件的访问控制属性**，例如读、写、追加、非阻塞、加锁等，**但并不设置文件或设备本身的属性**，例如文件的读写权限、串口波特率等。

```c
int fcntl(int fd, int cmd);
int fcntl(int fd, int cmd, long arg);
int fcntl(int fd, int cmd, struct flock *lock);
```




**ioctl函数用于设置某些设备本身的属性**，例如串口波特率、终端窗口大小，注意区分这两个函数的作用。

ioctl用于向设备发控制和配置命令，有些命令也需要读写一些数据，但这些数据是不能用read/write读写的，称为Out-of-band数据。也就是说，read/write读写的数据是in-band数据，是I/O操作的主体，而ioctl命令传送的是控制信息，其中的数据是辅助的数据。例如，在串口线上收发数据通过ead/write操作，而串口的波特率、校验位、停止位通过ioctl设置，A/D转换的结果通过read读取，而A/D转换精度和工作频率通过ioctl设置。

```c
int ioctl(int d, int request, ...);
```



