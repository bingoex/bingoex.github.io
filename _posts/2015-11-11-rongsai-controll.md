---
layout: post
title:  【精品】传统拥塞控制
categories: 计算机网络
description: 
keywords: 
---


![](/images/posts/2015-11-11-rongsai-controll.md/1.png)

**RTT**(Round-Trip Time): 往返时延。

在计算机网络中它是一个重要的性能指标，表示从发送端发送数据开始，到发送端收到来自接收端的确认（接收端收到数据后便立即发送确认），总共经历的时延。

RTT测量方法

1、TCP Timestam选项: RTT = 当前时间 -  数据包中Timestamp选项的回显时间。

2、重传队列中数据包的TCP控制块。在TCP重传队列中保存着发送而未被确认的数据包，数据包skb中的TCP控制块包含着一个变量，tcp_skb_cb->when，记录了该数据包的第一次发送时间。RTT = 当前时间– when。

**RTO**（RetransmissionTimeOut）即重传超时时间。



# 慢启动

## 什么时候进行慢启动？

1）新建连接；

2）RTO超时；

3）连接空闲超过一定时间；


慢开始算法的思路就是，不要一开始就发送大量的数据，先探测一下网络的拥塞程度，也就是说由小到大逐渐增加拥塞窗口的大小；

- 1、当cwnd<ssthresh时，使用慢开始算法；
- 2、cwnd初始化为1个MSS长度；
- 3、每收到一个ACK，cwnd增加1个MSS长度；
（第一次发送一个包，收到一个ack，cwnd变为2，意味着下一次可以同时发两个包，当两个包都收到了ack，则cwnd则加2,变为4了，其实一点都不慢！）

linux 3.0版本内核之后，初始化cwnd为10了。



# 拥塞避免

- 1、当cwnd>ssthresh时，改用拥塞避免算法
- 2、拥塞避免算法让拥塞窗口缓慢增长，即每经过一个往返时间RTT就把发送方的拥塞窗口cwnd加1；



# 发生RTO时

当发生RTO时，TCP认为情况太糟糕，反应也很强烈
处理方法：
- 1、sshthresh =  cwnd /2
- 2、cwnd 重置为 1（或者10）
- 3、进入慢启动过程



# 上面的改进

收到3个重复ACK，即启动**快速重传和快速恢复**；

处理方法：
- 1）引入ssthresh, ssthresh = cwnd / 2；
- 2）cwnd重新设为，cwnd = ssthresh + 3 * MSS；
- 3）重传丢失的数据段；
- 4）每收到一个重复ACK，cwnd增加一个分组大小，重传一个数据段；
- 5）收到新的ACK，设置cwnd为ssthresh，其实就是原来的cwnd减半；





