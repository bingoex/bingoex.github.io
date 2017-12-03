---
layout: post
title: session的保存方式
categories: 系统架构 Java
description: 
keywords: 
---

# session的保存方式

## 1、保存在服务器的内存中
问题：单机内存是有限的、集群需解决session一致性问题（简单广播、TCP-Ring方式）


## 2、保存在单一数据源（如数据库）
问题：数据库性能、接口多样性不统一


## 3、保存在客户端
问题：长度限制、安全问题


## 4、组合法
将大部分session数据保存在cookie中，将小部分关键和涉及安全的数据保存在服务器上。由于我们只把少量关键的信息保存在服务端，因而服务器的压力不会非常大。

将session数据保存在cookie和Berkeley DB（或其它类似存储技术）中。


