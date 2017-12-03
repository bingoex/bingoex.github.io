---
layout: post
title: maven平板化依赖的弊端
categories: Java 工具
description: 
keywords: 
---

利用 maven 引入中间件可能[导致包冲突的问题](https://bingoex.github.io/2017/07/01/maven-instruct/#%E5%A6%82%E4%BD%95%E9%9A%94%E7%A6%BBjar%E5%8C%85)，这是 maven 平板化依赖的一个痛点

例子：
```xml
<dependency>
    <groupId>com.xxx.hsf</groupId>
    <artifactId>hsf.app.spring</artifactId>
    <version>2.1.1.4</version>
</dependency>
<dependency>
    <groupId>com.xxx.diamond</groupId>
    <artifactId>diamond-client</artifactId>
    <version>3.6.0</version>
</dependency>
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.16</version>
</dependency> 
```

利用 **mvn dependency:resolve** 或者 **mvn dependency:tree** 命令查看这几个中间件的依赖包，其中难免有些包会产生冲突。

在这里我们假设 hsf 内部依赖了 log4j:1.0.0，而应用依赖了 log4j:1.2.16，在编译的时候 hsf 有可能加载到应用引入的 log4j:1.2.16 而不是期望的 log4j:1.0.0，若是 log4j:1.0.0 和 log4j:1.2.16 相差较大，那么程序就可能抛出 **NoClassDefFoundError** 或者 **ClassNotFoundException**。

针对这种三方包冲突，通常我们都是在 maven 中把冲突包排掉，强行指定同一个版本，但是对于兼容性差的三方包无论我们怎么排冲突应用都有可能跑不起来。类似的冲突同样会在二方包之间发生，比如说 hsf 和应用中都依赖了 diamond，排掉 hsf 的 diamond 有可能导致 hsf 不可用，排掉应用中的 diamond 有可能导致我们的业务代码需要修改，显然哪一种做法都不是最佳选择。


# 结论：

**我们需要一个库（jar包）隔离容器**

