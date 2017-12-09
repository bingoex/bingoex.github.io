---
layout: post
title: 轻量级任务调度中间件
categories: Java 系统架构 工作
description: 
keywords: 
---


# 一、遇到的问题
近段时间这边遇到了一些类似的需求，而且未来可能还会有……源源不断的需求-_-

**案例1**: 当一个内容产生时，需要在**1小时后开始**，**每隔1小时**去检测该内容的质量评分（根据阅读量、点赞率等指标算出），**最多检测两天或者1000次**，如果满足阈值则进行大规模推广，从而提高优质内容的曝光率。

**案例2**: 当一个内容开始大规模推广时，可能因算法出错（阅读量高的内容不一定是好内容，如涉黄内容就是阅读量大而质量差的）等因素导致“差”的内容被大规模推广。因此我们需要在内容被大规模推广后，启动一个定时调度任务，**每隔4小时**后，检测内容的负反馈情况，达到阈值后及时删除内容，终止推广。

上面两个案例都需要一个定时调度中间件，在适当的时候“**提醒**”我们，执行相关的业务逻辑，从而达到我们的目标。这个时候老大给我提了下面需求
## 项目目标
1、功能支持：**多租户、多任务类型（阻塞定时调度、非阻塞定时调度、一次性）、起始时间、截止时间、最大执行次数、自定义执行间隔、任务发布频率、任务自定义清除**。

2、任务监控：每个租户、每种类型的任务执行频率、**堆积任务数、任务数据查看**、结果分发的监控。

3、性能指标：支持2W TPS的任务并发调度，**支持10亿的任务堆积**。

4、接入方式：odps离线导入、**HSF发布**。

# 二、方案选择
为了不重复造轮子，事先调研了市场上的定时调度中间件。

## 1、Timer、TimerTask
Timer是jdk中提供的一个定时器工具，使用的时候会在主线程之外起一个单独的线程执行指定的计划任务，含一个TimeTask队列。TimerTask是一个实现了Runnable接口的抽象类，代表一个可以被Timer执行的任务。

#### 优点：
- 提供简单调度功能。

#### 缺点：
- Timer实现为单线程，性能不足，需自己维护一个Timer池子。
- 需自己维护任务上下文
- 需要自己持久化任务。
- 需要自己实现分布式、异常容灾、负载均衡等问题。
- 需要自己实现任务监控。

## 2、ScheduledThreadPoolExecutor
Timer和TimeTask的多线程版本，性能更优。

#### 优点：
- 提供高性能定时调度功能。

#### 缺点：
- 需自己维护任务上下文。
- 需要自己持久化任务。
- 需要自己实现分布式、异常容灾、负载均衡等问题。
- 需要自己实现任务监控。

## 3、Quartz
底层使用ScheduledThreadPoolExecutor实现，提供了更友好的且带上下文的API。

#### 优点：
- 提供高性能定时调度功能。
- 提供任务上下文

#### 缺点：
- 需要自己持久化任务。
- 需要自己实现分布式、异常容灾、负载均衡等问题。
- 需要自己实现任务监控（框架封装过深，不便加监控）。

## 4、Spring定时器（Spring-task）
可以将它看成一个轻量级的Quartz，而且使用起来比Quartz简单许多。基于配置文件的，不支持API方式，不便于动态生产调度任务，直接放弃。

## 5、基于metaq实现
利用metaq自带的延迟发送功能可实现定时调度功能，消息体作为任务上下文，让机器做到任务无关性。metaq提供了分布式、容灾等功能、支持亿级别的堆积能力。基于上面的考虑，第一版的中间件以metaq为基础，提供了定时调度的功能。

#### 优点：
- 提供高性能定时调度功能。
- 提供任务上下文。
- 实现超简单。
- metaq服务自带分布式、容灾、负责均衡属性。

#### 缺点：
- 需要自己持久化任务。
- 需要自己实现任务监控（监控不方便）。
- 容易造成消息风暴。
- 当metaq消息丢失时，定时调度任务终止（一般metaq会保证消息不丢失）。

## 6、公司组件DTS
公司中间件团队开发的一款分布式任务调度产品，调度底层实现利用Quartz，且提供友好的API、分布式、容灾、负载均衡等重要的功能。

#### 优点：
- 提供高性能定时调度功能。
- 提供任务上下文。
- DTS服务自带分布式、容灾、负责均衡属性。

#### 缺点：
- 需要实现任务的持久化。
- 需要实现任务监控。
- 不便于动态生成需求定制的定时任务（什么时间点开始、结束、调度间隔、调度次数等）

## 结论：
为了避免重复造轮子和摒着站在巨人的角度，最终采用的方案是任务持久化落地到iDB并分库分表。DTS定时发布常驻任务，做到负载均衡，且每个分片任务只负责一个分表，避免竞争，提高性能。通过数据库操作提供任务实时监控功能。


# 四、总体设计

![](/images/posts/2017-11-04-task-sche-middle.md/1.png)

## 主要流程
1、用户通过HSF API发布定时调度任务，系统收到请求后做发布频率控制并把任务插入到数据库（512个分表）中。

2、DTS在系统启动时，发布常驻任务（512个任务分片），均匀分布在机器集群中（目前7台），并每隔一小段时间（目前为200ms）促发业务逻辑，根据业务分片号去自己负责的DB分表（加分布式锁）中查询可运行（到达定时间隔时间）的任务并更改状态为运行中。

3、精卫检测到数据库更改后，组装任务上下文信息，并发送metaq消息给用户。

4、客户端收到metaq后，回调用户实现的接口，实现业务逻辑。并把此次执行结果上报系统。


# 五、遇到的问题及解决方案

**问题一**、系统遇到了负载均衡问题，哪个系统进程负载哪个数据库的任务？扩容和当机后怎么解决负载问题？
- 通过DTS的常驻任务，可以解决分片均匀的问题。当系统扩容或缩容等容灾情况下能做到自动调节。

**问题二**、为了实现任务的持久化，我们需要把任务信息落地存储到数据库中，但频繁查询和修改数据库会出现性能问题，怎么解决？
- 我们把数据库分库分表（16个库、512个表）。并且每个分片任务只查询和加锁自己负责的分表中，避免了全局锁表。
- 给关键字段添加索引，提供查询效率。
- 定时删除过期长久的任务，减少数据库堆积量。

**问题三**、怎么做到修改任务状态后必然发出metaq消息通知客户端。
- 数据库修改直接用（update+where），而不是先select后update。
- 通过精卫可以确保数据库任务修改后，回调发送消息的业务逻辑，确保了事务性。

**问题四**、用户怎么接受调度任务通知
- 底层通过metaq，并封装友好的SDK提供给用户，用户只需实现简单的接口就可以。


# 六、业务接入方式
## 任务类型

阻塞型任务：到达时间间隔后，回调用户方法，只有用户回调结束后才开启新的一轮时间间隔。如果回调延迟，则下次回调顺延。

非阻塞型任务：不管用户是否回调结束，严格按照时间间隔执行任务。

一次性任务：只执行一次的任务。

详情请看com.xxx.perceptor.task.constant.TaskType

## 任务发送
```java
TaskOptionTO taskOptionTO = new TaskOptionTO();
taskOptionTO.setNamespace((long) NamespaceConstant.TYPE_WEITAO_FEED_PRIVATE_EXPAND);
taskOptionTO.setType(TaskType.SCHEDULE_BLOCKING);

taskOptionTO.setBeginTime(System.currentTimeMillis() / 1000); // 马上开始
taskOptionTO.setEndTime(getEndTime() / 1000); //2天
taskOptionTO.setIntervalTime((long) (2 * 60 * 60)); //间隔两小时
taskOptionTO.setMaxExecuteCnt(SysSwitch.scheduleMaxCheckCntForWeiTao);

Map<String, String> params = new HashMap<>();//上下文
params.put(MicrotaoTaskConstant.FEED_ID, String.valueOf(feedId));
params.put(MicrotaoTaskConstant.SCENE_ID, SysSwitch.scheduleSceneIdForWeiTao);
taskOptionTO.setParamsMap(params);

Long taskId = taskScheduleService.addTask(taskOptionTO);
PgLog.debug("taskScheduleService.addTask", taskId);
```

## 回调方法
```java
package com.xxx.perceptor.task.demo;

import com.xxx.fastjson.JSON;
import com.xxx.perceptor.common.PgLog;
import com.xxx.perceptor.task.domain.TaskContextTO;
import com.xxx.perceptor.task.processor.ScheduleTaskListener;
import com.xxx.perceptor.task.processor.TaskProcessResult;

/**
 * @author xxx
 * @since 2017/10/17
 */
public class DemoScheduleTaskListener implements ScheduleTaskListener {

    @Override
    public TaskProcessResult scheduleTaskProcess(TaskContextTO taskContextTO) {
        try {
            // 业务逻辑
            PgLog.task("DemoScheduleTaskListener", JSON.toJSONString(taskContextTO));
            // 用户可以设置next_execute_time, 默认按intervalTime间隔调度
            // 一天后执行
            // taskContextTO.getTaskUpdateTO().setNextExecuteTime(System.currentTimeMillis() / 1000 + 24 * 60 * 60));

            // 用户可以设置是否终止调度
            // taskContextTO.getTaskUpdateTO().setStop(true);

            // 用户修改上下文
            // taskContextTO.getTaskUpdateTO().getParams().put("newKey", "newVal");
        } catch (Exception e) {
            // PgLog.exp(e, "processScheduleTask error");
            return TaskProcessResult.FAIL;
        }

        return TaskProcessResult.SUCCESS;
    }
}

```

## xml配置
```xml
<bean id="demoScheduleTaskProcess" class="com.xxx.perceptor.task.processor.ScheduleTaskProcess" init-method="init" >
    <property name="listenerMap">
        <map>
            <entry key="0" value-ref="demoScheduleTaskListener" />
        </map>
    </property>
</bean>

<bean id="demoScheduleTaskListener" class="com.xxx.perceptor.task.demo.DemoScheduleTaskListener"/>

<bean id="taskScheduleService" class="com.xxx.hsf.app.spring.util.HSFSpringConsumerBean">
    <property name="interfaceName" value="com.taobao.perceptor.task.service.TaskScheduleService"/>
    <property name="version" value="${task.hsf.version}"/>
    <property name="group" value="HSF"/>
</bean>
```


# 四、未来展望
目前perceptor系统已经接入，并对外提供了带规则引擎的定时调度任务接口。未来还会有社区等其他业务会利用该中间件发布简单的定时调度任务。欢迎大家接入使用。

系统目前只提供了简单定时调度功能和查询功能，未来需要做到监控更精细化，提供Web等方式查看任务状态等信息。

业务odps任务导入仍未支持，这个能力也需要补上。


# 参考资料
<http://www.cnblogs.com/lingiu/p/3782813.html>

<http://blog.csdn.net/xieyuooo/article/details/8607220>

<http://www.quartz-scheduler.org/>






