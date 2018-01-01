---
layout: post
title: 【精品】free命令引发的思考
categories: Linux
description: free命令引发的思考
keywords: free, swap, 内存优化
---


下文所操作机器已关闭swap。如描述有误，望轻拍，谢谢。[本文理论基础](https://bingoex.github.io/2016/01/01/linux-memory-1/)


# free查看内存使用的基本命令
![](/images/posts/2016-08-02-linux-memory-2/1.png)

第一行7625M是系统实际使用的所有内存，但其中包含了buffer/cache。237M是系统当前空闲的物理内存，这是不是意味着我们的系统内存不够呢？答案是不一定。
  
# 什么是Buffer Cache和Page Cache？
前者针对磁盘块的读写，后者针对文件inode的读写。这些Cache有效缩短了I/O系统调用(比如read,write)的时间，提高了系统性能，**所以Linux系统会占用部分空闲物理内存作缓存。当程序需要内存且当前空余物理内存不足时，linux系统会释放缓存的内存供进程使用**。
  
第二行的6253M就是加上缓存之后，系统剩余的物理内存。那是不是意味着我们系统内存很宽裕呢？答案也是不一定的。
  
在大部分Linux系统中6253M**这部分缓存内存不一定能够被回收供进程使用，如共享内存、动态库、临时文件系统（tmpfs）、当前状态属于Dirty的缓存等**。所以当我们计算系统空余物理内存时，还需要减去上述内存。
  
# 如何查看系统共享内存
因历史原因，共享内存一般分为SystemV和POSIX两种。其中POSIX的底层实现是使用mmap的，而SytemV则有可能不是（目前公司的suse及tlinue均不是）。
  
Linux提供一种“临时”文件系统叫做tmpfs，它可以将内存的一部分空间拿来当做文件系统使用，使内存空间可以当做目录文件来用，如 /dev/shm的tmpfs目录，其中包括POSIX共享内存。
  
注意**共享内存是按需分配**的，即使新建一个很大的共享内存，但是没有使用或者只使用了（如访问了相应的地址）其中的一部分，那么只会分配使用了的部分内存，而不会全部分配。但因为我们业务新建共享内存后会进行一次memset操作并且业务关闭了Swap，所以是全部分配的。
  
## SytemV内存查看命令ipcs -m
![](/images/posts/2016-08-02-linux-memory-2/2.png)
  
## POSIX内存查看命令du -sm /dev/shm
![](/images/posts/2016-08-02-linux-memory-2/3.png)


# 如何清除cache
```shell
 echo1 > /proc/sys/vm/drop_caches   表示清除page cache。
 echo2 > /proc/sys/vm/drop_caches   表示清除slab分配器中的对象（包括目录项缓存和inode缓存）。
 echo3 > /proc/sys/vm/drop_caches   表示清除page cache和slab分配器中的缓存对象。
```
![](/images/posts/2016-08-02-linux-memory-2/4.png)

经过上面的命令，linux系统已经尽可能地把可以清理的缓存清理掉了。所以得出可用物理内存1320M。
  
操作系统、所有进程使用的物理内存（堆、栈内存，不包括共享内存）的大小 = 清理缓存后的used（6542M）- 共享内存（2581M +2364M）=  1597M
  
伴随着cache清除的结果往往是IO的飙高，因为内核要对比cache中的数据和对应磁盘文件上的数据是否一致，如不一致需写回磁盘
  
# Swap了一定是内存不够了？

当内存不够用时，linux系统除了清理缓存外，还有可能会将很久没有使用的内存交换到磁盘，从而释放更多的空间，也就是我们俗称的Swap。这样的作用是有利也有弊的。
  
好处：当被交换出去的内存是长时间不被使用的，那swap后将得到更大的空闲内存供进程使用，而代价也仅仅是这一瞬间的磁盘IO，且不会影响现有的缓存数据。
  
坏处：当交换出去的内存在短时间内系统又再次使用上则会出现频繁的磁盘IO，严重影响性能，这种情况一般是内存到达了瓶颈。
  
Linux系统当出现内存吃紧时，到底会选择释放内存还是Swap内存是有倾向的，可以通过调整/proc/sys/vm/swappiness进而影响系统的行为。（例如：当swappiness值为90，内存不足时，系统将更倾向于使用swap。）

#### Swap的启动与关闭
```shell
swapoff -a && swapon -a
``` 
#### 如何查看进程使用的Swap
```shell
 /proc/pid/smaps
```

# 其他查看内存的命令
- top
- pmap
- smem
- valgrind

# 参考内容：
<http://www.tuicool.com/articles/muqIBvy>
<https://www.centos.org/docs/5/html/5.1/Deployment_Guide/s2-proc-meminfo.html>


