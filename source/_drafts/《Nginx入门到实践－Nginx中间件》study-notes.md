---
title: 《Nginx入门到实践－Nginx中间件》study notes
author: sonzonzy
date: 2022-05-21 10:35:03
updated: 2022-05-21 10:35:03
categories:
  - nginx
tags:
  - nginx
---

# Nginx起步

## 简介

- Nginx是一个开源且高性能、可靠的http中间件、代理服务
  - 高效：支持海量的并发请求
  - 可靠：Nginx的服务是可靠运行的
  - 开源

<br/>

## 特点（优势）

- IO多路复用epoll
- 轻量级：功能模块少、代码模块化
- cpu亲和
- sendfile

<br/>

## 下载与安装

### 环境准备

- centos7
- 四项确认
  - 确认系统网络（可以连接到公网）：	`ping www.baidu.com`
  - 确认yum源可用： `yum list|grep gcc`
  - 确认关闭iptables规则 （规则会对验证HTTP服务造成影响）
  - 确认停用selinux

<br/>

```bash
# 查看是否有iptables规则
iptables -L
iptables -t nat -L
# 关闭规则
iptables -F
iptables -t nat -L



# 查看SELinux状态，如果SELinux status参数为enabled即为开启状态
getenforce

# 临时关闭（不用重启机器）
setenforce 0

#设置SELinux 成为permissive模式
#setenforce 1 设置SELinux 成为enforcing模式

# 修改配置文件需要重启机器
# 修改/etc/selinux/config 文件
# 将SELINUX=enforcing改为SELINUX=disabled
```

- 依赖工具包安装

```bash
yum -y install gcc gcc-c++ autoconf pcre pcre-devel make automake wget httpd-tools vim
```

- 一次初始化目录

```bash
# cd /opt目录下
cd /opt && mkdir app download logs work backup

# 软件、应用、代码 	 app		
# 备份文件 			backup		
# 下载内容			download	
# 自定义日志		   logs		
# shell脚本		 work		
```

<br/>

### 下载安装

- 使用yum源方式安装

```
# 基于yum源（这种方式不需要源码一个个编译，加入package参数。这种方式效率高）
cd /etc/yum.repos.d && touch nginx.repo && vim /etc/yum.repos.d/nginx.repo

# 添加如下内容 $releasever 换成centos版本
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true


yum list | grep nginx

yum install nginx

# 查看版本
nginx -v

# 查看编译参数
nginx -V

rpm -ql nginx
```

<br/>

## Nginx启停、重载与检查

```linux
systemctl start nginx.service
systemctl stop nginx.service
systemctl restart nginx.service

# 重载服务
nginx -s reload -c /etc/nginx/nginx.conf

# -t 检查配置文件的正确与否
# -c 路径检查
nginx -t -c /etc/nginx/nginx.conf
nginx -tc /etc/nginx/nginx.conf

```

<br/>

## Nginx目录详解

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/1656978bb93c905fb3c43afa17d66c7.1ucnkaxhiihs.webp" width="75%"/>



<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/4e657e9c7d08ca72f6a3fcf6b075202.6t6iyec78d00.webp" width="75%"/>

<br/>

## Nginx安装编译参数

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.64mpovli4l80.webp" width="75%" />



# Nginx基础

## Nginx默认配置语法

### nginx.conf（分三大块）

```nginx
user  nginx;	# 设置nginx服务的系统使用用户
worker_processes  1;	# 工作进程数

error_log  /var/log/nginx/error.log warn;	# nginx的错误日志
pid        /var/run/nginx.pid;	# nginx服务启动时候pid
# （1）以上是全局、服务模块配置

events {
    worker_connections  1024;	# 每个进程允许最大连接数
    use  1;	#	工作进程数
}
# （2）以上是事件模块

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;
		
		# 包含该文件
    include /etc/nginx/conf.d/*.conf;
}
# （3）以上是http模块
```

<br/>

### default.conf

```nginx
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log     main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```

<br/>

## Nginx 虚拟主机及实现方式

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.29gkicljqdxc.webp" width="65%" />

<br/>

### 虚拟机主机配置方式（三种）

#### 一：基于主机多IP方式（两种）

1. 多网卡多IP
2. 单网卡多IP

```bash
# 这里我们使用单网卡多ip方式实践，适用于 centos虚拟机

# 临时性的，重启网络后失效
ip a add 192.168.163.202/24 dev ens32

# 配置成功后如下
[root@localhost backup]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens32: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:f9:3c:04 brd ff:ff:ff:ff:ff:ff
    inet 192.168.163.201/24 brd 192.168.163.255 scope global noprefixroute ens32
       valid_lft forever preferred_lft forever
    inet 192.168.163.202/24 scope global secondary ens32
       valid_lft forever preferred_lft forever
    inet6 fe80::851d:65b2:ca19:571a/64 scope link noprefixroute
       valid_lft forever preferred_lft forever

```

- 在nginx 的 conf.d 目录下，新建两个配置文件,并做如下配置。另外还需要分别准备对应的访问目录及文件

```nginx
	listen       192.168.163.201:80;
    server_name  localhost;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /opt/app/code_ip_201/;
        index  index.html index.htm;
    }
```

```nginx
	listen       192.168.163.202:80;
    server_name  localhost;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /opt/app/code_ip_202/;
        index  index.html index.htm;
    }
```

- 配置好后，先检查nginx配置文件的正确与否，然后重载nginx即可访问查看效果

<br/>

#### 二：基于端口配置方式

- 在nginx 的 conf.d 目录下，新建两个配置文件,并做如下配置。另外还需要分别准备对应的访问目录及文件

```nginx
 	listen       81;
    server_name  localhost;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /opt/app/code_port_81/;
        index  index.html index.htm;
    }
```

```nginx
	listen       82;
    server_name  localhost;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /opt/app/code_port_82/;
        index  index.html index.htm;
    }
```

- 配置好后，先检查nginx配置文件的正确与否，然后重载nginx即可访问查看效果

<br/>

#### 三：基于多个host名称方式（多域名方式）

```nginx
# 这里，我们在访问机上配置dns，来实现多域名方式
# 修改访问机器的hosts文件

192.168.163.201 a.sonzonzy.com
192.168.163.201 b.sonzonzy.com
```

- 在nginx 的 conf.d 目录下，新建两个配置文件,并做如下配置。另外还需要分别准备对应的访问目录及文件

```nginx
	listen       80;
    server_name  a.sonzonzy.com;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /opt/app/code_host_a/;
        index  index.html index.htm;
    }
```

```nginx
	listen       80;
    server_name  b.sonzonzy.com;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /opt/app/code_host_b/;
        index  index.html index.htm;
    }
```

- 配置好后，先检查nginx配置文件的正确与否，然后重载nginx即可访问查看效果

<br/>

## 

## Nginx日志

- error.log
  - 处理HTTP请求的错误状态
  - 以及Nginx错误运行的状态

- access_log
  - 记录每一次请求的访问状态
  - 分析请求

<br/>

### log_formate

```shell
Syntax:	log_format name [escape=default|json|none] string ...;
Default:	
log_format combined "...";
Context:	http
```

## Nginx 变量

- HTTP 请求变量
- 内置变量 — Nginx 内置的
- 自定义变量 — 自己定义



## Nginx模块

- Nginx官方模块
- 第三方模块

### ngx_http_stub_status_module

- 查看nginx的连接状态等基本信息

```bash
Syntax:	stub_status;
Default:	—
Context:	server, location
```

```bash
# 配置示例
location = /basic_status {
    stub_status;
}
```

```bash
# The current number of active client connections including Waiting connections.
# nginx当前活跃的连接数
Active connections: 2 
# nginx处理的握手次数 连接数 总的请求数
server accepts handled requests
 3 3 3 
# 当前正在读的数量 正在写的数量 等待总数量（开启长连接时） 
Reading: 0 Writing: 1 Waiting: 1 
```

### ngx_http_random_index_module

- 随机生成首页

- 使用场景：给用户不同的首页展示感觉

```shell
Syntax:	random_index on | off;
Default:	
random_index off;
Context:	location
```

```bash
 location /random {
 		# 不会展示隐藏文件
        root /opt/app/code_random_html;
        random_index on;
 }
```

### ngx_http_sub_module

- 指定字符串替换

```bash
Syntax:	sub_filter string replacement;
Default:	—
Context:	http, server, location

Syntax:	sub_filter_last_modified on | off;
Default:	
sub_filter_last_modified off;
Context:	http, server, location
This directive appeared in version 1.5.1.

Syntax:	sub_filter_once on | off;
Default:	
sub_filter_once on;
Context:	http, server, location

Syntax:	sub_filter_types mime-type ...;
Default:	
sub_filter_types text/html;
Context:	http, server, location
```

### ngx_http_limit_conn_module & ngx_http_limit_req_module

#### 连接频率限制

```bash
Syntax:	limit_conn_zone key zone=name:size;
Default:	—
Context:	http

Syntax:	limit_conn zone number;
Default:	—
Context:	http, server, location
```

#### 请求频率限制

```bash
Syntax:	limit_req_zone key zone=name:size rate=rate [sync];
Default:	—
Context:	http

Syntax:	limit_req zone=name [burst=number] [nodelay | delay=number];
Default:	—
Context:	http, server, location
```

```bash
limit_conn_zone $binary_remote_addr zone=conn_zone:1m;
# binary_remote_addr 和 remote_addr，都表示客户端地址，binary_remote_addr更节省空间。rate=1r/ 速率 每秒一个。对同一个ip访问，进行速率限制，每秒一个
limit_req_zone $binary_remote_addr zone=req_zone:1m rate=1r/s;


location / {
	root /opt/app/code_conn_req
	 # limit_conn conn_zone 1;
	 # burst=3 超过请求频率，则 遗留3个到下一秒执行，达到访问限速，延迟响应这样一个效果。nodelay 表示直接返回503
     limit_req zone=req_zone burst=3 nodelay;
     # limit_req req_zone burst=3;
     #  limit_req zone=req_zone;
	index index.html;
}

```

- 使用工具 `ab`进行测试

```bash
# -n 请求数 -c 最大并发数
ab -n 20 -c 20 http://192.168.163.201:80/index.html
```

### ngx_http_access_module （基于IP的访问控制）

```bash
Syntax:	allow address | CIDR | unix: | all;
Default:	—
Context:	http, server, location, limit_except

Syntax:	deny address | CIDR | unix: | all;
Default:	—
Context:	http, server, location, limit_except
```

```bash
 location ~ ^/access.html {
        root /opt/app/code_access;
        deny 192.168.163.128;
        deny  192.168.163.201;
        allow 192.168.163.1;
        index index.html;
    }
```

#### 局限性

- nginx`ngx_http_access_module` 是基于 `$remote_addr` 来识客户端ip的

<img src ="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.46z15z24rj00.webp" width="65%" />

#### http_x_forwarded_for

- http head 中常用的变量

<img src ="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.6vvihm5g7wo0.webp" width="65%" />

#### http _ access _ module 局限性解决办法

1. 采用别的http头信息控制访问，如：`http_x_forwarded_for` （但这个也有要求，cdn厂商、代理方不一定按照要求做。另外还存在被篡改的可能性）
2. 结合geo模块
3. 通过http自定义变量传递（在http头规定一个变量，联系所有上一级设备，手动把 `$remote_addr` 的信息携带到我们规定的变量里面去。这样既可以避免像 `http_x_forwarded_for` 被改写，也可以准确读到客户端的ip地址）

### ngx_http_auth_basic_module （基于用户的信任登录）

```bash
Syntax:	auth_basic string | off;
Default:	
auth_basic off;
Context:	http, server, location, limit_except

Syntax:	auth_basic_user_file file;
Default:	—
Context:	http, server, location, limit_except
```

```bash
[root@localhost nginx]# htpasswd -c ./auth_conf sonzonzy
New password:
Re-type new password:
Adding password for user sonzonzy

[root@localhost nginx]# cat auth_conf
sonzonzy:$apr1$VkobqTlq$kdqJWa831J8phMplUdYt90
```

```bash
    location ~ ^/access.html {
        root /opt/app/code_access;
        auth_basic           "closed site";
        auth_basic_user_file /etc/nginx/auth_conf;
        index index.html;
    }
```

#### 局限性

- 用户信息依赖文件方式
- 操作管理机械，效率底下

#### 解决局限性

- nginx 结合 lua 实现高效验证
- nginx和ldap打通，利用 nginx-auth-ldap模块

# 场景实践

## Nginx作为静态资源web服务

### 静态资源web服务相关配置

- 静态资源类型

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/4693d69b183208f24ab65ba1d33028b.slr3oozhg6o.webp" width="80%"/>

- 静态资源服务场景-CDN

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.ny2gyp55en4.webp" width="80%"/>

- 文件读取

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.qbmthl454z4.webp" width="60%"/>

- tcp_nopush（多个传输包进行整合，一次发送出去。适用于大文件场景）

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.39961x4hd340.webp" width="60%"/>

- tcp_nodelay（不等待，实时发送，适用于对传输实时性比较高的场景）

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.3a6n93k83h20.webp" width="60%" />

- 压缩传输（减少带宽资源，传输文件大小。提高传输效率）

```nginx
Syntax:	gzip on | off;
Default:	
gzip off;
Context:	http, server, location, if in location
```

```nginx
# 压缩级别、比率。好处：压缩比率大，传输文件小，效率高。坏处：压缩本身就要耗性能
Syntax:	gzip_comp_level level;
Default:	
gzip_comp_level 1;
Context:	http, server, location
```

```nginx
# gzip http 协议版本 主流 1.1
Syntax:	gzip_http_version 1.0 | 1.1;
Default:	
gzip_http_version 1.1;
Context:	http, server, location
```

```nginx
# 预读gzip （先去目录下找同名的 gz文件 减少cpu压缩时间，压缩性能损耗）
Syntax:	gzip_static on | off | always;
Default:	
gzip_static off;
Context:	http, server, location
```

```nginx
# gunzip 解决部分浏览器无法进行gzip压缩
Syntax:	gunzip on | off;
Default:	
gunzip off;
Context:	http, server, location
```

```nginx
server {
    listen       80;
    server_name  localhost;

    sendfile on;
    #charset koi8-r;
    access_log  /var/log/nginx/access.log  main;


    location ~ .*\.(jpg|gif|png)$ {
        gzip off;
        gzip_http_version 1.1;
        gzip_comp_level 2;
        gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
        root  /opt/app/code/images;
    }

    location ~ .*\.(txt|xml)$ {
        gzip off;
        gzip_http_version 1.1;
        gzip_comp_level 1;
        gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
        root  /opt/app/code/doc;
    }

    location ~ ^/download {
        gzip_static off;
        tcp_nopush on;
        root /opt/app/code;
    }

    error_page   500 502 503 504 404  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

### 浏览器缓存

- http协议定义的缓存机制（如 Expires；Cache-control）

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.1rsot6o5v5uo.webp" width="80%"/>

- 缓存校验过期机制（expires http1.0版本 ，Cache-Control http1.1版本）

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.5zy4ev2morw.webp" width="80%"/>

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.5f9qg6jggm0.webp" width="80%"/>

- expires

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.5lpqimpu2p80.webp" width="80%"/>

### 跨域

- 为什么浏览器禁止跨域访问

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.6j59fpj9b140.webp" width="80%"/>

- 允许跨域

```nginx
Syntax:	add_header name value [always];
Default:	—
Context:	http, server, location, if in location

Access-Control-Allow-Origin
```

```nginx
location ~ .*\.(htm|html)$ {
        add_header Access-Control-Allow-Origin *;
        add_header Access-Control-Allow-Methods GET,POST,PUT,DELETE,OPTIONS;
        root  /opt/app/code;
    }
```

### 防盗链

- 目的：防止资源被盗用
- 防盗链设置思路：区别哪些请求是非正常的用户请求

```nginx
Syntax:	valid_referers none | blocked | server_names | string ...;
Default:	—
Context:	server, location
```

```nginx
valid_referers none blocked 116.62.103.228 jeson.imoocc.com ~wei\.png;
if ($invalid_referer) {
    return 403;
}
root  /opt/app/code/images;
```

## Nginx作为代理服务

### 代理分类

- 按应用场景分为正向代理和反向代理

- 正向代理

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.13yo9bbezcv4.webp" width="50%"/>

- 反向代理

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.sg1icdn13tc.webp" width="50%"/>

### Nginx可支持的代理协议

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.1vyne8343hvk.webp" width="65%"/>

- Nginx作为反向代理支持的协议

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.3l728c8m6520.webp" width="65%" />

- 反向代理模式与nginx代理模块

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.42l3gl0jaba0.webp" width="60%"/>

- Nginx作为正向代理支持的协议

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.4431yi9hnzk0.webp" width="60%"/>

### http_pass

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.41w6nf29vw60.webp" width="65%" />

```nginx
```





## Nginx作为缓存服务（代理缓存）

- 缓存：请求集中在前端，减轻后端压力
- 缓存分类：客户端缓存、代理缓存、服务器缓存

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.2i34i938qx20.webp" width="65%"/>

- **proxy_cache_path**

```nginx
Syntax:	proxy_cache_path path [levels=levels] [use_temp_path=on|off] keys_zone=name:size [inactive=time] [max_size=size] [min_free=size] [manager_files=number] [manager_sleep=time] [manager_threshold=time] [loader_files=number] [loader_sleep=time] [loader_threshold=time] [purger=on|off] [purger_files=number] [purger_sleep=time] [purger_threshold=time];
Default:	—
Context:	http
```

- **proxy_cache**

```nginx
Syntax:	proxy_cache zone | off;
Default:	
proxy_cache off;
Context:	http, server, location
```

- **proxy_cache_valid**

```nginx
Syntax:	proxy_cache_valid [code ...] time;
Default:	—
Context:	http, server, location
```

- **proxy_cache_key**

```nginx
Syntax:	proxy_cache_key string;
Default:	
proxy_cache_key $scheme$proxy_host$request_uri;
Context:	http, server, location
```









## Nginx作为负载均衡服务









# 备注

- sendfile需要进一步了解
- 正向代理这块暂时没办法测试
- 
