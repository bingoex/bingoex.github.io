---
layout: post
title: nginx源码浅析－编译前准备configure简介
categories: Nginx
description: nginx源码浅析－编译前准备configure简介
keywords: 
---

# configure文件的作用及其产出

- 生成obj、src等文件夹，用于存放编译中间文件
- 生成Makefile文件
- 记录特性检测日志autoconf.err
- 生成宏定义头文件ngx_auto_config.h和ngx_auto_headers.h
- 生成ngx_modules.c文件（含各模块extern声明、名字字符串、ngx_module_t数组ngx_modules）

# configure流程


- configure入参处理（option、case、sed）
```shell
. auto/options
```

- 定义宏（ngx_modules.c等），生成简单Makefile
```shell
 . auto/init
```

- 定义宏（源文件、模块）
```shell
 . auto/sources
```

- 检查操作系统（uname）

- 设置编译器相关参数，**编译期特性检测（检测方法：生成相应的代码并执行）**
```shell
 . auto/cc/conf
```

- 设置操作系统宏、unix相关宏及特性检测、线程相关宏
```shell
 . auto/os/conf
 . auto/unix
 . auto/threads
```

- 设置模块相关宏及文件ngx_modules.c（其中含有ngx_module_t数组）
```shell
 . auto/modules
```

- 依赖库宏设置及特性检测（如openssl）
```shell
 . auto/lib/conf
```

- 生成Makefile文件
```shell
 . auto/make
 . auto/lib/make
 . auto/install
```

