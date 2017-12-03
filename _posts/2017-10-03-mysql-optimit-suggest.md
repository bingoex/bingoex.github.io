---
layout: post
title: 数据库设计优化
categories: 数据库 优化
description: 
keywords: 
---



字段长度选择够用就好，越小越好。能用数值类型尽量使用数值类型，如果不需要用到变长类型的话，那么就统一采用char型。
 
TINYINT一般用于存储性别、是与否、用户状态之类等少数可选项（不建议使用enum枚举类型，扩展只能更改表的字段类型或ONLINE DDL、动作很大）
 
INT UNSIGNED可用于存储UNIX时间戳、IPV4地址（内置INET_ATON/INET_NTOA快速转换）
 
使用float时，一定要指定精度（alter table t6 modify num float(5,2)）
 
对于VARCHAR数据类型来说，硬盘上的存储空间虽然都是根据实际字符长度来分配存储空间的，但是对于内存来说，则不是。其实使用固定大小的内存块来保存值。
 
timestamp用4个字节，在5.6.4之后，datetime变为5个字节了，但范围却比timestamp大很多，所以在5.6中，还是建议使用datetime类型。
 
InnoDB每一个表都要显式设置主键。
 
主键越短越好，最好是自增类型，且由一个字段构成
 
字段属性尽量都加上NOT NULL约束，可一定程度提高性能；
1、浪费存储空间，因为InnoDB需要有额外一个字节存储；
2、表内默认值NULL过多会影响优化器选择执行计划。
 
尽可能不使用TEXT/BLOB类型，像图片、文件等大数据就不要存放在数据库中了，确实需要的话，建议拆分到子表中，不要和主表放在一起
 
在SELECT语句前放上关键词EXPLAIN，MySQL将解释它如何处理SELECT，提供有关表如何联接和联接的次序.
 
删除表中的所有数据，使用truncate table才是王道，比使用delete一行行删除数据要快得多，特别是清空大数据的表.
 
DDL通常是指添加、修改或删除表字段，添加索引等，这类操作通常是会直接锁表，导致大量的业务阻塞，甚至数据库宕机，尤其是添加索引操作
 
SET LOW_PRIORITY_UPDATES=1语句指定具体连接中的所有更新使用低优先级
 
可以使用 LOCK TABLES 来提高速度，因为在一个锁定中进行许多更新比没有锁定的更新要快得多.
 
索引优化

<http://blog.csdn.net/b2222505/article/details/78234666>


  
  

