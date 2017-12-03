---
layout: post
title: 踩坑日记－Integer没设默认值的教训
categories: Java
description: 
keywords: 
---


# 背景
最近做了一个需要，需要新增一个Mtop接口的参数（一个string字符串，通过fastjason反序列化为一个java类），其中新增的java类字段类型为Integer。因为自测不够严谨，导致了线上出现了exception。

# 问题代码

```java
public class FeedBackDTO extends BaseDO {
    private Long         ContentId;
    private Long         AccountId;
    private Integer      BizType;
    private Long         userId;   //反馈者id
    private List<String> tags;
    private Integer      feedBackType   //有问题的代码，老接口没有

        public Integer getFeedBackType() {
            return feedBackType;
        }


    //使用处
    switch (feedBackdto.getFeedBackType()) {  // core!!!!!!!   bug!!!!!
        case FeedBackConstants.FEED_BACK_DISLIKE:
            dislikeTO.setNamespace(FeedBackConstants.NAME_SPACE_DISLIKE);
            break;
        case FeedBackConstants.FEED_BACK_BAD:
            dislikeTO.setNamespace(FeedBackConstants.NAME_SPACE_BAD);
            break;
        default:
            dislikeTO.setNamespace(FeedBackConstants.NAME_SPACE_DISLIKE);
    }
```

# 原因
因为老接口上报的string字符串不包新的字段feedBackType。fastjason在反序列化时，将其对象默认值设为null，当使用switch (feedBackdto.getFeedBackType())时对null进行了拆箱操作，出现了nullporintexption。

# 修正
```java
public class FeedBackDTO extends BaseDO {
    private Long         ContentId;
    private Long         AccountId;
    private Integer      BizType;
    private Long         userId;   //反馈者id
    private List<String> tags;
    private Integer      feedBackType = new Integer(0)   //!!!!!!给定默认值
    public Integer getFeedBackType() {
        return feedBackType;
    }
```

