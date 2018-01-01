---
layout: post
title: 【精品】TCP三次握手
categories: 计算机网络
description: 
keywords: 
---

# 三次握手过程

![](/images/posts/2015-11-09-net-3-hank-tcp.md/1.png)


LISTEN: 表示服务器端的某个SOCKET处于监听状态，可以接受连接了。

SYN_SENT : 当客户端SOCKET执行CONNECT连接时，它首先发送SYN报文（同步序列编号SynchronizeSequence Numbers），随即进入SYN_SENT状态，并等待服务端的发送三次握手中的第2个报文。SYN_SENT状态表示客户端已发送SYN报文。（发送端）

SYN_RCVD : 这个状态与SYN_SENT遥想呼应，表示接受到了SYN报文，在正常情况下，这个状态是服务器端的SOCKET在建立TCP连接时的三次握手会话过程中的一个中间状态，很短暂，基本上用netstat你是很难看到这种状态的，除非你特意写了一个客户端测试程序，故意将三次TCP握手过程中最后一个 ACK报文不予发送。因此这种状态时，当收到客户端的ACK报文后，它会进入到ESTABLISHED状态。（服务器端）

ESTABLISHED：表示连接已经建立了。



# 为什么需要三次握手

client发出的第一个连接请求报文段并没有丢失，而是在某个网络结点长时间滞留了，以致延误到连接释放以后的某个时间才到达server。本来这是一个早已失效的报文段。但**server收到此失效的连接请求报文段后，就误认为是client再次发出的一个新的连接请求。于是就向client发出确认报文段，同意建立连接。假设不采用“三次握手”，那么只要server发出确认，新的连接就建立了**。由于现在client并没有发出建立连接的请求，因此不会理睬server的确认，也不会向server发送数据。但server却以为新的运输连接已经建立，并**一直等待client发来数据**。这样，**server的很多资源就白白浪费掉了**。采用“三次握手”的办法可以防止上述现象发生。例如刚才那种情况，client不会向server的确认发出确认。server由于收不到确认，就知道client并没有要求建立连接。主要目的**防止server端一直等待，浪费资源**。
 


 

