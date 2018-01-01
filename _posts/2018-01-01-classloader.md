---
layout: post
title: 【精品】类加载的过程
categories: Java
description: 
keywords: 
---



```java
class A {
    public void m() {
        A.class.getClassLoader().loadClass(“B”);
    }

    public void doSomething() {
        B b = new B();
        b.doSomethingElse();
    }
}


B b = new B();
//等同于 
B b = Class.forName(“B”, false, A.class.getClassLoader()).newInstance()

```

# 类加载器
CurrentClassLoader，称之为当前类加载器，简称CCL，在代码中对应的就是类型A的类加载器；

SpecificClassLoader，称之为指定类加载器，简称SCL，在代码中对应的是 A.class.getClassLoader()，如果使用任意的ClassLoader进行加载，这个ClassLoader都可以称之为SCL；

ThreadContextClassLoader，称之为线程上下文类加载器，简称TCCL，每个线程都会拥有一个ClassLoader引用，而且可以通过Thread.currentThread().setContextClassLoader(ClassLoader classLoader)进行切换。

CCL的加载过程是由JVM运行时来控制的，是无法通过Java编程来更改的。

SystemDictionary，系统字典，这个数据结构是保存Java加载类型的数据结构。以类的全限定名再加上类加载器作为key，进而确定Class引用。


# Class.forName的过程

![](/images/posts/2018-01-01-classloader.md/1.png)

首先根据ClassLoader，我们称之为initialClassLoader和类名查找缓存，如果缓存有，则返回；

如果缓存没有，则调用ClassLoader.loadClass(类名)，加载到类型后，保存<类名+真实加载类的ClassLoader，类型引用>到缓存，这里真实加载类的ClassLoader我们可以叫做defineClassLoader；

返回的类型在交给Java之前，将会判断defineClassLoader是否等于initialClassLoader，如果不等，则新增<类名+initialClassLoader，类型引用>到缓存。

这里区分initialClassLoader和defineClassLoader的原因在于，调用initialClassLoader的loadClass，可能最终委派给其他的ClassLoader进行了加载。

# ClassLoader

当启动一个JVM时，bootstrap 类加载器就会加载java的核心类，例如：rt.jar中的类。bootstrap 类加载器是其他类加载器的parent，它是唯一一个没有parent的类加载器。

接下来是extension 类加载器，它以bootstrap 类加载器作为parent，它用来从Java系统变量java.ext.dir中的jar包中加载类的。

第三个，也是最重要的一个就是开发者使用的system classpath 类加载器 。它是extension 类加载器 的child，它用来从Java系统变量java.class.path下面加载类，可以通过 -classpath 来指定这个位置。

[双亲委派加载](https://bingoex.github.io/2015/09/17/jvm-book-3-classloader/)


