---
layout: post
title: Java必知必会及规范
categories: Java
description: 
keywords: 
---

# 基础

对于Integer i=?在**-128至127**之间的赋值，Integer对象是在IntegerCache.cache产生，会复用已有对象，这个区间内的Integer值可以直接使用==进行判断，但是这个区间之外的所有数据，都会在堆上产生，并不会复用已有对象，这是一个大坑。


对象的clone默认是浅拷贝，若想实现深拷贝需要重写clone方法实现属性对象的拷贝。


只要重写equals，就必须重写hashCode。


集合类 | Key | Value |  Super |  说明
Hashtable |  不允许为null |   不允许为null  |  Dictionary   | 线程安全
ConcurrentHashMap |  不允许为null  |  不允许为null |  AbstractMap | 分段锁技术
TreeMap | 不允许为null  |  允许为null |  AbstractMap | 线程不安全
HashMap | 允许为null | 允许为null | AbstractMap | 线程不安全



Serializable接口不能继承

不要在foreach循环里进行元素的remove/add操作。remove元素请使用Iterator方式，如果并发操作，需要对Iterator对象加锁。

泛型通配符<? extends T>来接收返回的数据，此写法的泛型集合不能使用add方法，而<? super T>不能使用get方法，两者在接口调用赋值的场景中易出错。
- 第一、频繁往外读取内容的，适合用<? extends T>。
- 第二、经常往里插入的，适合用<? super T>。

调用远程操作必须有超时设置。

JVM的Xms和Xmx设置一样大小的内存容量，避免在GC后调整堆大小带来的压力


# 命名规范
1. 数据对象：xxxDO，xxx即为数据表名。
2. 数据传输对象：xxxDTO，xxx为业务领域相关的名称。
3. 展示对象：xxxVO，xxx一般为网页名称。
4. POJO是DO/DTO/BO/VO的统称，禁止命名成xxxPOJO


- 所有的POJO类属性必须使用包装数据类型。
- RPC方法的返回值和参数必须使用包装数据类型。
- 所有的局部变量推荐使用基本数据类型。

定义DO/DTO/VO等POJO类时，**不要设定任何属性默认值**

POJO类必须写toString方法（可考虑ReflectionToStringBuilder）
- DO（Data Object）：与数据库表结构一一对应，通过DAO层向上传输数据源对象。
- DTO（Data Transfer Object）：数据传输对象，Service和Manager向外传输的对象。
- BO（Business Object）：业务对象。可以由Service层输出的封装业务逻辑的对象。
- Query：数据查询对象，各层接收上层的查询请求。注：超过2个参数的查询封装，禁止使用Map类来传输。
- VO（View Object）：显示层对象，通常是Web向模板渲染引擎层传输的对象。


# 类访问相关
1. 如果不允许外部直接通过new来创建对象，那么构造方法必须是private。
2. 工具类不允许有public或default构造方法。
3. 类非static成员变量并且与子类共享，必须是protected。 
4. 类非static成员变量并且仅在本类使用，必须是private。
5. 类static成员变量如果仅在本类使用，必须是private。
6. 若是static成员变量，必须考虑是否为final。
7. 类成员方法只供类内部调用，必须是private。 
8. 类成员方法只对继承类公开，那么限制为protected。

# equals和hashCode
1.  只要重写equals，就必须重写hashCode。
2.  因为Set存储的是不重复的对象，依据hashCode和equals进行判断，所以Set存储的对象必须重写这两个方法。
3.  如果自定义对象做为Map的键，那么必须重写hashCode和equals。


# 参数校验

下列情形中，需要进行参数校验
1. 调用频次低的方法。
2. 执行时间开销很大的方法。此情形中，参数校验时间几乎可以忽略不计，但如果因为参数错误导致中间执行回退，或者错误，那得不偿失。
3. 需要极高稳定性和可用性的方法。
4. 对外提供的开放接口。
5. 敏感权限入口。

下列情形中，不需要进行参数校验：
1. 极有可能被循环调用的方法。但在方法说明里必须注明外部参数检查。
2. 底层调用频度比较高的方法。毕竟是像纯净水过滤的最后一道，参数错误不太可能到底层才会暴露问题。一般DAO层与Service层都在同一个应用中，部署在同一台服务器中，所以DAO的参数校验，可以省略。
3. 被声明成private只会被自己代码所调用的方法。如果能够确定调用方法的代码传入参数已经做过检查或者肯定不会有问题，此时可以不校验参数。

# 异常

捕获异常是为了处理它，不要捕获了却什么都不处理而抛弃之，如果不想处理它，请将该异常抛给它的调用者。最外层的业务使用者，必须处理异常，将其转化为用户可以理解的内容。


不能在finally块中使用return，finally块中的return返回后方法结束执行，不会再执行try块中的return语句

# 数据库
单表行数超过500万行或者单表容量超过2GB，才推荐进行分库分表。

表必备三字段：id, gmt_create, gmt_modified。

小数类型为decimal，禁止使用float和double

任何操作，都是先保存数据库成功后，再进行缓存的新增、更新、清除操作。

区分度最高的在最左边。存在非等号和等号混合判断条件时，在建索引时，请把等号条件的列前置。如：where a>? and b=? 那么即使a的区分度更高，也必须把b放在索引的最前列。

唯一索引名为uk_字段名；普通索引名则为idx_字段名

varchar是可变长字符串，不预先分配存储空间，长度不要超过5000，如果存储长度大于此值，定义字段类型为TEXT，独立出来一张表，用主键来对应，避免影响其它字段索引效率

冗余字段应遵循
- 不是频繁修改的字段。
- 不是varchar超长字段，更不能是text字段

# 并发

如果每次访问冲突概率小于20%，推荐使用乐观锁，否则使用悲观锁。乐观锁的重试次数不得小于3次

在高并发场景中，避免使用“等于”判断作为中断或退出的条件。应该使用大于或小于

ThreadLocal对象建议使用static修饰。这个变量是针对一个线程内所有操作共享的，所以设置为静态变量，所有此类实例共享此静态变量 ，也就是说在类第一次被使用时装载，只分配一块存储空间，所有此类的对象(只要是这个线程内定义的)都可以操控这个变量

volatile解决多线程内存不可见问题。对于一写多读，是可以解决变量同步问题，但是如果多写，同样无法解决线程安全问题。


# 其他
更多内容可参考 <https://bingoex.github.io/2015/08/18/think-in-java-book/>


