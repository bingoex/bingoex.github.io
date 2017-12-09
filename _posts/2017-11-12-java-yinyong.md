---
layout: post
title: 浅谈Java引用
categories: Java
description: 
keywords: 
---


# 强引用
```java
//java.lang.ref.Reference
String str = new String("strong");
```
只要内存对象被root强引用，内存空间不足时，jvm宁愿抛出OutOfMemoryError错误，让程序异常终止，也不会释放这个对象占用的内存。


# 软引用
软引用会尽可能地将对象留存在内存中，供程序使用，直到程序结束被释放或者内存不足时被回收。

由于软引用随时可能被释放，所以从软引用获取对象时，要判断对象是否为null，为null需要重新创建对象。
```java
Memory m = new Memory();
Object o = new SoftReference(m);
m = null;
// do something
if (m != null && m.get() != null) {
    // do something
}
```

分配内存时, 当Eden内存不够用的时候，某些情况下会尝试到Old里进行分配(比如说要分配的内存很大)，如果还是没有分配成功，于是会触发一次ygc的动作，而ygc完成之后我们会再次尝试分配，如果仍不足以分配此时的内存，那会接着做一次full gc(不过此时的soft reference不会被强制回收)，将老生代也回收一下，接着再做一次分配，仍然不够分配,那会做一次强制将soft reference也回收的full gc，如果还是不能分配，那这个时候就抛出OutOfMemoryError。


# 弱引用
弱引用的对象，不论内存是否够用，在垃圾收集线程扫描到的时候都会将其回收。


# 虚引用
虚引用故名思议，和没有引用效果一样。虚引用不决定对象的生命周期。

