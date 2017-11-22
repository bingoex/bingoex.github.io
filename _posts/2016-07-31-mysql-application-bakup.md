---
layout: post
title: Mysql备份-简单应用篇
categories: 数据库
description: Mysql备份-简单应用篇
keywords: 数据库备份, Mysql备份, 备份
---

最近做mysql数据库备份和搭建。固作此文以此记录。

# 一、数据库备份方式简单介绍

数据库存放目录在配置文件/etc/my.cnf下的datadir参数进行配置。每个数据库名对应一个文件夹（如 create database db_name，那么就能在数据存放目录下找到db_name目录.。表相关文件要看具体的引擎，如果是MyISAM引擎，每个表会对应有tbl_name.frm、tbl_name.myi、tbl_name.myd三个文件。Innodb引擎则对应一个tbl_name.frm结构文件，数据和索引文件都存放在 ibdata1表空间里。

#### 1、裸备份
直接拷贝磁盘文件，也是最直接、最快速的方式。但受限于数据库引擎，目前只支持MyISAM引擎间备份


#### 2、mysqldump
原理：mysqldump采用SQL级别的备份机制，它将数据表导成 SQL 脚本文件（不包含数据库创建语句），在不同的 MySQL 版本之间备份时相对比较兼容，是最常用的备份方法之一。

##### 备份：
-  mysqldump -hhost -uuser -p db_name > sql文件（xxx.sql）

##### 还原：
- mysql -hhost-uuser -p db_name（要还原到的数据库,这个数据库事先创建） < sql文件
- 进入mysql，执行source sql文件


#### 3、mysqlhotcopy

原理：mysqlhotcopy 是PERL 程序，由Tim Bunce编写，使用 LOCK TABLES、FLUSH TABLES 和 cp 或 scp 来快速备份数据库。它是备份数据库或单个表的最快的途径，但它只能运行在数据库文件（包括数据表定义文件、数据文件、索引文件）所在的机器上且只能用于备份 MyISAM引擎数据。

##### 备份：
mysqlhotcopy -h=localhost -u=user -p=user_password db_name /tmp (把数据库目录 db_name 拷贝到 /tmp 下)

##### 还原：
直接拷贝到对应的数据存储目录


#### 4、SQL 语法备份（支持不同分割符）
原理：锁表，然后拷贝数据文件。它能实现在线备份，但是效果不理想，不推荐使用。它只拷贝表结构文件和数据文件，不同时拷贝索引文件。

##### 备份：
- BACK TABLE tbl_name TO '/tmp/db_name/';
- SELECT * INTO OUTFILE '/tmp/db_name/tbl_name.txt' FROM tbl_name;

##### 恢复：
- RESTORE TABLE FROM '/tmp/db_name/';
- LOAD DATA INFILE '/tmp/db_name/tbl_name.txt' INTO TABLE tbl_name;


#### 5、binlog备份
支持增量备份，待续

##### 备份：
- 拷贝 binlog.XXXX 以及 binlog.index

##### 恢复：
- mysqlbinlog /tmp/binlog.XXXX | mysql -uuser  -puser_password db_name



# 二、备份过程细节记录

因我们业务数据库均为MyISAM引擎，且源机器上无多余空间和新增数据，固选择裸备份方式，通过rsycn将数据进行全量备份。

## 1、相关命令及参数

#### -A参数:
```sql
mysql -uuser -puser_password -A
```
如果数据库太大，预读数据库信息（表等）将非常慢。使用-A参数时，可以更快地启动mysql。

#### 查看数据库引擎类型：
```sql
show engines
show variables like '%storage_engine%';
```

#### 检测数据库是否已经存在脚本：
```shell
SQL_EXE="mysql -u local ${DB}"
echo "exit" | $SQL_EXE
if [ ! $? -eq 0 ]; then
　　echo "Notice:NO databases:${DB}; Need to create ${DB}!"
Fi
```

#### 执行sql脚本：
```sql
mysql -uuser -puser_password < XXX.sql
```

#### 创建数据库相关语句：
```sql
set names utf8;
create database IF NOT EXISTS XXXX DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci
```
[编码问题](https://stackoverflow.com/questions/766809/whats-the-difference-between-utf8-general-ci-and-utf8-unicode-ci)

#### 用户授权相关语句：
<http://7567567.blog.51cto.com/706378/710159>

#### 创建空白用户
```sql
GRANT USAGE ON *.* TO 'user1'@'%' IDENTIFIED BY PASSWORD 'XXX';
GRANT SELECT ON `db_name`.*`tbl_name1`TO 'user1'@'%'
```

#### 创建用户并授权
```sql
GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' IDENTIFIED BY PASSWORD 'XXX' WITH GRANT OPTION;
```

#### 刷新权限
```sql
flush privileges;
```

#### Mysql服务器重启命令:
```shell
service mysqld restart
```


## 2、Rsync相关：

#### 配置文件：
/etc/rsyncd.conf 

#### 配置文件Demo：
```xml
gid = users
read only = true
use chroot = true
transfer logging = true
log format = %h %o %f %l %b
log file = /home/log/rsyncd.log
hosts allow = XXX.XXX.XXX.XXX #允许访问的ip
address = A.B.C.D#本机ip
port = 28000


[mysql_bak]
path = /data/bak/
read only = false
uid = 0 
gid = 0 
```

#### 命令Demo：
```shell
rsync -avh --port=28000 /source_path root@A.B.C.D::mysql_bak/dst_path
```

#### -c参数:
数据多可以不用-c参数。减少校验位的计算所消耗的性能

<http://superuser.com/questions/318964/rsync-takes-a-very-long-time-to-send-file-list>

#### --bwlimit参数
--bwlimit=50000限制速度为50000KB

#### Rsync服务器启动:
/usr/bin/rsync --daemon




