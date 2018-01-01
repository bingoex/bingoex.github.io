---
layout: post
title: 【精品】浅谈数据库索引
categories: 数据库 优化
description: 
keywords: 
---


# InnoDB索引

索引组表（b+tree索引，自平衡，任意叶节点到根节点高度相同）。主键索引叶子节点存储了所有表数据（主键key＋表数据）。辅助索引存储key和主键值key。

![](/images/posts/2017-10-01-mysql-suoyin.md/1.png)



# MyISAM索引

堆组织表。主键索引只存储主键key和行指针，不存储实际的数据，实际表数据通过行指针指向。

![](/images/posts/2017-10-01-mysql-suoyin.md/2.png)



# 什么情况会使用索引

## 会用索引的情况

<、<=、=、>、>=、BETWEEN、IN、like、order by、group by、 ”xxxx%”是可以用到索引的，但尽量少用。

在使用索引字段作为条件时，如果该索引是复合索引，那么必须使用到该索引中的第一个字段作为条件时才能保证系统使用该索引（最左原则），否则该索引将不会被使用，并且应尽可能的让字段顺序与索引顺序相一致。
KEY `idx1` (`id`,`user`,`passwd`)

![](/images/posts/2017-10-01-mysql-suoyin.md/3.png)

## 不用索引的情况

select *、<>、not in、！=、like“%xxx”、

函数运算（如 where md5(password) = “xxxx”）、

WHERE index=1 OR A=10、

存了数值的字符串类型字段（如手机号），查询时记得不要丢掉值的引号、

表结构时应避免NULL的存在（用其他方式表达想表达的NULL，比如 -1或0？



# 什么情况适合建立索引

- 列的值唯一性太小（如性别，类型什么的），不适合建索引。
- 太长的列，可以选择只建立部分索引，（如：只取前十位做索引）。
- 更新非常频繁的数据不适宜建索引。
- 建议尽量减少对主键索引的更新,这样会导致所有辅助索引的更新。
- 数据的变更（增删改）都需要维护索引，因此更多的索引意味着更多的维护成本，更多的索引意味着也需要更多的空间。



# 其他

- 前缀索引
- alter table t1 add index idx_name(name(7)); ## 对name字段创建最前7个字符的索引
- select count(distinct Name)/count(*)from City;   ## name字段为char(35)
- select count(distinctleft(Name,7))/count(*) from City;##对比性能，比上面要更高效







