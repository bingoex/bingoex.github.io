---
layout: post
title: HBase简介
categories: 数据库 系统架构
description: 
keywords: 
---



# 简介

HBase – Hadoop Database，是一个高可靠性、高性能、面向列、可伸缩的分布式存储系统，利用HBase技术可在廉价PC Server上搭建起大规模结构化存储集群。



# 生态

- 利用Hadoop HDFS作为其文件存储系统。
- 利用Hadoop MapReduce来处理HBase中的海量数据。
- 利用Zookeeper作为协同服务

![](/images/posts/2017-09-01-hbase-instruct.md/1.jpg)




# 数据模型

## Table & Column Family

![](/images/posts/2017-09-01-hbase-instruct.md/1.jpeg)

- Row Key: 行键，Table的主键，Table中的记录按Row Key排序
- Timestamp: 时间戳，每次数据操作对应的时间戳，可以看作是数据的version number
- Column Family：列簇，Table在水平方向有一个或者多个Column Family组成，一个Column Family中可以由任意多个Column组成，所有Column均以二进制格式存储，用户需要自行进行类型转换。

row key为表的主键，是必然存在的一列，其次HBase表中的除row key外的列被划分为若干个列族(column family),列族下面是的一些列。

**数据按照Row key的字典序(byte order)排序存储**。设计key时，要充分排序存储这个特性，将经常一起读取的行存储放到一起。

hbase表中的每个列，都归属与某个列族。必须在使用表之前定义。列名都以列族作为前缀。例如courses:history，courses:math 都属于courses 这个列族。

cell中的数据是没有类型的，全部是字节码形式存贮。

针对一行的读与写操作都是原子的。

通常column family的数量比较少（百数量级），而且一般相对固定，通常在表创建之后就很难甚至不可能修改了。与此相对的column key一般没有数量限制，也可以动态增删。

```jason
"com.tx.lottery" : { // row key
    "issues" : { // first column family
        "ssq": {  // column key
            1 : "2014150" // single timestamp
        },      
        "dlt": "15015", // no timestamp
        "k3" : {
            10 : "20150301013", // multiple timestamp
             7  : "20150301010",
             1  : "20150301006"
        }           
    }
    "apps" : { // second column family
        "" : "hsf"  // empty column key
    }
    "fangInfo" : { // third column faimily
        "location":,
        "price":
    } 
}
```


## Table & Region

当Table随着记录数不断增加而变大后，会逐渐分裂成多份splits，成为regions，一个region由[startkey,endkey)表示，不同的region会被Master分配给相应的RegionServer进行管理：

![](/images/posts/2017-09-01-hbase-instruct.md/2.jpg)



## -ROOT- && .META. Table（0.96以前的版本）

HBase中有两张特殊的Table，-ROOT-和.META.

- .META.：记录了用户表的Region信息，.META.可以有多个regoin

- -ROOT-：记录了.META.表的Region信息，-ROOT-只有一个region

- Zookeeper中记录了-ROOT-表的location

![](/images/posts/2017-09-01-hbase-instruct.md/3.jpg)



Client访问用户数据之前需要首先访问zookeeper，然后访问-ROOT-表，接着访问.META.表，最后才能找到用户数据的位置去访问，中间需要多次网络操作，不过client端会做cache缓存。



# HBase系统架构

![](/images/posts/2017-09-01-hbase-instruct.md/4.jpg)

## Client
HBase Client使用HBase的RPC机制与HMaster和HRegionServer进行通信。对于管理类操作，Client与HMaster进行RPC；对于数据读写类操作，Client与HRegionServer进行RPC。



## Zookeeper
Zookeeper Quorum中除了存储了-ROOT-表的地址和HMaster的地址，HRegionServer也会把自己以Ephemeral方式注册到Zookeeper中，使得HMaster可以随时感知到各个HRegionServer的健康状态。此外，Zookeeper也避免了HMaster的单点问题。



## HMaster
HMaster没有单点问题，HBase中可以启动多个HMaster，通过Zookeeper的Master Election机制保证总有一个Master运行，HMaster在功能上主要负责Table和Region的管理工作：

- 管理用户对Table的增、删、改、查操作
- 管理HRegionServer的负载均衡，调整Region分布
- 在Region Split后，负责新Region的分配
- 在HRegionServer停机后，负责失效HRegionServer 上的Regions迁移


## HRegionServer
Client直接通过HRegionServer读写数据（从HMaster中获取元数据，找到RowKey所在的HRegion/HRegionServer后）

HRegionServer主要负责响应用户I/O请求，向HDFS文件系统中读写数据，是HBase中最核心的模块。HRegionServer内部管理了一系列HRegion对象，每个HRegion对应了Table中的一个Region，HRegion中由多个HStore组成。每个HStore对应了Table中的一个Column Family的存储。

HStore存储是HBase存储的核心了，其中由两部分组成，一部分是MemStore，一部分是StoreFiles。MemStore是Sorted Memory Buffer，用户写入的数据首先会放入MemStore，当MemStore满了以后会Flush成一个StoreFile（底层实现是HFile），当StoreFile文件数量增长到一定阈值，会触发Compact合并操作，将多个StoreFiles合并成一个StoreFile，合并过程中会进行版本合并和数据删除，因此可以看出HBase其实只有增加数据，所有的更新和删除操作都是在后续的compact过程中进行的，这使得用户的写操作只要进入内存中就可以立即返回，保证了HBase I/O的高性能。当StoreFiles Compact后，会逐步形成越来越大的StoreFile，当单个StoreFile大小超过一定阈值后，会触发Split操作，同时把当前Region Split成2个Region，父Region会下线，新Split出的2个孩子Region会被HMaster分配到相应的HRegionServer上，使得原先1个Region的压力得以分流到2个Region上。下图描述了Compaction和Split的过程：

![](/images/posts/2017-09-01-hbase-instruct.md/5.gif)



在理解了上述HStore的基本原理后，还必须了解一下HLog的功能，因为上述的HStore在系统正常工作的前提下是没有问题的，但是在分布式系统环境中，无法避免系统出错或者宕机，因此一旦HRegionServer意外退出，MemStore中的内存数据将会丢失，这就需要引入HLog了。每个HRegionServer中都有一个HLog对象，HLog是一个实现Write Ahead Log的类，在每次用户操作写入MemStore的同时，也会写一份数据到HLog文件中（HLog文件格式见后续），HLog文件定期会滚动出新的，并删除旧的文件（已持久化到StoreFile中的数据）。当HRegionServer意外终止后，HMaster会通过Zookeeper感知到，HMaster首先会处理遗留的 HLog文件，将其中不同Region的Log数据进行拆分，分别放到相应region的目录下，然后再将失效的region重新分配，领取 到这些region的HRegionServer在Load Region的过程中，会发现有历史HLog需要处理，因此会Replay HLog中的数据到MemStore中，然后flush到StoreFiles，完成数据恢复。

![](/images/posts/2017-09-01-hbase-instruct.md/6.png)

![](/images/posts/2017-09-01-hbase-instruct.md/7.png)



## HFile

![](/images/posts/2017-09-01-hbase-instruct.md/8.jpg)

首先HFile文件是不定长的，长度固定的只有其中的两块：Trailer和FileInfo。Trailer中有指针指向其他数据块的起始点。File Info中记录了文件的一些Meta信息，例如：AVG_KEY_LEN, AVG_VALUE_LEN, LAST_KEY, COMPARATOR, MAX_SEQ_ID_KEY等。Data Index和Meta Index块记录了每个Data块和Meta块的起始点。

Data Block是HBase I/O的基本单元，为了提高效率，HRegionServer中有基于LRU的Block Cache机制。每个Data块的大小可以在创建一个Table的时候通过参数指定，大号的Block有利于顺序Scan，小号Block利于随机查询。每个Data块除了开头的Magic以外就是一个个KeyValue对拼接而成, Magic内容就是一些随机数字，目的是防止数据损坏。后面会详细介绍每个KeyValue对的内部构造。

HFile里面的每个KeyValue对就是一个简单的byte数组。但是这个byte数组里面包含了很多项，并且有固定的结构。我们来看看里面的具体结构：

![](/images/posts/2017-09-01-hbase-instruct.md/9.jpg)


开始是两个固定长度的数值，分别表示Key的长度和Value的长度。紧接着是Key，开始是固定长度的数值，表示RowKey的长度，紧接着是RowKey，然后是固定长度的数值，表示Family的长度，然后是Family，接着是Qualifier，然后是两个固定长度的数值，表示Time Stamp和Key Type（Put/Delete）。Value部分没有这么复杂的结构，就是纯粹的二进制数据了。








# HBase表特点

- 大：一个表可以有上亿行，上百万列
- 面向列:面向列(族)的存储和权限控制，列(族)独立检索。
- 稀疏:对于为空(null)的列，并不占用存储空间，因此，表可以设计的非常稀疏。



# 常用API

```java
Configuration config = new Configuration();
config.set(HConstants.ZOOKEEPER_CLIENT_PORT, port);
config.set(HConstants.ZOOKEEPER_QUORUM, quorum);
config.set(HConstants.ZOOKEEPER_ZNODE_PARENT, cluster);
config = HBaseConfiguration.create(config);
HTable hTable = new HTable(config, "lottery_tb_order");


HBaseAdmin hBaseAdmin = new HBaseAdmin(config);
HTableDescriptor[] hTableDescriptors = hBaseAdmin.listTables();


//创建配置
Configuration configuration = HBaseConfiguration.create();
HBaseAdmin admin = new HBaseAdmin(configuration);

// 新建表的基本信息，包含表名，column family信息等
HTableDescriptor tableDescriptor = new TableDescriptor(TableName.valueOf("lottery_info"));
tableDescriptor.addFamily(new HColumnDescriptor("issues"));
tableDescriptor.addFamily(new HColumnDescriptor("apps"));
tableDescriptor.addFamily(new HColumnDescriptor("fangInfo"));

// 执行建表
admin.createTable(tableDescriptor);



HColumnDescriptor columnDescriptor = new HColumnDescriptor("matches");
admin.addColumn("lottery_info", columnDescriptor);
admin.deleteColumn("lottery_info","fangInfo");



Configuration configuration = HBaseConfiguration.create();
HBaseAdmin admin = new HBaseAdmin(configuration);

String tableName = "lottery_info";
if (admin.tableExists(tableName) && !admin.isTableDisabled(tableName)) {
    admin.disableTable(tableName);
    admin.deleteTable(tableName);
}



Configuration configuration = HBaseConfiguration.create();
HTable table = new HTable(configuration, "lottery_info");
String rowKey = "com.tx.lottery";
String columnFamily = "issues";
String columnKey = "ssq";
String data = "2014150";

// 插入或更新一行
Put put = new Put(Bytes.toBytes(rowKey));
put.add(Bytes.toBytes(columnFamily), Bytes.toBytes(columnKey), Bytes.toBytes(data));
table.put(put);

// 查询一行
Get get = new Get(Bytes.toBytes(rowKey));
Result result = table.get(get);
byte[] value = result.getValue(Bytes.toBytes(columnFamily), Bytes.toBytes(columnFamily));



Configuration configuration = HBaseConfiguration.create();
HTable table = new HTable(configuration, "lottery_info");
String rowKey = "com.tx.lottery";
String columnFamily = "issues";
String columnKey = "ssq";

// 删除一行
Delete delete = new Delete(rowKey.getBytes());
table.delete(delete);

// 删除column下的最新版本数据
Delete specifiedDelete = new Delete(rowKey.getBytes());
specifiedDelete.addColumn(columnFamily.getBytes(), columnKey.getBytes());
table.delete(specifiedDelete);
```







