---
layout: post
title: Shell常用骚操作
categories: Shell
description: 
keywords: 
---



# 基本语法
```shell
while [ 1 ]; do sleep 1; ll; done # 无限循环
while [ $i -lt 10 ]; do echo $i;let "i=$i+1"; done # 有限循环
cat raw.txt | while read line; do echo $line; done # readline
until [ 1 = 0 ]; do sleep 1; ll; done # 无限循环
for i in /media/m* ; do ls -l $i; done # 与目录资源结合
if [ 1 -eq 1 ]; then ll ;fi # test常用判断
if [[ 0 -eq 0 && 1 -eq 0 ]]; then ll ;fi
if [ 0 -eq 0 -a 1 -eq 0 ]; then ll ;fi
if [ ! -e /tmp/111 -a -z "$a" ]; then ll ;fi # 不存在111文件 且a变量长度为0 则执行ll
```


# 常用文本操作命令

```shell
ps -ef | grep java | grep -v eclipse # 查看进程，筛选出java的，排除eclipse的
echo helloworld | tr -d "o" # 删除字符o，输出 hellwrld
echo 'a:b:c' | tr -s ':' '*' # 替换字符:为*，输出 a*b*c
echo 'a:b:c' | awk -F ':' '{print $1 "+" $3 "+" $2}' # 按:切分后，按下标调整顺序，空格分割输出。a+c+b
awk -F':' '{print $1}' temp2.log | awk '{ arr[$1]++ } END { for( no in arr) { print no , arr[no] } }' | sort -n -t" " -k 2 -r # 一句话实现group by
echo 'a:b:c' | sed -e 's#:#*#g' # 替换字符:为*，输出 a*b*c
```


# 常用命令

```shell
sudo su admin # 切换为admin身份
sudo -u admin kill -9 xxx # 以admin身份执行kill命令
ps -ef # 查看java进程
zip -9 -p haha -r bak.zip src # 以9级压缩比、haha为密码，压缩src目录，压缩后的文件是bak.zip
```


# 远程骚操作

```shell
sudo su admin # 切换为admin身份
sudo -u admin kill -9 xxx # 以admin身份执行kill命令
ps -ef # 查看java进程
zip -9 -p haha -r bak.zip src # 以9级压缩比、haha为密码，压缩src目录，压缩后的文件是bak.zip
```








