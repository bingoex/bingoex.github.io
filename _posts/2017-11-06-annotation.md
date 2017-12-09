---
layout: post
title: 注解原理
categories: Java
description: 
keywords: 
---


# 作用

注解很多时候的确能帮助我们减少、美化代码，或为现有代码增加额外功能等

1、为编译器提供信息：注解可以被编译器（如Eclipse、IntelliJ）用来检测错误和已知警告，如Java内置的@Deprecated、@Override和@SuppressWarnnings就被经常用到。

2、编译时和部署时附加操作：软件工具可以在编译时或部署时根据注解信息生成可执行代码，XML文件等等。如生成说明文档，Jax-ws,Jax-rs使用注解生成服务描述的XML文件。

3、运行时操作：一些注解可以在程序运行时再去进行解释。
- Hibernate和Mybatis使用它做动态表结构和SQL语句生成等操作
- Junit从4.x版本开始使用注解简化测试代码和流程
- AspectJ使用注解提供非侵入式的面向切面编程
- Spring也使用注解做XML配置管理的一种配合或替代



# 原理

**JVM通过Java的反射机获取程序代码中的注解，然后“注解处理器”根据预先设定的处理规则解析处理相关注解以达到主机本身设定的功能目标**。

**注解本质是一个继承了Annotation的特殊接口**。

jdk中是通过AnnotatedElement接口实现对注解的解析。Class类实现了AnnotatedElement接口，接口里面会通过反射解析本类中所有的注解信息并生成Annotation结构存储起来，后续供用户（如“注解处理器”）获取。
```java
//注解代码在包java.lang.reflect中
public final class Class<T> implements java.io.Serializable,
       GenericDeclaration,
       Type,
       AnnotatedElement {
       ......
}
```



# 元注解

Java提供了3种内置标准注解（@Override,@Deprecate,@SuppressWarnings）和四种元注解（@Target,@Retention,@Documented,@Inherited）

@Target,标示该注解可用在什么地方。可用参数包括（构造方法、字段、本地变量、方法、包、参数、类型（类、接口、注解、枚举））。详情请看ElementType枚举参数。

@Retention，标示注解的生命周期，详情请看RetentionPolicy枚举参数。
- RetentionPolicy.SOURCE：表明注解会被编译器丢弃，字节码中不会带有注解信息
- RetentionPolicy.CLASS：表明注解会被写入字节码文件，且是@Retention的默认值
- RetentionPolicy.RUNTIME：表明注解会被写入字节码文件，并且能够被JVM 在运行时获取到，可以通过反射的方式解析到

@Documented，是包含在JavaDoc中

@Inherited，允许子类继承父类的注解



# demo
```java
//声明注解（业务提供方）
@Retention(RetentionPolicy.RUNTIME)//声明注解的保留期限。运行期仍保留该注解，可通过反射获得
@Target(ElementType.METHOD) //声明可以使用该注解的目标类型
public @interface NeedTest { //定义注解
    boolean value() default true; //声明注解成员
}

//使用注解（业务使用方）
@NeedTest(value = true)
public void deleteForum(int forumId) {
    System.out.println("删除：" + forumId);
}

//访问注解（业务提供方 “注解处理器”）
public void ToolTest() {
Class clazz = ForumService.class;
Method[] methods = clazz.getDeclaredMethods();
System.out.println(methods.length);
for (Method method : methods) {
    NeedTest nt = method.getAnnotation(NeedTest.class);
    if (nt != null) {
        if (nt.value()) {
            System.out.println(method.getName() + "()需要测试");
        } else {
            System.out.println(method.getName() + "()不需要测试");
        }
    }
}
}
```




# 其他

- 注解工具APT
- javap -verbose HelloAnnotation.class 


