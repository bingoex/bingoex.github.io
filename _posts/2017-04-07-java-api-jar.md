---
layout: post
title: Java常用API包及骚操作
categories: Java
description: 
keywords: 
---


- lombok（@data）
<https://my.oschina.net/darkness/blog/510808>


- org.apache.commons.lang3.StringUtils


- ReflectionToStringBuilder//不用写toString了


- org.apache.commons.beanutils//操作javabean


- stream API
```java
private List<CheckResult> checkResults = new ArrayList<>();
// ....
return checkResults.stream().filter(CheckResult::isSuccess).findFirst().orElse(null);
```


- FastJSON
```java
configTO.setHeaderPageInfo(JSON.parseObject(configDO.getHeaderPageInfoStr(), HeaderPageInfo.class));

JSONArray jsonArray = JSON.parseArray(configDO.getBizResDataStr());
JSONObject jsonObject = jsonArray.getJSONObject(index);
BusinessResourceDO bizResDO = JSON.parseObject(JSON.toJSONString(jsonObject), BusinessResourceDO.class);

Class queryOptionClass = Class.forName(STREAM_OPTION_PACKAGE + feedStreamQueryOptionClassName);
queryOption = (MtopFeedStreamQueryOption) JSON.parseObject(JSON.toJSONString(configQueryParamsMap),queryOptionClass);

feedBackdto = JSON.parseObject(URLDecoder.decode(Submit,"utf-8"), new TypeReference<FeedBackDTO>()
```



- map

```java
for (Map.Entry<String,Object> entry : streamQueryParamsJO.entrySet()){
    streamQueryParamsMap.put(entry.getKey(),String.valueOf(entry.getValue()));
}

configQueryParamsMap.putAll(urlQueryParamsMap);
```


org.apache.commons.lang3.EnumUtils

```java
String time = "H1";
TimeUnit timeUnit = EnumUtils.getEnum(TimeUnit.class, time);
```


org.apache.commons.collections.MapUtils


org.apache.commons.lang3.BooleanUtils


com.alibaba.fastjson







