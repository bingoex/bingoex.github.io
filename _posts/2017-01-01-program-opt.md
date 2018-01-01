---
layout: post
title: 【精品】程序优化的那些事
categories: 优化
description: 
keywords: 
---


# 系统设计层面

攒包/定时地进行批量处理。提高单包承载率，减少网络io、减少系统调用次数。
 
数据处理（cpu消耗型代码）、io独立线程处理。
 
多地部署，就近接入。分set，避免跨城、跨地区流量访问。
 
消息队列尽量无锁化、一写一读等。
 
尽量减少无用的代理层，去中间商。
 
灰度机制。
 
内核级优化（操作系统多队列网卡特性等）。
 
数据自动报表化，多维度展现。



# C语音层面

用register修饰高频访问的变量。register修饰的变量将尽可能放在CPU内部寄存器中，减少了访问内存的时间。
 
inline修饰高频调用的函数
 
操作char型数据转换成操作long型数据，因为32位系统简单操作一次32位数据的时间几乎和操作一次8位数据的时间相同
```c
for (j=0;j<8;j++)
   src_buf[j]^=iv_crypt[j];
```
需要存取内存8*2次，每次只操作1字节，它优化之后的语句：
`*(uint64_t *)src_buf ^= *(uint64_t *)iv_crypt;`
只需存取内存2*2次，每次可以操作4个字节。
    
优化之前的语句：
```c
while(nInBufLen) {
   if (src_i<8) {
       src_buf[src_i++]=*(pInBuf++);
       nInBufLen--;
   }
   ...
}
```

每个字节需要循环一次，优化之后，每8个字节循环一次。优化之后的语句如下：
```c
while(nInBufLen) {
    if (src_i == 0 && nInBufLen >= 8) {
        *(uint64_t *)src_buf = *(uint64_t *)pInBuf;
        pInBuf += 8;
        nInBufLen -= 8;
        src_i = 8;
        ...
    }
    ...
}
```

内存对齐访问。



