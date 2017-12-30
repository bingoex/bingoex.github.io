---
layout: post
title: Linux下开发－realloc
categories: Linux C/C++
description: 
keywords: 
---


```c
realloc(void *__ptr, size_t __size)
```
更改已经配置的内存空间，即更改由malloc()函数分配的内存空间的大小。

如果将分配的内存减少，realloc仅仅是改变索引的信息。

如果是将分配的内存扩大，则有以下情况：
1. 如果当前内存段后面有足够的内存空间，则直接扩展这段内存空间，realloc()将返回原指针。
2. 如果当前内存段后面的空闲字节不够，那么就使用堆中的第一个能够满足这一要求的内存块，将目前的数据复制到新的位置，并将原来的数据块释放掉，返回新的内存块位置。
3. 如果申请失败，将返回NULL，此时，原来的指针仍然有效。


