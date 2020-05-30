---
layout: post
title:  nginx1.16编译安装
date:   2018-12-03
categories: nginx
permalink : nginx/configureInstall.html
description: nginx1.16编译安装
show : true
---

##### 下载

```shell
wget http://nginx.org/download/nginx-1.16.0.tar.gz
```

##### 编译安装

```shell
tar -xvzf nginx-1.16.0.tar.gz
cd nginx-1.16.0

./configure --prefix=/usr/local/nginx \ 
--user=nginx \
--group=nginx \
--with-http_slice_module \
--with-http_ssl_module \
--with-http_mp4_module \
--with-http_sub_module \ 
--with-http_gzip_static_module \
--with-http_stub_status_module \
--with-http_auth_request_module \
--with-http_gunzip_module \
--with-threads \
--with-http_v2_module \
--with-http_realip_module \
```

--prefix 指定目录 --user --group 用户与用户组 --with-***module 指定安装某个模块

如果出现以下错误，执行对应命令，安装依赖：

./configure: error: the HTTP rewrite module requires the PCRE library.

```shell
yum -y install pcre pcre-devel
```

./configure: error: SSL modules require the OpenSSL library.

```shell
yum -y install openssl openssl-devel
```

configure成功之后执行：

**make && make install **

