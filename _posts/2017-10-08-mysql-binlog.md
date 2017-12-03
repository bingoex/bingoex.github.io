---
layout: post
title: Binlog详解
categories: 数据库
description: 
keywords: 
---


# 简介

Binlog（Binary Log）日志用于记录所有更新了数据或者以及潜在更新了数据（例如，没有匹配任何行的一个DELETE）。它记录了数据库的更改，所以我们可以利用binlog来对误操作的数据进行恢复，也可以用来进行主从数据库的同步，当然也可以用来监听和分发数据变更。



# Binlog的三种模式

Statement,ROW,MiXED

## Statement

statement（基于语句的复制）：每一条会修改数据的sql都会记录在binlog中。

优点：不需要记录每一行的变化，减少了binlog日志量，节约了IO，提高性能。

缺点：由于记录的只是执行语句，为了这些语句能在slave上正确运行，因此还必须记录每条语句在执行的时候的一些相关信息，以保证所有语句能在slave得到和在master端执行时候相同的结果。另外mysql的复制,像一些特定函数功能，slave可与master上要保持一致会有很多相关问题(如sleep()，rand()，last_insert_id()，user-definedfunctions(udf)，LOAD_FILE()、UUID()、USER()、FOUND_ROWS()、SYSDATE()等函数会出现问题).

## Row

row（基于行的复制）:不记录sql语句上下文相关信息，仅保存哪条记录被修改。

优点：非常清楚的记录下每一行数据修改的细节。而且不会出现某些特定情况下的存储过程，或function，以及trigger的调用和触发无法被正确复制的问题。

缺点:所有的执行的语句当记录到日志中的时候，都将以每行记录的修改来记录，这样可能会产生大量的日志内容,比如一条update语句，修改多条记录，则binlog中每一条修改都会有记录，这样造成binlog日志量会很大，特别是当执行alter table之类的语句的时候，由于表结构修改，每条记录都发生改变，那么该表每一条记录都会记录到日志中。

# mixed

mixed（混合方式），不会在从库产生歧义的语句只记录sql语句，会产生歧义的语句使用row方式，兼顾前两者的优点。



# Binlog相关命令及参数
```sql
show variables like 'log_%'
show binlog events;//binlog记录
show variables like 'binlog_format';//binlog格式
set binlog_format = xxx
reset master;//清空binlog
```

```xml
//my.cnf
log-bin=log_bin=/home/XXX/mysql-bin.log
expire_logs_days = 7 //binlog过期清理时间
max_binlog_size 100m //binlog每个日志文件大小
binlog_format = MIXED //binlog日志格式
```

查看命令:
```sql
./mysqlbinlog -v xxxx.bin
```




# Binlog拉取速度引起的问题

MySQL 主备之间数据同步是通过binlog进行的，当主库更新产生Binlog时，备库需要同步主库的数据，所以备库通过Binlog协议从主库拉取Binlog进行数据同步，以达到主备数据一致性的目的，但当主库tps较高时会产生大量的Binlog，以致备库拉取主库产生的Binlog时占用较多的网络带宽，引起以下问题：

1、在MySQL中，写Binlog与读Binlog使用的是同一把锁，频繁的读取Binlog，会加剧锁冲突，影响主库执行，进而造成TPS降低或抖动;

2、当备库数量较多时，备库拉取Binlog会占用过多的带宽，影响应用的响应时间；

## 解决方案:
可以通过设置Binlog的发送频率及休眼时间精确调整Binlog的发送速度



# sync_binlog

用来控制将binlog刷到磁盘的频率

默认为0，即mysql仅将binlog_cache中的数据写入Binlog文件，不会刷binlog到磁盘，则文件系统来刷新binlog。风险是如果操作系统或机器（不仅仅是Mysql服务器）崩溃，有可能binlog中最后的语句丢失。

sync_binlog=N，表示每N次事务提交，都会把binlog刷到磁盘；为1时，每次事务提交都会刷磁盘，增加磁盘IO；设置为1000，可以有效利用IO，但是可能出现binlog与磁盘数据不同步。

sync_binlog参数设置为0的时候，如果max_binlog_size设置很大的话，就会有概率出现性能大幅波动现象，不过也要取决于磁盘瞬时的吞吐能力。



# innodb_flush_log_at_trx_commit(innodb redo log)

![](/images/posts/2017-10-08-mysql-binlog.md/1.png)

- 当值为0时，每次事务提交时，日志写到InnoDB Log Buffer就立即返回。Log Buffer会每秒将日志写入到InnoDB日志文件并刷写到磁盘上。
- 当值为1时，每次事务提交时，日志写到InnoDB Log Buffer后，会等待Log Buffer中的日志写到Innodb日志文件并刷新到磁盘上才返回。（最慢最安全）
- 当值为2时，每次事务提交时，日志写到InnoDB日志文件（在OS的Pagecache中）就立即返回。日志文件的日志会每秒刷写一次到磁盘上。
 
 
 


