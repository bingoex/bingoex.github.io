---
layout: post
title: 深入理解Java虚拟机读书笔记－java Class类文件结构
categories: Java 读书笔记
description: 深入理解Java虚拟机读书笔记－java Class类文件结构
keywords: 
---

java Class类文件结构

![](/images/posts/2015-09-15-jvm-book-4-class-file.md/1.png)

# Class文件简介

任何一个Class文件都对应着唯一一个类或接口的定义信息，但反过来说，类或接口并不一定都得定义在文件里（譬如类或接口也可以通过类加载器直接生成）。

Class文件是一组以8位字节为基础单位的二进制流，各个数据项目严格按照顺序紧凑地排列在Class文件之中，中间没有添加任何分隔符。

class文件含两种数据类型：无符号数和表

**无符号数**属于基本的数据类型，以u1、 u2、 u4、 u8来分别代表1个字节、 2个字节、 4个字节和8个字节的无符号数，无符号数可以用来描述数字、 索引引用、 数量值或者按照UTF-8编码构成字符串值。

**表**是由多个无符号数或者其他表作为数据项构成的复合数据类型（如一个前置的容量计数器加若干个连续的数据项的形式）。


# Class文件具体格式

![](/images/posts/2015-09-15-jvm-book-4-class-file.md/2.png)

## 魔数（Magic Number）

每个Class文件的头4个字节称为魔数（Magic Number），它的唯一作用是确定这个文件是否为一个能被虚拟机接受的Class文件（0xCAFEBABE）。

## 版本号

第5和第6个字节是次版本号（Minor Version），第7和第8个字节是主版本号（Major Version）

## 常量池

常量池中主要存放两大类常量：字面量（Literal）和符号引用（Symbolic References）

- 字面量如文本字符串、 声明为final的常量值等。
- 符号引用如类和接口的全限定名（Fully Qualified Name）、字段的名称和描述符（Descriptor）、方法的名称和描述符

Java代码在进行Javac编译的时候，并不像C和C++那样有“连接”。因此字段、 方法的符号引用不经过运行期转换的话无法得到真正的内存入口地址，也就无法直接被虚拟机使用。当虚拟机运行时，需要从常量池获得对应的符号引用，再在类创建时或运行时解析、 翻译到具体的内存地址之中。

常量池中每一项常量都是一个表。表开始的第一位是一个u1类型的标志位代表当前这个常量属于哪种常量类型（如下图）。

![](/images/posts/2015-09-15-jvm-book-4-class-file.md/3.png)

![](/images/posts/2015-09-15-jvm-book-4-class-file.md/4.png)

![](/images/posts/2015-09-15-jvm-book-4-class-file.md/5.png)






## 访问标识

两个字节代表访问标志（如这个Class是类还是接口；是否定义为public类型；是否定义为abstract类型；如果是类的话，是否被声明为final等）


![](/images/posts/2015-09-15-jvm-book-4-class-file.md/6.png)

## 类索引、 父类索引和接口索引集合

类索引用（**指向前面介绍的常量池**）于确定这个类的全限定名，父类索引用于确定这个类的父类的全限定名。除了java.lang.Object外，所有Java类的父类索引都不为0。

## 字段表集合

字段表（field_info）用于描述接口或者类中声明的**变量**（类级变量以及实例级变量，但不包括在方法内部声明的局部变量）。字段表集合中不会列出从超类或者父接口中继承而来的字段，但有可能列出原本Java代码之中不存在的字段，譬如在内部类中为了保持对外部类的访问性，会自动添加指向外部类。

### 实例的字段

包含字段的作用域（public、 private、 protected修饰符）、 是实例变量还是类变量（static修饰符）、 可变性（final）、 并发可见性（volatile修饰符，是否强制从主内存读写）、 可否被序列化（transient修饰符）、 字段数据类型（基本类型、 对象、 数组）、 字段名称（指向前面介绍的常量池）、描述符。

其中描述符的作用是用来描述字段的数据类型

![](/images/posts/2015-09-15-jvm-book-4-class-file.md/7.png)

![](/images/posts/2015-09-15-jvm-book-4-class-file.md/8.png)

![](/images/posts/2015-09-15-jvm-book-4-class-file.md/9.png)




对于数组类型，每一维度将使用一个前置的“[”字符来描述，如一个定义为“java.lang.String[][]”类型的二维数组，将被记录为：“[[Ljava/lang/String；”，一个整型数组“int[]”将被记录为“[I”

## 方法表集合

方法表的结构如同字段表一样，依次包括了访问标志（access_flags）、 名称索引（name_index）、 描述符索引（descriptor_index）、 属性表集合（attributes）几项。如果父类方法在子类中没有被重写（Override），方法表集合中就不会出现来自父类的方法信息。

其中描述符的作用是用来描述方法的参数列表（包括数量、 类型以及顺序）和返回值。

![](/images/posts/2015-09-15-jvm-book-4-class-file.md/10.png)

方法里的Java代码，经过编译器编译成字节码指令后，存放在方法属性表集合中一个名为“Code”的属性里面。

用描述符来描述方法时，按照先参数列表，后返回值的顺序描述，参数列表按照参数的严格顺序放在一组小括号“（）”之内。 如方法void inc（）的描述符为“（）V”，方法java.lang.String toString（）的描述符为“（）Ljava/lang/String；”，方法intindexOf（char[]source,int sourceOffset,int sourceCount,char[]target,int targetOffset,int targetCount,int fromIndex）的描述符为“（[CII[CIII）I”

## 属性表集合

对于每个属性，它的名称需要从常量池中引用一个CONSTANT_Utf8_info类型的常量来表示，而属性值的结构则是完全自定义的。

![](/images/posts/2015-09-15-jvm-book-4-class-file.md/11.png)

### 1.Code属性

方法体中的代码经过Javac编译器处理后，最终变为字节码指令存储在Code属性内。Code属性出现在方法表的属性集合之中，但并非所有的方法表都必须存在这个属性，譬如接口或者抽象类中的方法就不存在Code属性

![](/images/posts/2015-09-15-jvm-book-4-class-file.md/12.png)


#### 异常表

异常表对于Code属性来说并不是必须存在的。

![](/images/posts/2015-09-15-jvm-book-4-class-file.md/13.png)

如果当字节码在第start_pc行到第end_pc行之间（不含第end_pc行）出现了类型为catch_type或者其子类的异常（catch_type为指向一个CONSTANT_Class_info型常量的索引），则转到第handler_pc行继续处理。 当catch_type的值为0时，代表任意异常情况都需要转向到handler_pc处进行处理。

### 2.Exceptions属性

方法描述时在throws关键字后面列举的异常。

### 3.LineNumberTable属性

用于描述Java源码行号与字节码行号（字节码的偏移量）之间的对应关系。如果选择不生成LineNumberTable属性，对程序运行产生的最主要的影响就是当抛出异常时，堆栈中将不会显示出错的行号，并且在调试程序的时候，也无法按照源码行来设置断点.

### 4.LocalVariableTable属性

用于描述栈帧中局部变量表中的变量与Java源码中定义的变量之间的关系

### 5.SourceFile属性

用于记录生成这个Class文件的源码文件名称

### 6.ConstantValue属性

通知虚拟机自动为静态变量赋值。
如果同时使用final和static来修饰一个变量，并且这个变量的数据类型是基本类型或者java.lang.String的话，就生成ConstantValue属性来进行初始化，如果这个变量没有被final修饰，或者并非基本类型及字符串，则将会选择在＜clinit＞方法中进行初始化

### 7.InnerClasses属性

用于记录内部类与宿主类之间的关联

### 8.Deprecated及Synthetic属性

Deprecated属性用于表示某个类、 字段或者方法，已经被程序作者定为不再推荐使用，它可以通过在代码中使用@deprecated注释进行设置。
Synthetic属性代表此字段或者方法并不是由Java源码直接产生的，而是由编译器自行添加的。

### 9.StackMapTable属性


### 10.Signature属性

在字节码（Code属性）中，泛型信息编译（类型变量、 参数化类型）之后都通通被擦除掉。使用擦除法的好处是实现简单（主要修改Javac编译器，虚拟机内部只做了很少的改动）、 非常容易实现Backport，运行期也能够节省一些类型所占的内存空间。 但坏处是运行期就无法像C#等有真泛型支持的语言那样，将泛型类型与用户定义的普通类型同等对待，例如运行期做反射时无法获得到泛型信息。Signature属性就是为了弥补这个缺陷而增设的，现在Java的反射API能够获取泛型类型，最终的数据来源也就是这个属性。

### 11.BootstrapMethods属性

这个属性用于保存invokedynamic指令引用的引导方法限定符。




