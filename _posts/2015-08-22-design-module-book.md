---
layout: post
title: Head First设计模式读书笔记
categories: 设计模式 读书笔记
description: Head First设计模式读书笔记
keywords: 设计模式, 读书笔记
---

Head First设计模式读书笔记

# 设计原则
- 找出应用中可能需要变化之处，把他们独立出来，不要和那些不需要变化的代码混在一起。
- 多用组合，少用继承
- 为了交互对象之间的松耦合设计而努力
- 类应该对扩展开放，对修改关闭
- 要依赖抽象，不要依赖具体类
- 最少知识原则，只和你的密友谈话。
- 一个类应该只有一个引起变化的原因。
- 别调用（打电话给）我们，我们会调用（打电话给）你


# 策略模式：
定义了算法簇，分别封装起来，让他们之间可以互相替换，此模式让算法的变化独立于使用算法的客户。

![](/images/posts/2015-08-22-design-module-book/1.png)




# 观察者模式：

定义了对象之间的一对多依赖，这样一来，当一个对象改变状态时，它的所有依赖者都会收到通知并自动更新。

![](/images/posts/2015-08-22-design-module-book/2.png)

例子：

![](/images/posts/2015-08-22-design-module-book/3.png)






# 装饰者模式：

动态地将责任附加到对象上。若要扩展功能，装饰者提供了比继承更具有弹性的替代方案。

![](/images/posts/2015-08-22-design-module-book/4.png)





# 简单工程模式：

![](/images/posts/2015-08-22-design-module-book/5.png)






# 工厂方法模式：

定义了一个创建对象的接口，但由子类决定要实例化的类是哪一个。工厂方法让类把实例化推迟到子类。

![](/images/posts/2015-08-22-design-module-book/6.png)
![](/images/posts/2015-08-22-design-module-book/7.png)
![](/images/posts/2015-08-22-design-module-book/8.png)


# 抽象工厂模式：

提供一个接口，用于创建相关或依赖对象的家族，而不需要明确指定具体类。

![](/images/posts/2015-08-22-design-module-book/9.png)



# 单件模式：

确保一个类只有一个实例，并提供一个全局访问点




# 命令模式：

将“请求”封装成对象，以便使用不同的请求、队列或者日志来参数化其他对象。命令模式也支持可撤销的操作。

![](/images/posts/2015-08-22-design-module-book/10.png)




# 适配器模式：

将一个类的接口，转换成客户期望的另一个接口。适配器让原本接口不兼容的雷可以合作无间。

![](/images/posts/2015-08-22-design-module-book/11.png)








# 外观模式：

提供了一个统的接口，用来访问子系统中的一群接口。外观定义了一个高层的接口，让子系统更容易使用。

![](/images/posts/2015-08-22-design-module-book/12.png)






# 模板方法模式：

在一个方法中定义一个算法的骨架，而将一些步骤延迟到子类中。模板方法使得子类可以在不改变算法结构的情况下，重新定义算法中的某些步骤。

![](/images/posts/2015-08-22-design-module-book/13.png)







# 迭代器模式：

提供一种方法顺序访问一个聚合对象中的各个元素，而又不暴露其内部的表示。

![](/images/posts/2015-08-22-design-module-book/14.png)







# 组合模式：

允许你将对象组合成树形结构来表现“整体/部分”层次结构。组合能让客户以一致的方式处理个别对象以及对象组合。

![](/images/posts/2015-08-22-design-module-book/15.png)






# 状态模式：

允许对象在内部状态改变时改变它的行为，对象看起来好像修改了它的类。

![](/images/posts/2015-08-22-design-module-book/16.png)






# 代理模式：

为另一个对象提供一个替身或者占位符以控制这个对象的访问。

![](/images/posts/2015-08-22-design-module-book/17.png)
![](/images/posts/2015-08-22-design-module-book/18.png)







# mvc模式：
![](/images/posts/2015-08-22-design-module-book/19.png)




# 模式分类：
![](/images/posts/2015-08-22-design-module-book/20.png)
![](/images/posts/2015-08-22-design-module-book/21.png)




