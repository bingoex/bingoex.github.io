---
layout: post
title: Java泛型理解
categories: 
description: 
keywords: 
---


# 问题


```java
public class ArrayList {
    public Object get(int i) { ... }
    public void add(Object o) { ... }
    ...
    private Object[] elementData;
}
```

当没有泛型时，我们会使用obejct替代。但这样会存在以下问题
- 第一有关get方法：每次调用get方法都会返回一个Object对象，每次都要强制类型转换为我们需要的类型。
- 第二有关add方法：假如我们往聚合了String对象的ArrayList中加入一个File对象，编译器不会产生任何错误提示。

为了解决上面的问题，范型应运而生。


# 泛型类


```java
Pair<String, Integer> pair = new Pair<String, Integer>();
```

你可能想象List<Integer>代表一个E被全部替换成Integer的版本。它可能导致误解，因为泛型声明绝不会实际的被这样替换。没有代码的多个拷贝。这是和C++模板的很大的区别。

如果Foo是Bar的一个子类型，而G是泛型声明，那么G<Foo>是G<Bar>的子类型并不成立


# 泛型方法

```java
static <T> void fromArrayToCollection(T[] a, Collection<T> c){
    for (T o : a) {
        c.add(o); // correct
    }
}
```

编译器根据实参为我们推断类型参数的值。它通常推断出能使调用类型最明确的类型参数

```java
class Collections {

    public static <T>  void copy(List<T> dest, List<? extends T> src){...}

}

class Collections {

    public static <T, S extends T>  void copy(List<T> dest, List<S> src){...}

}
```

```java
<T> T writeAll(Collection<T> coll, Sink<? super T> snk) { … }
String str = writeAll(cs, s); // YES!!!
```
推断出来的T是String。


# 通配符


Collection<?>。它的元素类型可以匹配任何类型。

```java
Collection<?> c = new ArrayList<String>();
c.add(new Object()); // 编译时错误
```
因为我们不知道c的元素类型，我们不能向其中添加对象。另一方面，我们可以调用get()方法并使用其返回值。

返回值是一个未知的类型，但是我们知道，它总是一个Object，因此把get的返回值赋值给一个Object类型的对象。

## 通配符的上限(upper bound)


`public void drawAll(List<? extends Shape> shapes) { //..}`
这里？代表一个未知的类型，就像我们前面看到的通配符一样。但是，在这里，我们知道这个未知的类型实际上是Shape的一个子类

```java
public void addRectangle(List<? extends Shape> shapes) {
    shapes.add(0, new Rectangle()); // compile-time error!
}
```

你应该能够指出为什么上面的代码是不允许的。因为shapes.add的第二个参数类型是? extends Shape ——一个Shape未知的子类。因此我们不知道这个类型是什么，所以这里传递一个Rectangle不安全。

- 如果你有一个只使用类型参数T作为参数的API，它的使用应该利用下限通配符( ? super T )的好处。
- 如果API只返回T，你应该使用上限通配符( ? extends T )来给你的客户端更大的灵活性。

可以使用&将多个通配符链接



# 新老代码兼容（java5为分界线）


当一个泛型类型，比如Collection被使用而没有类型参数时，它被称作一个raw type(自然类型??)。类型Collection表示一个未知类型元素的集合，就像Collection<?>。

自然类型和通配符类型很像，但是他们的类型检查不是同样严格。允许泛型与已经存在的老代码相交互是一个深思熟虑的决定。

一旦你把泛型编程和非泛型编程混合起来，泛型系统所提供的所有安全保证都失效。然而，你还是比你根本不用泛型要好。至少你知道你这一端的代码是稳定的。


# 本质（编译和擦除）

```java
public String loophole(Integer x) {
    List<String> ys = new LinkedList<String>();
    List xs = ys;
    xs.add(x); // compile-time unchecked warning
    return ys.iterator().next();

}

//上面的代码与下面的代码的行为一样：

public String loophole(Integer x) {
    List ys = new LinkedList();
    List xs = ys;
    xs.add(x);
    return (String) ys.iterator().next(); // run time error
}
```

泛型是通过java编译器的称为擦除(erasure)的前端处理来实现的。你可以（基本上就是）把它认为是一个从源码到源码的转换，它把泛型版本的loophole()转换成非泛型版本。


```java
List<String> l1 = new ArrayList<String>();
List<Integer> l2 = new ArrayList<Integer>();
System.out.println(l1.getClass() == l2.getClass());//true
```
所有的泛型类型在运行时有同样的类(class)，而不管他们的实际类型参数


```java
Collection cs = new ArrayList<String>();
if (cs instanceof Collection<String>) { ...} // 非法
```
类似的，如下的类型转换
`Collection<String> cstr = (Collection<String>) cs;`
得到一个unchecked warning，因为运行时环境不会为你作这样的检查。
对类型变量也是一样:
```java
<T> T badCast(T t, Object o) {
    return (T) o; // unchecked warning
}
```

类型参数在运行时并不存在。这意味着它们不会添加任何的时间或者空间上的负担，这很好。不幸的是，这也意味着你不能依靠他们进行类型转换。


String.class类型代表 Class<String>

`public static <T extends Comparable> T min(T[] a)`
编译后经过类型擦除会变成下面这样：
`public static Comparable min(Comparable[] a)`


# 注意事项


- 不能用基本类型实例化类型参数（new Pair<int, int>()）
- 不能抛出也不能捕获泛型类实例（T t;throw t;）
- 参数化类型的数组不合法(new Pair<String, String>[10])
- 不能实例化类型变量(new T(...))
- E（element）、K（key）、V（value）



# 参考

<http://blog.csdn.net/explorers/article/details/454837>
<http://www.cnblogs.com/absfree/p/5270883.html>




