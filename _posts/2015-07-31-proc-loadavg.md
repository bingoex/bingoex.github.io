---
layout: post
title: /proc/loadavg
categories: Linux
description: some word here
keywords: loadavg, 系统负载, 系统load
---

# 一、概念：系统平均负载(Load average)
系统平均负载:在特定时间间隔内运行队列中的平均进程数。

# 二、例子
[0] %cat /proc/loadavg 

0.32 0.29 0.13 1/357 1909

前三个是1、5、15分钟内的平均进程数。第四个的分子是正在运行的进程数，分母是进程总数；最后一个最近运行的进程ID号

一般来说每个CPU的当前活动进程数不大于3那么系统的性能就是良好的。如果每个CPU的任务数大于5，那么就表明机器的性能有严重问题。
对于上面的例子来说，假设系统有8个CPU，那么其每个CPU在1分钟内的进程数为：0.32/8=0.04。

# 三、查看系统平均负载的常用命令
1. cat /proc/loadavg
2. uptime
3. w
4. top
5. tload
