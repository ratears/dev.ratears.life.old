---
title: 《Nginx 核心知识 150 讲》study notes
author: ratears
categories:
	- [Nginx]
tags:
	- Nginx
	- study-notes
date: 2022-10-06 23:07:51
updated: 2022-10-06 23:07:51
---

# 初识Nginx

## Nginx的三个主要应用场景

- 静态资源服务
  - 通过本地文件系统提供服务

- 反向代理服务
  - Nginx的强大性能
  - 缓存
  - 负载均衡

- API服务
  - OpenResty

<br>

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.4ie7urbi69s0.webp" width="60%">

<br>

<br>

## Nginx出现的历史背景

- 互联网的数据量快速增长
  - 互联网的快速普及
  - 全球化
  - 物联网
- 摩尔定律：性能提升
- 低效的Apache
  - 一个连接对应一个进程

<br>

<br>

## Nginx的5个主要优点

（1）高并发，高性能

（2）可扩展性好

（3）高可靠性

（4）热部署

（5）BSD许可证

<br>

<br>

## Nginx的4个主要组成部分

（1）Nginx 二进制可执行文件

> 由各模块源码编译出的一个文件

（2）Nginx.conf 配置文件

> 控制 Nginx 的行为

（3）access.log 访问日志

> 记录每一条 http 请求信息

（4）error.log 错误日志

> 定位问题

<br>

<br>

## Nginx的版本发布历史

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.cmiy02n409c.webp" width="60%">

<br>

- 开源免费的Nginx与商业版Nginx Plus
- 阿里巴巴的Tengine
- 免费OpenResty与商业版OpenResty

<br>

<br>

## 编译Nginx

- 如果要扩展第三方功能，可以使用编译方式安装nginx

```shell
# （1）下载nginx
wget https://nginx.org/download/nginx-1.22.0.tar.gz

tar -zxvf nginx-1.22.0.tar.gz

# 安装相关依赖库
yum install -y pcre pcre-devel gcc zlib zlib-devel

# （2）编译安装
[root@bogon nginx-1.22.0]# ./configure --prefix=/usr/local/nginx

[root@bogon nginx-1.22.0]# make

[root@bogon nginx-1.22.0]# make install
```















<br>

<br>

<br>

# 学习备注

> 1. 课程介绍中的问题
> 2. 

```html
&emsp;&emsp;
```

<br>

<br>

<br>

<img src="" width="60%">

<br>