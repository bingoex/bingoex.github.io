---
layout: post
title: Java问题排查利器
categories: Java
description: 
keywords: 
---

当遇到java相关的问题时，可通过下面的工具快速定位问题。

# jps
输出JVM中运行的进程状态信息
```
-q 不输出类名、Jar名和传入main方法的参数
-m 输出传入main方法的参数
-l 输出main类或Jar的全限名
-v 输出传入JVM的参数
```
```shell
jps -m -l
1217 com.xxxx.sunfire.agent.Main
3093 org.apache.catalina.startup.Bootstrap start
84153 sun.tools.jps.Jps -m -l

jps -mlvV
```


# jinfo
```shell
//查看进程系统配置信息和参数
jinfo 3093
```



# jstat
```shell
jstat [option] pid
```
- -gc：
输出每个堆区域的当前可用空间以及已用空间（伊甸园，幸存者等等），GC执行的总次数，GC操作累计所花费的时间。

- -gcutil：
输出每个堆区域使用占比，以及GC执行的总次数和GC操作所花费的事件。

- -gccause：
输出-gcutil提供的信息以及最后一次执行GC的发生原因和当前所执行的GC的发生原因

- -gccapactiy：
输出每个堆区域的最小空间限制（ms）/最大空间限制（mx），当前大小，每个区域之上执行GC的次数。（不输出当前已用空间以及GC执行时间）。

- -gcnew：
输出新生代空间的GC性能数据

- -gcnewcapacity：
输出新生代空间的大小的统计数据。

- -gcold：
输出老年代空间的GC性能数据。

- -gcoldcapacity：
输出老年代空间的大小的统计数据。

- -gcpermcapacity：
输出持久带空间的大小的统计数据。

```shell
// 查看进程ID为3093的GC数据，每隔1000毫秒刷新一次
jstat  -gc 3093 1000 

S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
218432.0 218432.0 22575.7  0.0   2184576.0 1408239.1 1572864.0   276702.7  249216.0 240828.9 29312.0 27607.0     26    2.436   0      0.000    2.436
218432.0 218432.0 22575.7  0.0   2184576.0 1411475.7 1572864.0   276702.7  249216.0 240828.9 29312.0 27607.0     26    2.436   0      0.000    2.436
218432.0 218432.0 22575.7  0.0   2184576.0 1413157.0 1572864.0   276702.7  249216.0 240828.9 29312.0 27607.0     26    2.436   0      0.000    2.436
218432.0 218432.0 22575.7  0.0   2184576.0 1427651.8 1572864.0   276702.7  249216.0 240828.9 29312.0 27607.0     26    2.436   0      0.000    2.436
218432.0 218432.0 22575.7  0.0   2184576.0 1427706.7 1572864.0   276702.7  249216.0 240828.9 29312.0 27607.0     26    2.436   0      0.000    2.436

//额外输出上次GC原因
jstat -gccause
```
```
S0C：输出Survivor0空间的大小。单位KB。
S1C：输出Survivor1空间的大小。单位KB。
S0U：输出Survivor0已用空间的大小。单位KB。
S1U：输出Survivor1已用空间的大小。单位KB。
EC：输出Eden空间的大小。单位KB。
EU：输出Eden已用空间的大小。单位KB。
OC：输出老年代空间的大小。单位KB。
OU：输出老年代已用空间的大小。单位KB。
MC(PC)：输出方法区(持久代)空间的大小。单位KB。
MU(PU)：输出方法区(持久代)已用空间的大小。单位KB。
CCSC：输出压缩类空间大小。单位KB。
CCSU：输出压缩类空间使用大小。单位KB。
YGC：新生代空间GC时间发生的次数。
YGCT：新生代GC处理花费的时间。
FGC：full GC发生的次数。
FGCT：full GC操作花费的时间
GCT：GC操作花费的总时间。

NGCMN： 新生代最小空间容量，单位KB。
NGCMX： 新生代最大空间容量，单位KB。
NGC：新生代当前空间容量，单位KB。
OGCMN： 老年代最小空间容量，单位KB。
OGCMX： 老年代最大空间容量，单位KB。
OGC：老年代当前空间容量制，单位KB。
PGCMN： 持久代最小空间容量，单位KB。
PGCMX： 持久代最大空间容量，单位KB。
PGC：持久代当前空间容量，单位KB。
PC：持久代当前空间大小，单位KB。
PU：持久代当前已用空间大小，单位KB。
LGCC：最后一次GC发生的原因。
GCC：当前GC发生的原因。
TT：老年化阈值。被移动到老年代之前，在新生代空存活的次数。
MTT：最大老年化阈值。被移动到老年代之前，在新生代空存活的次数。
DSS：幸存者区所需空间大小，单位KB。
```



# top
```shell
//查看3093进程内所有线程信息
top -Hp 3093 

//获得线程10进制转16进制后jstack去抓看这个线程到底在干啥
printf "%x\n" 3093
```



# jstack
查看某个Java的线程堆栈信息

- -l long listings，会打印出额外的锁信息，在发生死锁时可以用jstack -l pid来观察锁持有情况
- -m mixed mode，不仅会输出Java堆栈信息，还会输出C/C++堆栈信息（比如Native方法）

```shell
jstack -F -m 3093
```


# jmap
```shell
//查看堆内存使用状况
jmap -heap 3093

Attaching to process ID 3093, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.112-b129

using parallel threads in the new generation.
using thread-local object allocation.
Concurrent Mark-Sweep GC

Heap Configuration:
MinHeapFreeRatio         = 40
    MaxHeapFreeRatio         = 70
    MaxHeapSize              = 4294967296 (4096.0MB)
    NewSize                  = 2684354560 (2560.0MB)
    MaxNewSize               = 2684354560 (2560.0MB)
OldSize                  = 1610612736 (1536.0MB)
    NewRatio                 = 2
    SurvivorRatio            = 10
    MetaspaceSize            = 536870912 (512.0MB)
    CompressedClassSpaceSize = 1073741824 (1024.0MB)
    MaxMetaspaceSize         = 536870912 (512.0MB)
G1HeapRegionSize         = 0 (0.0MB)

    Heap Usage:
    New Generation (Eden + 1 Survivor Space):
    capacity = 2460680192 (2346.6875MB)
    used     = 82042008 (78.2413558959961MB)
    free     = 2378638184 (2268.446144104004MB)
    3.3341190889709895% used
    Eden Space:
    capacity = 2237005824 (2133.375MB)
    used     = 57162688 (54.51458740234375MB)
    free     = 2179843136 (2078.8604125976562MB)
    2.555321375864241% used
    From Space:
    capacity = 223674368 (213.3125MB)
    used     = 24879320 (23.726768493652344MB)
    free     = 198795048 (189.58573150634766MB)
    11.123008962743555% used
    To Space:
    capacity = 223674368 (213.3125MB)
    used     = 0 (0.0MB)
    free     = 223674368 (213.3125MB)
    0.0% used
    concurrent mark-sweep generation:
    capacity = 1610612736 (1536.0MB)
    used     = 290819000 (277.3466110229492MB)
    free     = 1319793736 (1258.6533889770508MB)
    18.056419988473255% used

    86588 interned Strings occupying 8608688 bytes.
```

```shell
//查看堆内存中的对象数目
jmap -histo 3093
```

```shell
//把进程内存使用情况dump到文件中，后续再用jhat、MAT、VisualVM等工具分析查看
jmap  -dump:format=b,file=/tmp/dump.dat -F 3093
```


# jhat


# hprof
```shell
java -agentlib:hprof=cpu=samples,interval=20,depth=3 程序名
//每隔20毫秒采样CPU消耗信息，堆栈深度为3，在当前目录生成的profile文件（名称为java.hprof.txt）
```

#btrace

通过在运行中的java类中注入trace代码， 并对运行中的目标程序进行热交换(hotswap)来达到对代码的跟踪。使用btrace可以在不改进代码，不影响当前线上运行的基础上进行运行时环境的跟踪，可以免去打日志，发部等繁琐的工作，是一个非常不错的java诊断工具

```java
@BTrace
public class ThreadPoolTrace {

    private static volatile long count;

    @OnMethod(
            clazz="java.util.concurrent.ThreadPoolExecutor",
            method="<init>"
            )
        public static void onNew() {
            count++;//计数
            BTraceUtils.println(count);
            BTraceUtils.jstack();
        }

}
```

```shell
./btrace pid ThreadPoolTrace.java > btrace.log
```

# 原始GC日志查看
看应用启动参数找到日志文件（如 -Xloggc:/home/admin/logs/gc.log

- -XX:+PrintGC 输出GC日志
- -XX:+PrintGCDetails 输出GC的详细日志
- -XX:+PrintGCTimeStamps 输出GC的时间戳（以基准时间的形式）
- -XX:+PrintGCDateStamps 输出GC的时间戳（以日期的形式，如 2013-05-04T21:53:59.234+0800）
- -XX:+PrintHeapAtGC 在进行GC的前后打印出堆的信息
- -Xloggc:/home/admin/logs/gc.log Gc日志文件输出路径。

```shell
//例子
[GC（Allocation Failure） 2017-09-25T01:29:00.538+0800: 353407.711[ParNew: 1754248K- >7388K(1922432K), 0.0284345 secs]； 3008857K->1262047K(4019584K), 0.0290499 secs] [Times: user=0.05 sys=0.00, real=0.03 secs] 。

//详解
[GC（Allocation Failure） 2017-09-25T01:29:00.538+0800: （时间戳）353407.711（代表虚拟机启动以来的gc总秒数）[ParNew（GC的区域）: 1754248K（垃圾回收前的大小）- >7388K（垃圾回收以后的大小）(1922432K)（该区域总大小）, 0.0284345 secs（回收时间，秒）]； 3008857K（堆区垃圾回收前的大小）->1262047K（堆区垃圾回收后的大小）(4019584K)（堆区总大小）, 0.0290499 secs（回收时间）] [Times: user=0.05（GC用户耗时） sys=0.00（GC系统耗时）, real=0.03 secs（GC实际耗时）] 。
```

GC 和 Full GC 是垃圾回收的停顿类型，而不是区分是新生代还是年老代，如果有 Full 说明发生了 Stop-The-World。（如调用 System.gc()



# JMX

JMX（Java Management Extensions，即Java管理扩展）是一个为应用程序、设备、系统等植入管理功能的框架。JMX可以跨越一系列异构操作系统平台、系统体系结构和网络传输协议，灵活的开发无缝集成的系统、网络和服务管理应用。在java中就是java.lang.management这个包。


# 建立JMX连接

我们要用JMX监控性能指标，首先就需要和应用程序建立JMX连接，但并不是每一个应用程序都可以建立JMX连接的。应用在启动的时候，必须开启JMX端口，我们才能和应用建立连接开启JMX端口的方法很简单，就是在启动应用程序的时候，加上以下执行参数：

```java
-Dcom.sun.management.jmxremote.port=7979 -Dcom.sun.managent.jmxremote.authenticate=true
-Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.password.file=password.properties
```
- com.sun.management.jmxremote.port 开启的端口号
- com.sun.managent.jmxremote.authenticate 链接时是否需要身份认证
- com.sun.management.jmxremote.ssl 是否允许ssl，通常情况下位false
- com.sun.management.jmxremote.password.file身份认证文件路径

```java
Map map = new HashMap();
map.put("jmx.remote.credentials", new String[] { "username", "password" });

String jmxURL = "service:jmx:rmi:///jndi/rmi://10.122.4.88:7979/jmxrmi";
JMXServiceURL serviceURL = new JMXServiceURL(jmxURL);
JMXConnector connector = JMXConnectorFactory.connect(serviceURL, map);
MBeanServerConnection mbsc = connector.getMBeanServerConnection();
```

# 获取MXBean
JMX里面有很多个官方的MXBean，用于监控jvm和运行环境的各种指标。

ClassLoadingMXBean  |    用于Java虚拟机的类加载系统的管理接口。
CompilationMXBean    |    用于Java虚拟机的编译系统的管理接口。
GarbageCollectorMXBean  |     用于Java虚拟机的垃圾回收的管理接口。
MemoryManagerMXBean    |   内存管理器的管理接口。
MemoryMXBean  |   Java虚拟机的内存系统的管理接口。
MemoryPoolMXBean  |     内存池的管理接口。
OperatingSystemMXBean  |     用于操作系统的管理接口，Java虚拟机在此操作系统上运行。
RuntimeMXBean   |  Java虚拟机的运行时系统的管理接口。
ThreadMXBean    |    Java虚拟机线程系统的管理接口。

```java
JvmMXBeans mxbeans = JvmMXBeansFactory.getJvmMXBeans(mbsc);
ThreadMXBean threadBean = mxbeans.getThreadMXBean();
```




## JVM内存使用率

```java
MBeanServerConnection mbsc = connector.getConnection();
MemoryMXBean memoryMXbean = (MemoryMXBean) ManagementFactory.newPlatformMXBeanProxy(mbsc, "java.lang:type=Memory",MemoryMXBean.class);

MemoryUsage heapMemoryUsage = memoryMXbean. getHeapMemoryUsage();
MemoryUsage nonHeapMemoryUsage = memoryMXbean. getNonHeapMemoryUsage();
Long heapMemoryUsed = heapMemoryUsage.getUsed();
Long nonHeapMemoryUsed = nonHeapMemoryUsage.getUsed();
Long totalMemoryUsed = heapMemoryUsed + nonHeapMemoryUsed;
Long heapMemoryMax = heapMemoryUsage.getMax();
Long nonHeapMemoryMax = nonHeapMemoryUsage.getMax();
Long totalMemoryMax = heapMemoryMax+ nonHeapMemoryMax;

//计算使用率
Double heapMemoryUsage = heapMemoryUsed / heapMemoryMax;
Double nonHeapMemoryUsage = nonHeapMemoryUsed / nonHeapMemoryMax;
Double totalMemoryUsage = totalMemoryUsed / totalMemoryMax;
```



## 内存池
```java
ObjectName mbeanName = new ObjectName(ManagementFactory.MEMORY_POOL_MXBEAN_DOMAIN_TYPE + ",*");
Set<ObjectName> memoryPoolMBeansNames = mbsc.queryNames(mbeanName, null);

//遍历所有的MemoryPoolMXBean
for (ObjectName on : memoryPoolMBeansNames){
    //获取MXBean
    MemoryPoolMXBean mpool = (MemoryPoolMXBean) ManagementFactory.newPlatformMXBeanProxy(mbsc,on.getCanonicalName(),MemoryPoolMXBean.class);

    //分区名
    String name = mpool.getName();

    //内存数据
    MemoryUsage memory = mpool.getUsage();

    //内存使用值，单位是bytes
    Long memoryUsed = memory.getUsed();

    //内存使用率
    Long memoryMax = memory.getUsed();
    Double memoryUsage = memoryUsed / memoryMax;
}
```


## GC
```java
MBeanServerConnector connector = MBeanServerConnectorFactory.connect(this.getCtx().getParams());
MBeanServerConnection mbsc = connector.getConnection();

//获取所有GarbageCollectorMXBean的名称
ObjectName mbeanName = new ObjectName(ManagementFactory. GARBAGE_COLLECTOR_MXBEAN_DOMAIN_TYPE+ ",*");
Set<ObjectName> garbageCollectorMXBeanNames = mbsc.queryNames(mbeanName, null);

//遍历所有的GarbageCollectorMXBean
for (ObjectName on : garbageCollectorMXBeanNames){
    GarbageCollectorMXBean gcBean = (GarbageCollectorMXBean) ManagementFactory.newPlatformMXBeanProxy(mbsc,on.getCanonicalName(),GarbageCollectorMXBean.class);

    //GC类型名
    String gcName = gcBean.getName();

    //GC次数
    Long gcCount = gcBean.getCollectionCount();

    //GC时间
    Long gcTime = gcBean. getCollectionTime();
}
```


# 其他
- eclipse的MAT插件
- JProfiler






