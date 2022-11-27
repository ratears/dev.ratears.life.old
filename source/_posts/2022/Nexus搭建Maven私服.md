---
title: Nexus搭建Maven私服
author: ratears
categories:
	- [Maven,Nexus]
tags:
	- Nexus
	- Maven
date: 2022-09-28 18:38:46
updated: 2022-09-28 18:38:46
---



# Nexus 安装

- [Nexus下载](https://www.sonatype.com/thanks/repo-oss)

```shell
# （1）上传 下载的软件到目录 /app
[root@bu2-vm-svr-67 app]# ll
total 212392
-rw-r--r-- 1 root root 217484934 Sep 28 14:06 nexus-3.42.0-01-unix.tar.gz

[root@bu2-vm-svr-67 app]# tar -zxvf nexus-3.42.0-01-unix.tar.gz

[root@bu2-vm-svr-67 app]# ll
total 212400
drwxr-xr-x 10 root root      4096 Sep 28 18:55 nexus-3.42.0-01
-rw-r--r--  1 root root 217484934 Sep 28 14:06 nexus-3.42.0-01-unix.tar.gz
drwxr-xr-x  3 root root      4096 Sep 28 18:55 sonatype-work

[root@bu2-vm-svr-67 app]# vi /app/nexus-3.42.0-01/bin/nexus

run_as_root=false

[root@bu2-vm-svr-67 app]# vi /app/nexus-3.42.0-01/etc/nexus-default.properties

application-port=7071

[root@bu2-vm-svr-67 app]# /app/nexus-3.42.0-01/bin/nexus start
Starting nexus
```





























































<br>

<br>

<br>

# 学习备注

> 1

```html
&emsp;&emsp;
```

<br>

<br>

<br>

<img src="" width="60%">

<br>