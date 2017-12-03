---
layout: post
title: 数据库引擎
categories: 数据库
description: 
keywords: 
---




![](/images/posts/2017-10-02-mysql-engine.md/1.png)

![](/images/posts/2017-10-02-mysql-engine.md/2.png)


# 文件格式


MySQL通用（不管什么引擎）物理文件有slow log、query log、表定义文件frm。
 
InnoDB存储引擎的文件
- 系统总空间文件，一般是ibdata1/ibdata2（可配置）
- 用户表空间文件（需要开启 innodbfileper_table），存放的目录是：数据库目录/表名.ibd，如t1.ibd/t2.ibd...。
- redolog文件（默认存储在系统总空间文件ibdata1）、倒排文件（开启全文索引后会有）



# InnoDB和MyISAM的区别

InnoDB支持事务，而MyISAM是非事务引擎；
 
InnoDB的buffer pool可以缓存索引页和数据页，而MyISAM的key buffer只能缓存索引页，数据页依赖于OS的cache；
 
InnoDB可以实现行级锁，而MyISAM只能是表级锁。MyISAM的表级锁是个致命的缺陷，会严重降低数据库的并发性，甚至你只是需要update一个字段，整个表都会被锁起来，而别的进程，就算是读进程都无法操作
 
在5.6之前InnoDB不支持全文索引，5.6也只是支持英文的全文索引，而MyISAM全面支持全文索引；
 
InnoDB对硬件的利用率较高，如5.6可以使用到64个CPU核等，而MyISAM对硬件的利用率较差，如只能使用一个CPU等。
 
InnoDB表数据和索引同在一个表空间中，而MyISAM数据文件和索引文件分开存储；
 
InnoDB支持Crashrecovery，而MyISAM则不支持
 
5.5.8之前，默认的存储引擎就是MyISAM，但从5.5.8开始，InnoDB为默认的存储引擎
 
MyISAM引擎行结构模式比较紧凑，磁盘占用较少，高速insert或loaddata。支持全文索引（解决在我们需要用like查询的低效问题），统计表中数据量（count）快。但不支持事务
 
innoDB高并发读写，支持事务、外键。
 
索引区别

<http://blog.csdn.net/b2222505/article/details/78234666>




