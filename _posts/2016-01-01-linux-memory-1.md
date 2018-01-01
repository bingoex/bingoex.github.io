---
layout: post
title: Linux内存浅析
categories: Linux 优化
description: Linux内存浅析
keywords: Linux, 内存, 优化, 内核参数, 系统调优
---

今天跟大家探讨下linux下内存相关的知识点。

# free命令
![](/images/posts/2016-01-01-linux-memory-1/1.png)
进程使用内存1.46G。Buffer、Cached使用内存6.2G。但**Buffer、Cache这部分空闲内存不一定能够被回收供进程使用**（如共享内存、动态库等），详情请看下文


 
# Buffer、cache简单概念
[点击打开链接](http://linuxperf.com/?p=32)
 
Page cache主要用来作为文件系统上的文件数据的缓存来用，尤其是针对当进程对文件有read／write操作的时候。下面的情况会使用到。
1. 作为可以映射文件到内存的系统调用（mmap
2. 作为其它文件类型的缓存设备来用
 
Buffer cache则主要是设计用来在系统对块设备进行读写的时候。
 
一般情况下Buffer cache、Page cache是一起配合使用的，比如当我们对一个文件进行写操作的时候，pagecache的内容会被改变，而buffercache则可以用来将page标记为不同的缓冲区，并记录是哪一个缓冲区被修改了。这样，内核在后续执行脏数据的回写（writeback）时，就不用将整个page写回，而只需要写回修改的部分即可
 


# 何时回收cache
- **内存将要耗尽的时候，触发内存回收的工作，以便释放出内存给急需内存的进程使用**。
- 伴随着cache清除的行为的，一般都是系统IO飙高。因为内核要对比cache中的数据和对应硬盘文件上的数据是否一致，如果不一致需要写回，之后才能回收。


 
# 手动清除cache的方法
- echo1 > /proc/sys/vm/drop_caches:表示清除pagecache。
- echo2 > /proc/sys/vm/drop_caches:表示清除回收slab分配器中的对象（包括dentries目录项缓存和inode缓存）。slab分配器是内核中管理内存的一种机制
- echo3 > /proc/sys/vm/drop_caches:表示清除pagecache和slab分配器中的缓存对象。


 
# 不能被回收的cache
1. Linux提供一种“临时”文件系统叫做tmpfs，它可以将内存的一部分空间拿来当做文件系统使用，使内存空间可以当做目录文件来用
如 /dev/shm的tmpfs目录
2. 共享内存
3. mmap申请标志状态为MAP_SHARED的内存
 
实际上shmget、mmap的共享内存，在内核层都是通过tmpfs实现的，tmpfs实现的存储用的都是cache
 
注意：新的发行版已经把共享内存等移到shared部分了
 

 
# Top命令(内存相关部分)
**VIRT**:进程地址空间的虚拟内存占用。已经映射到进程地址空间的内存(但这部分内存不一定分配了真实的物理内存，有可能没使用，有可能被换到了交换区)。可以通俗理解为程序中堆栈上定义的变量占用的内存、malloc调用分配的内存，使用shmat、mmap系统调用attach过的内存总和。这个值通常很大，进程实际占用的物理内存可能远小于虚拟内存(物理内存的实际分配通常会延迟到真正使用这块内存时：如变量赋值，memset)。
 
**RSS(RES)**(Residentset size): 进程实际正在使用的，没有被换出的物理内存。 (注：RSS也包含了正在使用的共享内存大小，并且共享内存大小会在多个attach这块共享内存的进程之间被重复计算) 

**SHR**:进程实际使用的共享内存(指attach过，并且实际使用了这块共享内存，如memset过)。

**PSS**:(Proportionalset sizes) : 进程实际使用的内存，与RSS最主要的区别在是共享内存大小的计算会在多个attatch进程之间均分。因此，PSS能比较好的衡量一个进程实际占用的物理内存大小。(PSS是开源工具sem最先提出的概念，由于很有用该工具已经被BSD内核的系统集成到内核。

cat /proc/pid/smaps  //也可查看PSS

![](/images/posts/2016-01-01-linux-memory-1/2.png)



# 如何查看系统共享内存
#### systemV共享内存
```shell
ipcs -m |grep -vP 'Shared|key' | awk 'BEGIN{sum=0}{sum+=$5;print $1"  --- " $5/1024/1024} END{print sum/1024/1024}'
ipcs -mu
```

```shell
[root@lang]#/usr/bin/ipcs -mu|/bin/egrep '^(segments allocated|pages allocated)'
segments allocated 322
pages allocated 1163012
```

systemV共享内存个数：322

systemV共享内存页数：1163012页（每页4K * 1163012 = 4652048K）

#### POSIX共享内存
```shell
ll -l /dev/shm/|grep -v 'total' | awk 'BEGIN{sum=0}{sum+=$5;}END{printsum/1024/1024/1024"G"}'
du -sm /dev/shm
```



# 如何查看单进程内存使用情况(pmap命令)
```shell
pmap -x 23809 | grep -P 'anon' | awk 'BEGIN{sum1=0;sum2=0} {sum1+=$2; sum2+=$3}END{printsum1" "sum2}'
```



# /proc/meminfo
<https://www.centos.org/docs/5/html/5.1/Deployment_Guide/s2-proc-meminfo.html>



# swap
#### 如何手动清除swap
```shell
swapoff -a && swapon –a
```

#### swap系统调参数
```shell
/proc/sys/vm/swappiness  10(60)
/proc/sys/vm/dirty_ratio 1(1)
/proc/sys/vm/dirty_background_ratio  1(1)
/proc/sys/vm/dirty_expire_centisecs  500(3000)
/proc/sys/vm/vfs_cache_pressure  500(100)
```
上述修改主要是为了
1. 当内存不够时，减少swap的使用，尽量清理cache
2. 当脏数据达到一定量、存在时间超过一定值，则尽快将数据同步回磁盘，减少cache的积压
<http://blog.163.com/czg_e/blog/static/461045612011224103942621/>
<http://blog.itpub.net/15480802/viewspace-753757/>


# valgrind
使用valgrind的massif可以查看程序动态使用内存的情况
```shell
valgrind --tool=massif ./你的二进制程序
ms_print massif.out.1285 > /tmp/a.txt //
```
![](/images/posts/2016-01-01-linux-memory-1/3.png)



