---
layout: page
title: 关于我
description: 最近会将很久之前总结的相关笔记整理成文章，以作总结。侵即删
keywords: HaiBinWu, bingo
comments: true
menu: 关于
permalink: /about/
---
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
