---
layout: post
title: /proc/stat
categories: Linux
description: /proc/stat
keywords: proc/stat, cpu使用效率, cpu总运行时间
---

探讨/proc/stat文件的各参数

```shell
[root@lang]#cat /proc/stat 
cpu  15579 99 13680 698457 10939 40 651 00
cpu0 1669 7 1974 338065 1396 5 9 0 0
cpu1 13910 91 11705 360391 9542 35 641 0 0
…//省略
```
![](/images/posts/2015-08-12-linux-proc-stat/1.png)

第一行表示的是CPU总的使用情况。其他行分别为各个核的使用情况。**单位均为jiffies**。
 
jiffies是内核中的一个全局变量，用来记录**自系统启动一来产生的节拍数**，在linux中，一个节拍大致可理解为操作系统进程调度的最小时间片，不同linux内核可能值有不同，通常在1ms到10ms之间。
 
下表解析第一行各数值的含义：
- user ( 15579 )      从系统启动开始累计到当前时刻，处于用户态的运行时间，不包含 nice值为负进程。
- nice (99)               从系统启动开始累计到当前时刻，nice值为负的进程所占用的CPU时间
- system (13680)    从系统启动开始累计到当前时刻，处于内核态的运行时间
- idle (698457)       从系统启动开始累计到当前时刻，除IO等待时间以外的其它等待时间
- iowait (10939)      从系统启动开始累计到当前时刻，IO等待时间
- irq (40)                 从系统启动开始累计到当前时刻，硬中断时间
- softirq (651)         从系统启动开始累计到当前时刻，软中断时间
- stealstolen(0) whichis the time spent in other operating systems when running in a virtualizedenvironment(since 2.6.11)
- guest(0)               whichis the time spent running a virtual  CPU  for  guest operatingsystems under the control of the Linux kernel(since 2.6.24)   
- intr:这行给出中断的信息，第一个为自系统启动以来，发生的所有的中断的次数；然后每个数对应一个特定的中断自系统启动以来所发生的次数。
- ctxt:给出了自系统启动以来CPU发生的上下文交换的次数。
- Btime：给出了从系统启动到现在为止的时间，单位为秒。
- processes(total_forks)：自系统启动以来所创建的任务的个数目。
- procs_running：当前运行队列的任务的数目。
- procs_blocked：当前被阻塞的任务的数目
     
**cpu总运行时间**= user + nice + system + idle + iowait + irq+ softirq + stealstolen +guest
     
**cpu使用效率**= (cpu总运行时间 - idle)/ cpu总运行时间


