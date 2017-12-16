---
layout: post
title: Servlet简介
categories: Java 协议
description: 
keywords: 
---



# 简介

**Servlet与Servlet容器分离是为了解耦**，以更加适合工业化的生产方式。他们通过标准化的接口来互相协作，对外提供高效动态的服务。Servlet容器作为一个独立的标准化产品，目前有很多种类，如tomcat、jetty、jboss、webSphere、web logic等。


# 容器

![](/images/posts/2017-07-03-servlet-instruct.md/1.jpeg)

- Container 是容器的父接口，所有子容器都必须实现这个接口。
- **Wrapper 代表一个 Servlet。它负责管理一个 Servlet，包括 Servlet 的装载、初始化、执行以及资源回收**。
- 如果要运行 war 程序，就必须要 Host，因为 war 中必有 的**web.xml 文件，这个文件的解析需要 Host **。
- 一个 Host 在 Engine 中代表一个虚拟主机，这个虚拟主机的作用就是运行多个应用，它负责安装和展开这些应用，并且标识这个应用以便能够区分它们。
- 如果有多个 Host 就要定义一个 top 容器 Engine 了。而 Engine 没有父容器了，一个 Engine 代表一个完整的 Servlet 引擎。


# Servlet

servlet就是一个**实现了某些接口（javax.servlet）的Java类**。

Servlet的框架是由两个Java包组成:**javax.servlet和javax.servlet.http**。

在javax.servlet包中定义了所有的Servlet类都必须实现或扩展的的通用接口和类。

在javax.servlet.http包中定义了采用HTTP通信协议的HttpServlet类。

一个servlet所必要的几个生命周期方法（init、service、destory)，**每一个servlet都必须实现这些方法，并在相应的生命周期阶段自动被调用执行**。

**懒加载**就是在第一个请求到来时才加载，而**预先加载**则是在容器启动时就进行servlet的加载和初始化。

每一个请求都在独立的线程中被处理，同时一个servlet实例是可以被多个线程所共享的，当不再需要时，servlet会被gc掉。

servlet容器会为每一个应用创建一个ServletContext并保存在内存中，同时web.xml也会被解析，其中定义的servlet、listener、filter也会被创建于内存中；


# 生命周期
- 客户端请求servlet
- 加载servlet类到内存
- 实例化、初始化servlet
- init()初始化参数
- 调用service() (doGet() 或者 doPost())
- destroy()

可以重写service()方法。如果继承HttpServlet类，可以重写doGet()或者doPost()来处理客户端通过Get或者Post方法过来的表单。实际上，HttpServlet类的service()方法会调用doGet()方法或者doPost()方法


# Servlet与HTTP服务的关系

Servlet容器依附于web服务器，web服务器一般会在80端口监听http请求。当客户端发送一个http请求时，容器就会创建一个新的HttpServletRequest和HttpServletResponse对象，并且把它们传递给相应的filter和servlet的处理方法中，这些都是在同一个线程中执行的。

request对象包含所有的http请求信息，如http请求头和请求体等；response对象提供了很多灵活方便的方法去设置Http响应信息，如响应头和响应体等，当http响应完成之后，request对象和response对象将会被销毁掉。

**当第一次调用request对象的getSession方法时，servlet容器就会创建它并生成一个唯一session id保存在内存中，同时容器还会设置http响应的cookie信息，键为JSESSIONID，值就为刚刚创建的session id值。只要在cookie的有效期内，客户端在接下来的每次请求都带上这个cookie信息，而Servlet容器则会检查每个请求的头信息，获取名为JSESSIONID的cookie，并通过它的值来获得请求的session对象。**

HttpSession的存活期可以在web.xml中配置，默认是30分钟，所以当客户端在30分钟之内没有再次发送请求时，servlet容器就会自动销毁session对象，对于接下来的请求，就算指定了cookie值也不会访问到同一个session对象，servlet容器会创建一个新的[sessio](https://bingoex.github.io/2017/04/03/session-save-way/)。

客户端的session cookie默认生命周期和浏览器的生命周期一致，关掉浏览器窗口后session cookie就无效了。

