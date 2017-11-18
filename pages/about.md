---
layout: page
title: 关于我
description: 最近会将很久之前总结的相关笔记整理成文章，以作总结。侵即删
keywords: HaiBinWu, bingo
comments: true
menu: 关于
permalink: /about/
---
对服务器开发（linux下开发）、操作系统、计算机网络（TCP/IP、HTTP协议）、算法有一定了解，突出的学习能力和学习热情，善于利用文档、StackOverflow学习。<br/>
有代码洁癖，把代码的“可维护性”和“可读性”置于首位。<br/>
了解分布式架构，有高可用、高并发场景下架构设计和开发的经验。<br/>

## 联系方式

{% for website in site.data.social %}
* {{ website.sitename }}：[@{{ website.name }}]({{ website.url }})
{% endfor %}

## 技能

{% for category in site.data.skills %}
### {{ category.name }}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
