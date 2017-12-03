---
layout: post
title: Nginx源码浅析－源码编译
categories: Nginx
description: Nginx源码浅析－源码编译
keywords: 
---


# nginx源码下载


<https://github.com/nginx/nginx.git>

# openssl源码下载


在nginx源码目录下建立bundle目录并下载openssl最新源码
<https://github.com/openssl/openssl>

```xml
nginx（git-root-path）
bundle
        openssl（git-root-path）
            my.configurae

```

# my.configure文件
```shell
#!/bin/bash
            ./auto/configure \
            --sbin-path=/Users/yourhomework/nginx/nginx \
            --conf-path=/usr/local/nginx/nginx.conf \
            --pid-path=/usr/local/nginx/nginx.pid \
            --with-http_ssl_module \
            --with-openssl=bundle/openssl \
# --with-pcre=/usr/local/Cellar/pcre/8.39 \
# --with-zlib=../zlib-1.2.11 \
```

```shell
./my.configure&&make
```

