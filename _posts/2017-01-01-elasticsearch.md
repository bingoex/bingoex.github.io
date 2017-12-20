---
layout: post
title: ElasticSearch简介
categories: Java 系统架构
description: 
keywords: 
---


ElasticSearch是一个开源的、可扩展、分布式的搜索和分析系统。

# 适用场景

- 不需要事务支持。
- 可以适应准实时的查询特点，注意ES 不是实时查询的数据库。
- 需要支持多维度查询的场景，即需要在很多字段进行过滤，比如数十、数百甚至更大的字段中选择过滤。
- 需要支持庞大数据量的聚合、排序、统计的要求，但对并发请求要求不高。
- 需要支撑千万、甚至数百亿以上的数据查询。



# 基本概念

集群（Cluster），有共同的 cluster name 的节点,通过配置一个相同的cluster name，互相发现，把自己组织成一个集群

节点（Node) ，集群中 Elasticearch 实例。一台机器上可以运行多个node，而一个node可以拥有多个shard（分片）。值得注意的是node和JVM实例一一对应，而shard虽然是一个完整的Lucene实例，但运行在一个node中的多个shard是共享一个JVM环境的。

索引（Index) ，等同于关系数据库中的database，集群可以有多个索引。常见思路是按时间纬度（如月）去定义ES索引。

主分片（Primary shard），一个索引数据可划分成多个shard（分片），相互shard之间无交集。分布到不同的集群节点上。分片对应的是 Lucene 中的索引。**注意：ES中一个索引的分片个数是建立索引时就要指定的，建立后不可再改变。所以开始建一个索引时，就要预计数据规模，将分片的个数分配在一个合理的范围**。最好是和节点数相关的。理论上对同一个索引，单机上的shards个数最好不要超过两个，这样每个查询尽可能并行。

副本分片（Replica shard），每个主分片可以有一个或者多个副本。ES会尽量将同一索引的不同分片分布到不同的节点上，提高容错性。

![](/images/posts/2017-01-01-elasticsearch.md/1.png)


类型（Type），等同于数据库中的table概念。同一个索引里可以包含多个 Type。

映射（Mapping），等同于 数据库中的schema。

文档（Document)，等同于 数据库中的行。

字段（Field），等同于数据库中的列。


![](/images/posts/2017-01-01-elasticsearch.md/2.jpeg)


# ES的系统结构

ES系统的源码和设计层次划分很清晰，根据系统实现对ES整体上的模块（Module）划分主要分为传输层、集群自身管理、节点请求（接收、分发请求）、数据读写、底层存储引擎、周边模块几大类。

![](/images/posts/2017-01-01-elasticsearch.md/3.jpeg)

## 传输层：
ES的传输层负责接收用户的请求、集群内部节点相互之间进行请求和数据传输等，ES内部可开启（开关）支持用户的http请求，请求和返回都是按json格式组织。同时，ES也可直接走ES内部的传输协议。对于网络传输，ES采用了开源的java网络框架netty，支持NIO，性能和稳定性上都很不错。

## 集群层：
ES启动（bootstrap）后，会通过guice查找依赖并注入各类实例，然后启动各个模块的入口函数。而集群层就是各模块启动后开始首先做事的层级。比如，Discover模块会启动节点Ping服务探测、选举Master、处理节点加入、publish集群信息变更等。cluster模块管理集群的元数据信息，包括有哪些节点、索引、路由等，集群信息变更会通知cluster模块变更元数据信息。一旦启动或者集群发生变更，Gateway会进行数据的恢复行为。indices模块负责索引（不是数据）的管理，包括增删查改。snapshot用户的备份请求，可以按集群、索引的粒度进行备份。

## 请求管理：
对与用户请求开不同的client实例进行请求收拢，因为对于，ES内部按请求类型分ES节点来说，它自身需要根据路由请求不同的数据节点，因此，本身来说自身就是一个client角色。client会根据请求调用底层的Action层，这层会根据路由进行请求不同的节点、或者本地节点，数据的写入路由、查询合并都在Action层完成。

## 读写：
读写层根据类型分开不同，Index层负责写入数据、写translog、刷新缓存、重启恢复、合并等。Script模块支持用户的脚步读写，例如js/mvel等。Search模块提供用户查询、聚合、排序、查询预热等读请求。

## 引擎层：
该层实际是封装了lucene，目的是为上层提供一个规范的engine接口。同时，为底层选择除lucene以外的存储引擎提供可能。和mysql的底层数据库引擎、VFS下面的文件系统的模型类似。

## 周边模块：
ES系统架构里面还包括监控、告警、缓存、线程持这些周边模块。这些模块为上层模块提供支持。



# 常用组合

![](/images/posts/2017-01-01-elasticsearch.md/4.jpeg)





# 数据写入

客户端实际上是不知道Master在哪个节点上的，它对于shard的路由情况当然也是一无所知。为了解决这个问题，Elasticsearch引入了coordinator(协调者)的概念。客户端如果需要写入一个document，可以把请求发往任意的一个Elasticsearch节点，那么收到请求的这个节点，就是当前的coordinator(步骤1)。通过扫描这条document, coordinator发现该数据应该写入标号为0的shard上，而shard 0对应的primary在NODE3上面，所以它会把请求原封不动的转发给NODE3(步骤2)。Node3先把数据写入P0中，但PO还有两个replica小伙伴分别在NODE1和NODE2上，所以NODE3与会把复制请求发送到NODE1和NODE2，并等待它们的回应(步骤3)。NODE1和NODE2把数据写入R0后，会回复NODE3，然后NODE3回复coordinatorNODE1，最后客户端得到了写入成功的信息。(当然跟据写入请求consisteny level设置的不同，NODE3在写入P0成功后，有可能不等待NODE1和NODE2的回应，就直接告诉P1数据已经写入成功了)


![](/images/posts/2017-01-01-elasticsearch.md/5.png)


# 通过id读取

通过id读取单个document的过程是比较简单的(实际上这里指的id是_id与type的组合)，以下图为例，NODE1收到客户端的请求后，发现此document分布在shard 0上。shard 0有三个实例，其中1个是master，另外2个为replica，它们承载的数据都是一模一样的，到底应该从哪个实例上去读取这个document呢？实际上这里会有负载均衡的算法来均匀的分发请求，在这个例子中，coordinator把请求发给了NODE2身上的R0。

 
![](/images/posts/2017-01-01-elasticsearch.md/6.png)


# 搜索

通过其他条件来搜索数据的过程，会比较复杂一些，因为对于任何一个节点来说，当收到一个搜索请求的时候，它完全不知道数据分布在哪一些shard上(只有确定id的document，才能立即找到确定的shard)。Elasticsearch通过2个阶段来完成搜索的任务，这个解决方案在官方被称为"query thenfetch"
 
Query阶段：假设NODE3接收到一个搜索请求，它会转发请求到其他的节点上，保证每一个标号的shard都能收到搜索请求，在这个例子中，NODE3作为coordinator决定把请求发给NODE2身上的R0和NODE1身上的P1，让这两个shard在自己身上完成搜索任务(步骤1和步骤2) P1和R0完成自己的这一部分搜索任务后，把对应的结果集回复给NODE3，但这个结果集中仅包含文档id和要排序的字段信息，并没有包含所有的field (步骤3)

![](/images/posts/2017-01-01-elasticsearch.md/7.png)

Fetch阶段：Node3收到结果集后，进行一次排序操作，并丢充掉超出范围的结果集，重新向NODE2身上的R0与NODE1身上的P1发起fetch请求获取全量数据，然后返回给客户端。

![](/images/posts/2017-01-01-elasticsearch.md/8.png)


 
 
 
 
 
 
 

