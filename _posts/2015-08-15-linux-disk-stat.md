---
layout: post
title: linux磁盘浅析
categories: Linux
description: linux磁盘浅析
keywords: df, /proc/diskstats, 文件大小
---

分析linux磁盘的使用情况及其参数

# df查看磁盘使用情况 
```shell
[root@lang]#df –h
Filesystem            Size  Used Avail Use% Mounted on
/dev/sda1             9.9G  6.2G 3.3G  66% /
/dev/sda3              20G  1.5G  18G   8% /usr/local
/dev/sda4             427G  213G 193G  53% /data
tmpfs                 7.0G  2.4G 4.7G  34% /dev/shm
```



# /proc/diskstats  
```shell
[root@lang]# cat /proc/diskstats
1       0 ram0 0 0 0 0 0 0 0 0 0 0 0
1       1 ram1 0 0 0 0 0 0 0 0 0 0 0
1       2 ram2 0 0 0 0 0 0 0 0 0 0 0
1       3 ram3 0 0 0 0 0 0 0 0 0 0 0
1       4 ram4 0 0 0 0 0 0 0 0 0 0 0
1       5 ram5 0 0 0 0 0 0 0 0 0 0 0
1       6 ram6 0 0 0 0 0 0 0 0 0 0 0
1       7 ram7 0 0 0 0 0 0 0 0 0 0 0
1       8 ram8 0 0 0 0 0 0 0 0 0 0 0
1       9 ram9 0 0 0 0 0 0 0 0 0 0 0
1      10 ram10 0 0 0 0 0 0 0 0 0 0 0
1      11 ram11 0 0 0 0 0 0 0 0 0 0 0
1      12 ram12 0 0 0 0 0 0 0 0 0 0 0
1      13 ram13 0 0 0 0 0 0 0 0 0 0 0
1      14 ram14 0 0 0 0 0 0 0 0 0 0 0
1      15 ram15 0 0 0 0 0 0 0 0 0 0 0
7       0 loop0 0 0 0 0 0 0 0 0 0 0 0
7       1 loop1 0 0 0 0 0 0 0 0 0 0 0
7       2 loop2 0 0 0 0 0 0 0 0 0 0 0
7       3 loop3 0 0 0 0 0 0 0 0 0 0 0
7       4 loop4 0 0 0 0 0 0 0 0 0 0 0
7       5 loop5 0 0 0 0 0 0 0 0 0 0 0
7       6 loop6 0 0 0 0 0 0 0 0 0 0 0
7       7 loop7 0 0 0 0 0 0 0 0 0 0 0
8       0 sda 24702152 11675882 487860316 265032360 1780849176 4727436837 52211385389 3596260696 0 2160245748 3877486488
8       1 sda1 16162238 9301970 350983740 171565848 98471941 380849008 3835456680 2092126948 0 505368456 2263626308
8       2 sda2 101760 215213 2533050 448816 96977 858864 7646760 10647488 0 823036 11097568
8       3 sda3 2041370 888489 60477760 20150096 141601539 335551868 3819740785 2809846016 0 700529280 2829966960
8       4 sda4 6391293 1269456 73815806 72814676 1540678719 4010177097 44548541164 2978607540 0 2001189960 3067708736
9       0 md0 0 0 0 0 0 0 0 0 0 0 0
```
sda为整个硬盘的统计信息，sda1为第一个分区的统计信息，sda2为第二个分区的统计信息。

ramdisk设备为通过软件将RAM当做硬盘来使用的一项技术。

以26行为例子：
- 8：主设备号
- 0：次设备号
- sda：设备名
- Filed0:24702152：**成功完成读的总次数**
- Filed1:11675882：合并读完成的总次数。为了效率可能会合并相邻的读和写。从而两次4K的读在它最终被处理到磁盘上之前可能会变成一次8K的读，才被计数（和排队），因此只有一次I/O操作。
- Filed2:487860316:**成功完成读的扇区总次数**。（一个扇区512字节）
- Filed3:265032360：所有读操作所花费的毫秒数。
- Filed4:1780849176：**成功写完成的总次数**
- Filed5:4727436837：合并写完成的总次数。
- Filed6:52211385389：**成功写扇区总次数**。
- Filed7:3596260696：所有写操作所花费的毫秒数。
- Filed8:0：I/O的当前进度，只有这个域应该是0。当请求被交给适当的request_queue_t时增加和请求完成时减小
- Filed9:700529280：**number of milliseconds spent doing I/Os. Thisfield is increased so long as field 8 is nonzero**
- Filed10:2829966960：number of milliseconds spent doing I/Os. Thisfield is incremented at each I/O start, I/O completion, I/O merge, or read ofthese stats by the number of I/Os in progress (field 8) times the number ofmilliseconds spent doing I/O since the last update of this field. This canprovide an easy measure of both I/O completion time and the backlog that may beaccumulating.

**sda平均每次I/O操作的等待时间(微秒)**= (Filed3 + Filed7) * 1000 / (Filed0 + Filed4)

**sda平均每次I/O操作的服务时间(微秒)** = Filed9 * 1000 / (Filed0 + Filed4)



# 查看某目录的文件大小
```shell
du -sh
```



# 查看文件句柄参数
```shell
/proc/sys/fs/file-nr
[root@lang]#cat /proc/sys/fs/file-nr
8896    0      805153
```
8896：**已分配文件句柄数**

0：已分配未使用文件句柄数

805153：系统最大文件句柄数





