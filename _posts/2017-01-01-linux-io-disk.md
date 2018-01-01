---
layout: post
title: 【精品】Linux文件IO
categories: Linux 操作系统原理
description: linux文件IO
keywords: 
---

深入剖析fwrite背后发生的事情

![](/images/posts/2015-09-09-linux-io-disk.md/1.gif)

```C 
{
    char *buf = malloc(MAX_BUF_SIZE);
    strncpy(buf, src, , MAX_BUF_SIZE);
    fwrite(buf, MAX_BUF_SIZE, 1, fp);
    fclose(fp);
}
```

malloc的buf对应图层中applicationbuffer，即应用程序buffer；

调用fwrite后，把数据从application buffer 拷贝到了 CLib buffer，即C库标准IObuffer。fwrite返回后，数据还在CLib buffer;

当调用fclose的时候，fclose调用会把数据刷新到磁盘介质上;

除了fclose方法外，还有一个主动刷新操作fflush 函数，不过fflush函数只是把数据从CLibbuffer 拷贝到page  cache中，并没有刷新到磁盘上，从page cache刷新到磁盘上可以通过调用fsync函数完成;

数据到了page cache后，内核有pdflush线程在不停的检测脏页，判断是否要写回到磁盘中。把需要写回的页提交到IO队列——即IO调度队列。由IO调度队列调度策略决定何时写回磁盘;
 
IO队列有2个主要任务。一是合并相邻扇区的，二是排序。合并相信很容易理解，排序就是尽量按照磁盘选择方向和磁头前进方向排序。因为磁头寻道时间是和昂贵的;

这里IO队列和我们常用的分析工具iostat关系密切。iostat中rrqm/s wrqm/s表示读写合并个数。avgqu-sz表示平均队列长度;

从IO队列出来后，就到了驱动层，驱动层通过DMA，将数据写入磁盘cache;
 
 
# 磁盘操作优化



#### 去掉CLib buffer

直接写到page cache，调用**write函数**，是直接通过系统调用把数据从应用层拷贝到内核层，从application buffer 拷贝到 pagecache 中
 
#### 去掉系统调用

write会触发用户态/内核态切换。**mmap**把page cache 地址空间映射到用户空间，应用程序像操作应用层内存一样，写文件。省去了系统调用开销
 

#### 去掉page cache

**open文件带上O_DIRECT参数**， write文件时直接写到设备。
 
#### 去掉文件系统

**直接写扇区就是所谓的RAW设备写**，绕开了文件系统，直接写扇区，像fdsik，dd，cpio之类的工具就是这一类操作.
 
 
# 数据读写操作原子性


write操作如果写大小小于PIPE_BUF（一般是4096），是原子操作，能保证两个进程“AAA”，“BBB”写操作，不会出现“ABAABB”这样的数据交错。

O_APPEND 标志能保证每次重新计算pos，写到文件尾的原子性。
 
 
# 磁盘IO性能


机械硬盘顺序写30MB\s，顺序读取速率一般50MB\s，好的可以达到100多M\s, SSD读达到400MB\s，SSD写性能和机械硬盘差不多。
 


