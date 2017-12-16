---
layout: post
title: 踩坑日记－maven包冲突
categories: Java
description: 
keywords: 
---


# 问题
java.lang.NoSuchMethodError:org.springframework.core.annotation.AnnotationAwareOrderComparator

当出现NoSuchMethodError时，极有可能是因为包冲突导致。两个包名中类名一样，如果包1的类的版本更新，拥有新方法。包2版本的类没有此方法。如果类加载器加载到了包2的类，程序中使用了较新的类的方法，则运行时会出现NoSuchMethodError异常。

# 解决方法

maven排包

command + o 查找类。如果找到多个想同的类名，则排除其中的包，留一个即可（可以勾选Show GroupId！！！）

![](/images/posts/2017-07-06-java-caikeng-maven.md/1.png)

![](/images/posts/2017-07-06-java-caikeng-maven.md/2.png)

![](/images/posts/2017-07-06-java-caikeng-maven.md/3.png)

