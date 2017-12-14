---
layout: post
title: 数据库范式
categories: 数据库
description: 
keywords: 
---



# 数据库范式

1NF（First Normal Form）：当且仅当所有域只包含原子值（**字段不可分**）

2NF（Second Normal Form）：当且仅当实体E满足第一范式，且每一个非键属性完全依赖主键时（**有主键，非主键依赖主键**）（**非主属性不能部分依赖于主键**（eg：**只依赖某个主键**）

3NF（Third Normal Form）：当且仅当实体E是第二范式（2NF），且E中没有非主属性传递依赖时（**非主键不能相互依赖**）

BCNF （**主键列之间，不存在依赖**）


# 范式判断流程图

![](/images/posts/2017-10-07-database-fanshi.md/1.png)



# 一、第一范式（1NF）

**列不可分**。每一列都是不可分割的基本数据项。

反例：

StudyNo |  Name | Sex  | Contact
20040901  |   john        | Male     | Email:kkkk@ee.NET,phone:222456
20040902  |   mary        | famale   | email:kkk@fff.Net phone:123455

contact字段可以再分，不符合第一范式。

正解：

StudyNo | Name  | Sex  | Email | Phone
20040901    | john        | Male     | Email:kkkk@ee.net | 123123
20040902     | mary        | famale   | email:kkk@fff.net | 123455



# 二、第二范式（2NF）

在第一范式基础上，**对于多关键字表，非主属性不能部分依赖于主键**（eg：只依赖某个主键）

因为我们知道在一个订单中可以订购多种产品，所以单单一个 OrderID 是不足以成为主键的，主键应该是（OrderID，ProductID）。显而易见Discount（折扣），Quantity（数量）完全依赖（取决）于主键（OderID，ProductID），而UnitPrice，ProductName 只依赖于 ProductID。所以 OrderDetail 表不符合 2NF。不符合 2NF 的设计容易产生冗余数据。

可以把【OrderDetail】表拆分为【OrderDetail】（OrderID，ProductID，Discount，Quantity）和【Product】（ProductID，UnitPrice，ProductName）来消除原订单表中UnitPrice，ProductName多次重复的情况。

反例：

StudyNo | Name  | Sex  | Email | Phone | ClassNo | ClassAddress
20040901   | john     |    Male      | Email:kkkk@ee.net  | 222456 | 200401 | #12A
20040902   | mary       | famale  | email:kkk@fff.net | 123455 | 200402 | #8A

主键是studyNo和classNo。classAddress部分依赖主键classNo，需要变为两个表。

正解：

学生表|  StudyNo | Name   | Sex   | Email |  Phone
20040901      | john        |  Male      | Email:kkkk@ee.net |  222456
20040902      | mary         | famale    | email:kkk@fff.net |  123455

教室表

ClassNo  | ClassAddress
200401 |  #12A
200402 |  #8A

优点： **消除数据冗余和增、删、改异常**。
- 数据冗余：同一门课程由n个学生选修，"学分"就重复n-1次；同一个学生选修了m门课程，姓名和年龄就重复了m-1次。
- 更新异常：若调整了某门课程的学分，数据表中所有行的"学分"值都要更新，否则会出现同一门课程学分不同的情况。
- 插入异常：假设要开设一门新的课程，暂时还没有人选修。这样，由于还没有"学号"关键字，课程名称和学分也无法记录入数据库。
- 删除异常：假设一批学生已经完成课程的选修，这些选修记录就应该从数据库表中删除。但是，与此同时，课程名称和学分信息也被删除了。很显然，这也会导致插入异常。


# 三、 第三范式（3NF）

在第二范式基础上，非主键列必须直接依赖于主键，不能传递依赖。即不能存在：非主键列 A 依赖于非主键列 B，非主键列 B 依赖于主键的情况。总之，**非主键列之间不能存在依赖关系**。

其中 OrderDate，CustomerID，CustomerName，CustomerAddr，CustomerCity 等非主键列都完全依赖于主键（OrderID），所以符合 2NF。不过问题是 CustomerName，CustomerAddr，CustomerCity 直接依赖的是 CustomerID（非主键列），而不是直接依赖于主键，它是通过传递才依赖于主键，所以不符合 3NF。

通过拆分【Order】为【Order】（OrderID，OrderDate，CustomerID）和【Customer】（CustomerID，CustomerName，CustomerAddr，CustomerCity）从而达到 3NF。

反例：

StudyNo |  Name   | Sex   | Email |  Phone |  BounsLevel |  Bouns
20040901      | john         | Male     |  Email:kkkk@ee.net |  222456 |  优秀 |  ￥1200
20040902     |  mary         | famale    | email:kkk@fff.net |  123455 |  良 |  ￥800

主键是StudyNo，只有一个主键StudyNo，而且符合第二范式。但是非主键列bounsLevel和bouns存在依赖关系。

正解：

学生表

StudyNo  | Name   | Sex   | Email |  Phone |  BounsNo
20040901      | john         | Male      | Email:kkkk@ee.net |  222456 |  1
20040902      | mary         | famale    | email:kkk@fff.net |  123455 |  2

奖学金等级表

BounsNo  | BounsLevel |  Bouns
1 |  优秀 |  ￥1200 
2 |  良 |  ￥800

优点：**消除数据冗余和增、删、改异常**。


# 四、鲍依斯-科得范式（BCNF）

在第三范式的基础上，数据库表中如果不存在任何字段对任一候选关键字段的传递函数依赖则符合第三范式。**即不存在关键字段决定关键字段的情况**。

反例：

StoreHouseID（仓库ID） |  GoodsID（商品ID） |  ManagerID（管理员ID） |  GoodsNum（商品数量）
001 |  20130104 |  1  | 200

主键是

(仓库ID, 商品ID) →(管理员ID, 数量) ，(管理员ID, 商品ID) → (仓库ID, 数量)，(仓库ID, 商品ID)和(管理员ID,商品ID)都是候选关键字，表中的唯一非关键字段为数量。满足第三范式。但是，存在关键字段决定关键字段情况。(仓库ID) → (管理员ID)和(管理员ID) → (仓库ID)

正解：

StoreHouseID（仓库ID） |  GoodsID（商品ID） |  GoodsNum（商品数量）
001 |  20130104 |  200

StoreHouseID（仓库ID） |  ManagerID（管理员ID）
001 |  1

优点： **消除增、删、改异常**。

- 删除异常：当仓库被清空后，所有"存储物品ID"和"数量"信息被删除的同时，"仓库ID"和"管理员ID"信息也被删除了。
- 插入异常：当仓库没有存储任何物品时，无法给仓库分配管理员。
- 更新异常：如果仓库换了管理员，则表中所有行的管理员ID都要修改。

# 总结

**应用的范式等级越高，则表越多**。

表多会带来很多问题：
- 查询时需要连接多个表，增加了查询的复杂度
- 查询时需要连接多个表，降低了数据库查询性能
- 并不是应用的范式越高越好，要看实际情况而定。
- 第三范式已经很大程度上减少了数据冗余，并且减少了造成插入异常，更新异常，和删除异常了。所以大多数情况应用到第三范式已经足够。

[更多数据库设计优化请看](https://bingoex.github.io/2017/10/03/mysql-optimit-suggest/)

