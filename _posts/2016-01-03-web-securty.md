---
layout: post
title: Web安全问题
categories: 安全
description: 
keywords: 
---


# XSS跨域攻击

XSS又称CSS，全称Cross Site Script，跨站脚本攻击，是Web程序中常见的漏洞，XSS属于被动式且用于客户端的攻击方式，所以容易被忽略其危害性。其原理是攻击者向有XSS漏洞的网站中输入(传入)恶意的HTML代码，当其它用户浏览该网站时，这段HTML代码会自动执行，从而达到攻击的目的。如，盗取用户Cookie、破坏页面结构、重定向到其它网站等。

XSS攻击分为DOM XSS攻击和Stored XSS攻击。

DOM XSS攻击是一种基于网页DOM结构的攻击，该攻击特点是中招的人是少数人。一个简单的例子就是web页面从url中获取参数值，然后直接显示在页面上。如果参数sContent包含恶意代码。最简单的例`sContent=<script>alert(‘123’)</script>`，那么页面就会出现显字符串123的弹窗。

StoredXSS攻击原理与Dom XSS攻击类似，只不过恶意代码被植入数据库。如果有一个web页面，需要每次读取数据库中含有恶意代码`<script>alert(‘123’)</script>`的sContent字段并在页面上展示，那么所有访问该页面的用户都会被出现字符串123的弹窗。
 
## 解决方法

前、后端对特殊字符转义，如"< >"等

HttpOnly属性：Cookie添加HttpOnly属性，使得攻击者无法通过JavaScript脚本窃取Cookie。



# 恶意文件上传

## 解决方法

校验上传文件的类型，限定上传文件格式。

限制存放文件目录的权限。



# 路径遍历

## 解决方法
将Js、HTML等资源文件独立部署。



# SQL注入

指攻击者在HTTP请求中注入恶意SQL命令，使得应用服务器调用请求构造SQL时和恶意SQL命令一起构造，并在数据库执行。
 
## 解决方法

通过正则表达式过滤"drop table"等可能注入的SQL。

使用预编译绑定参数是最好的手段，恶意SQL只会被当成SQL参数而不是命令。



# CSRF攻击（身份伪造）

攻击者通过跨站请求，以合法用户的身份进行非法操作，如转账交易等。利用用户浏览器Cookie或服务器Session策略，盗取用户身份（以你的身份干坏事）。
 
## 解决方法

表单Token：CSRF伪造用户请求，需构造用户请求的所有参数，表单Token通过在请求参数中增加一个随机数Token来阻止，每次响应页面Token均不同。

验证码：即需手动输入验证码，但这是一个不好的用户体验。

Referer源检查：Http请求头的Referer域中记录着请求来源，通过检查请求来源判断请求是否合法。

http://www.cnblogs.com/hyddd/archive/2009/04/09/1432744.html




# 错误回显

即Web服务器异常时，直接把异常堆栈信息输出给客户端浏览器。解决方法，将错误转到专门错误页面即可。
 
 

  


