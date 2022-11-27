---
title: 《Nginx体系化深度精讲》study notes
author: ratears
date: 2019-05-24 02:47:23
updated: 2022-05-24 02:47:23
categories:
  - Nginx
tags:
  - Nginx
---

# Nginx初体验

## Nginx概念

- Nginx (engine x) 是一个**高性能的HTTP和反向代理web服务器**，同时也提供了IMAP/POP3/SMTP服务。Nginx是由伊戈尔·赛索耶夫为俄罗斯访问量第二的Rambler.ru站点（俄文：Рамблер）开发的，公开版本1.19.6发布于2020年12月15日。

- 其将源代码以类BSD许可证的形式发布，因它的稳定性、丰富的功能集、简单的配置文件和低系统资源的消耗而闻名。2022年01月25日，nginx 1.21.6发布。
- Nginx是一款轻量级的Web 服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器，在BSD-like 协议下发行。其特点是占有内存少，并发能力强，事实上nginx的并发能力在同类型的网页服务器中表现较好。

<br>

<br>

## Nginx缘起历史

- 互联网数据的快速增长
- Apache处理请求的低效性

|        Apache        |        Nginx         |
| :------------------: | :------------------: |
| 一个进程处理一个请求 | 一个进程处理多个请求 |
|       阻塞式的       |      非阻塞式的      |

<br>

<br>

## Nginx三个主要企业应用场景

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.4z8gliurmhc0.webp" width="90%" />

<br>

<br>

## Nginx核心优势

- 高并发、高性能
- 扩展性好
- 异步非阻塞的事件驱动模型
- 高可靠性
- 热部署

<br>

<br>

## 安装rpm包Nginx

```shell
# （1）下载epel yum源
yum install epel-release -y

# （2）查看yum源里可安装的nginx
yum list all |grep nginx

# （3）下载nginx
yum install nginx -y

# （4）列出 nginx 安装的文件
rpm -ql nginx
	
# （5）查看nginx启动文件所在目录
rpm -ql nginx |grep bin

# Nginx的二进制文件所在目录
/usr/sbin/nginx
```

<br>

<br>

<br>

# Nginx进程结构与热部署

## 多进程与多线程

### 多进程



## Nginx的进程结构

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/56b03da1dfa88b5a292f320a90fbb9d.2bzqwmpmcuas.webp" width="70%"/>

- 真正处理请求的不是 master process，二是 worker process

<br>

<br>

## Linux的信号量管理机制

- linux中的所有信号量

```bash
[root@localhost nginx]# kill -l
 1) SIGHUP       2) SIGINT       3) SIGQUIT      4) SIGILL       5) SIGTRAP
 6) SIGABRT      7) SIGBUS       8) SIGFPE       9) SIGKILL     10) SIGUSR1
11) SIGSEGV     12) SIGUSR2     13) SIGPIPE     14) SIGALRM     15) SIGTERM
16) SIGSTKFLT   17) SIGCHLD     18) SIGCONT     19) SIGSTOP     20) SIGTSTP
21) SIGTTIN     22) SIGTTOU     23) SIGURG      24) SIGXCPU     25) SIGXFSZ
26) SIGVTALRM   27) SIGPROF     28) SIGWINCH    29) SIGIO       30) SIGPWR
31) SIGSYS      34) SIGRTMIN    35) SIGRTMIN+1  36) SIGRTMIN+2  37) SIGRTMIN+3
38) SIGRTMIN+4  39) SIGRTMIN+5  40) SIGRTMIN+6  41) SIGRTMIN+7  42) SIGRTMIN+8
43) SIGRTMIN+9  44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13
48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12
53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9  56) SIGRTMAX-8  57) SIGRTMAX-7
58) SIGRTMAX-6  59) SIGRTMAX-5  60) SIGRTMAX-4  61) SIGRTMAX-3  62) SIGRTMAX-2
63) SIGRTMAX-1  64) SIGRTMAX
```

- 常用信号量

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.2l38lbs30tc0.webp" width="50%"/>

<br>

<br>

## 利用信号量管理Nginx

```bash
# 关闭nginx
kill -s SIGTERM [nginx master进程pid]

# 重新读取配置文件，会关闭之前的work子进程，生成新的work子进程 
kill -s SIGHUP [nginx master进程pid]
```

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.682s0oj5i400.webp" width="50%"/>

<br>

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.4ouapvas0x40.webp" width="50%"/>

<br/>

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.zftbgjpnhzk.webp" width="50%"/>

<br>

<br>

## 配置文件重载的原理真相

### `reload` 重载配置文件的流程

1. 向master进程发送HUP信号（reload命令）
2. master进程检查配置语法是否正确
3. master进程打开监听端口
4. master进程使用新的配置文件启动新的worker子进程
5. master进程向老的worker子进程发送QUIT信号
6. 旧的worker进程关闭监听句柄，处理完当前连接后关闭进程



<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.7cv9qios6300.webp" width="65%" />

<br>

<br>

## Nginx的热部署

### 热升级的流程（前提：新旧Nginx的编译目录一致）

1. 将旧的nginx文件替换成新的nginx文件（二进制主程序文件）
2. 向master进程发送USR2信号
3. master进程修改pid文件，加后缀.oldbin
4. master进程用新nginx文件启动新master进程
5. 向旧的master进程发送WINCH信号，旧的worker子进程退出
6. 回滚槽形：向旧master发送HUP ,向新的master发送QUIT

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.3s80krndr7m0.webp" width="75%" />

<br>

### Nginx热部署完整步骤演示

#### Nginx热升级核心命令解释

```bash
# 备份nginx二进制文件
cp nginx nginx.bak

# 生成新的nginx 主进程 和子进程（新老master、worker进程共存）
kill -s SIGUSR2 [pid]

# 关闭nginx master进程下的worker子进程
kill -s SIGWINCH [pid]

# 关闭进程
kill -s SIGQUIT [pid]

# 让旧master进程 启动work子进程（使用的还是旧的nginx二进制文件）
kill -s SIGHUP [pid]
```

- 注意事项

> 不能使用 `kill -s SIGHU []` 退出旧的master进程，如果这样做的话，旧的 master主进程和其子进程会直接退出（被kill掉），这样（当新启的nginx进程有问题时）就无法回滚。
>
> 所以应该使用 `kill -s SIGWINCH []`，先将旧的master主进程下的work子进程全部kill掉，验证新启的nginx进程没有问题后，再使用 `kill -s SIGHU []`	kill掉旧的master进程。如果验证新启的nginx进程有问题，这时使用 `kill -s SIGHUP []` 就可以让旧master进程重新启动work子进程。（达到回滚的效果）

<br/>

#### Nginx正常热升级详细步骤演示

```bash
# （1）查看nginx处于正常运行状态
[root@bogon nginx]# ps -ef |grep nginx
root     105723   1480  0 16:35 pts/0    00:00:00 grep --color=auto nginx
[root@bogon nginx]# /usr/local/nginx/sbin/nginx
[root@bogon nginx]# ps -ef |grep nginx
root     105739      1  0 16:35 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
nginx    105740 105739  0 16:35 ?        00:00:00 nginx: worker process
nginx    105741 105739  0 16:35 ?        00:00:00 nginx: worker process
nginx    105742 105739  0 16:35 ?        00:00:00 nginx: worker process
nginx    105743 105739  0 16:35 ?        00:00:00 nginx: worker process
root     105747   1480  0 16:35 pts/0    00:00:00 grep --color=auto nginx

# （2）备份nginx的二进制文件，并将新的nginx替换成旧的nginx
[root@bogon nginx]# cd /usr/local/nginx/sbin/
[root@bogon sbin]# ll
总用量 6128
-rwxr-xr-x. 1 root root 6272440 10月  6 16:16 nginx
[root@bogon sbin]# cp nginx nginx.old
[root@bogon sbin]# ll
总用量 12256
-rwxr-xr-x. 1 root root 6272440 10月  6 16:16 nginx
-rwxr-xr-x. 1 root root 6272440 10月  6 16:36 nginx.old

# （3）给旧的nginx master 进程发送 SIGUSR2信号后。nginx 新旧master进程、worker子进程共存。
[root@bogon sbin]# ps -ef |grep nginx
root     105739      1  0 16:35 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
nginx    105740 105739  0 16:35 ?        00:00:00 nginx: worker process
nginx    105741 105739  0 16:35 ?        00:00:00 nginx: worker process
nginx    105742 105739  0 16:35 ?        00:00:00 nginx: worker process
nginx    105743 105739  0 16:35 ?        00:00:00 nginx: worker process
root     106243   1480  0 16:45 pts/0    00:00:00 grep --color=auto nginx
[root@bogon sbin]# kill -s SIGUSR2 105739
[root@bogon sbin]# ps -ef |grep nginx
root     105739      1  0 16:35 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
nginx    105740 105739  0 16:35 ?        00:00:00 nginx: worker process
nginx    105741 105739  0 16:35 ?        00:00:00 nginx: worker process
nginx    105742 105739  0 16:35 ?        00:00:00 nginx: worker process
nginx    105743 105739  0 16:35 ?        00:00:00 nginx: worker process
root     106255 105739  0 16:45 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
nginx    106256 106255  0 16:45 ?        00:00:00 nginx: worker process
nginx    106257 106255  0 16:45 ?        00:00:00 nginx: worker process
nginx    106258 106255  0 16:45 ?        00:00:00 nginx: worker process
nginx    106259 106255  0 16:45 ?        00:00:00 nginx: worker process
root     106268   1480  0 16:45 pts/0    00:00:00 grep --color=auto nginx


[root@bogon nginx]# ll pid/
总用量 8
-rw-r--r--. 1 root root 7 10月  6 16:45 nginx.pid
-rw-r--r--. 1 root root 7 10月  6 16:35 nginx.pid.oldbin
[root@bogon nginx]# cat pid/nginx.pid
106255
[root@bogon nginx]# cat pid/nginx.pid.oldbin
105739


# （4）向旧的master进程发送 SIGWINCH 信号，旧的 worker子进程退出。此时，所有请求会被新的worker进程处理
[root@bogon nginx]# kill -s SIGWINCH 105739
[root@bogon nginx]# ps -ef |grep nginx
root     105739      1  0 16:35 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
root     106255 105739  0 16:45 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
nginx    106256 106255  0 16:45 ?        00:00:00 nginx: worker process
nginx    106257 106255  0 16:45 ?        00:00:00 nginx: worker process
nginx    106258 106255  0 16:45 ?        00:00:00 nginx: worker process
nginx    106259 106255  0 16:45 ?        00:00:00 nginx: worker process
root     106643   1480  0 16:52 pts/0    00:00:00 grep --color=auto nginx

# （5）验证新的ngixn 的 worker子进程没有错误后，向旧的master进程发送 SIGQUIT 信号。此时 旧的master进程退出。
# 至此，nginx热升级成功
[root@bogon nginx]# kill -s SIGQUIT 105739
[root@bogon nginx]#
[root@bogon nginx]# ps -ef |grep nginx
root     106255      1  0 16:45 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
nginx    106256 106255  0 16:45 ?        00:00:00 nginx: worker process
nginx    106257 106255  0 16:45 ?        00:00:00 nginx: worker process
nginx    106258 106255  0 16:45 ?        00:00:00 nginx: worker process
nginx    106259 106255  0 16:45 ?        00:00:00 nginx: worker process
root     106814   1480  0 16:56 pts/0    00:00:00 grep --color=auto nginx

[root@bogon nginx]# ll pid/
总用量 4
-rw-r--r--. 1 root root 7 10月  6 16:45 nginx.pid
[root@bogon nginx]# cat pid/nginx.pid
106255

[root@bogon nginx]# ll sbin/
总用量 12256
-rwxr-xr-x. 1 root root 6272440 10月  6 16:16 nginx
-rwxr-xr-x. 1 root root 6272440 10月  6 16:36 nginx.old
[root@bogon nginx]# rm -rf sbin/nginx.old
```

<br/>

#### Nginx热升级——回滚情形演示

```bash
# （1）~（2）查看nginx处于正常运行状态；备份nginx的二进制文件，并将新的nginx替换成旧的nginx
[root@bogon nginx]# ll sbin/
总用量 6128
-rwxr-xr-x. 1 root root 6272440 10月  6 16:16 nginx
[root@bogon nginx]# cp sbin/nginx sbin/nginx.bak
[root@bogon nginx]# ll sbin/
总用量 12256
-rwxr-xr-x. 1 root root 6272440 10月  6 16:16 nginx
-rwxr-xr-x. 1 root root 6272440 10月  6 17:01 nginx.bak
[root@bogon nginx]# ps -ef |grep nginx
root     106255      1  0 16:45 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
nginx    106256 106255  0 16:45 ?        00:00:00 nginx: worker process
nginx    106257 106255  0 16:45 ?        00:00:00 nginx: worker process
nginx    106258 106255  0 16:45 ?        00:00:00 nginx: worker process
nginx    106259 106255  0 16:45 ?        00:00:00 nginx: worker process
root     107093   1480  0 17:01 pts/0    00:00:00 grep --color=auto nginx

# （3）给旧的nginx master 进程发送 SIGUSR2信号后。nginx 新旧master进程、worker子进程共存。
[root@bogon nginx]# kill -s SIGUSR2 106255
[root@bogon nginx]# ps -ef |grep nginx
root     106255      1  0 16:45 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
nginx    106256 106255  0 16:45 ?        00:00:00 nginx: worker process
nginx    106257 106255  0 16:45 ?        00:00:00 nginx: worker process
nginx    106258 106255  0 16:45 ?        00:00:00 nginx: worker process
nginx    106259 106255  0 16:45 ?        00:00:00 nginx: worker process
root     107206 106255  0 17:03 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
nginx    107207 107206  0 17:03 ?        00:00:00 nginx: worker process
nginx    107208 107206  0 17:03 ?        00:00:00 nginx: worker process
nginx    107209 107206  0 17:03 ?        00:00:00 nginx: worker process
nginx    107210 107206  0 17:03 ?        00:00:00 nginx: worker process
root     107217   1480  0 17:04 pts/0    00:00:00 grep --color=auto nginx

# （4）向旧的master进程发送 SIGWINCH 信号，旧的 worker子进程退出。此时，所有请求会被新的worker进程处理
[root@bogon nginx]# kill -s SIGWINCH 106255
[root@bogon nginx]# ps -ef |grep nginx
root     106255      1  0 16:45 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
root     107206 106255  0 17:03 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
nginx    107207 107206  0 17:03 ?        00:00:00 nginx: worker process
nginx    107208 107206  0 17:03 ?        00:00:00 nginx: worker process
nginx    107209 107206  0 17:03 ?        00:00:00 nginx: worker process
nginx    107210 107206  0 17:03 ?        00:00:00 nginx: worker process
root     107264   1480  0 17:04 pts/0    00:00:00 grep --color=auto nginx

# （5）验证新的ngixn 的 worker子进程 发现有错误，开启回滚：
# 向旧的master 发送 SIGHUP 信号；向 新的master进程发送 SIGQUIT 信号
[root@bogon nginx]# kill -s SIGWINCH 106255
[root@bogon nginx]# ps -ef |grep nginx
root     106255      1  0 16:45 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
root     107206 106255  0 17:03 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
nginx    107207 107206  0 17:03 ?        00:00:00 nginx: worker process
nginx    107208 107206  0 17:03 ?        00:00:00 nginx: worker process
nginx    107209 107206  0 17:03 ?        00:00:00 nginx: worker process
nginx    107210 107206  0 17:03 ?        00:00:00 nginx: worker process
root     107264   1480  0 17:04 pts/0    00:00:00 grep --color=auto nginx
[root@bogon nginx]# kill -s SIGHUP 106255
[root@bogon nginx]# ps -ef |grep nginx
root     106255      1  0 16:45 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
root     107206 106255  0 17:03 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
nginx    107207 107206  0 17:03 ?        00:00:00 nginx: worker process
nginx    107208 107206  0 17:03 ?        00:00:00 nginx: worker process
nginx    107209 107206  0 17:03 ?        00:00:00 nginx: worker process
nginx    107210 107206  0 17:03 ?        00:00:00 nginx: worker process
nginx    107401 106255  0 17:07 ?        00:00:00 nginx: worker process
nginx    107402 106255  0 17:07 ?        00:00:00 nginx: worker process
nginx    107403 106255  0 17:07 ?        00:00:00 nginx: worker process
nginx    107404 106255  0 17:07 ?        00:00:00 nginx: worker process
root     107412   1480  0 17:07 pts/0    00:00:00 grep --color=auto nginx
[root@bogon nginx]# kill -s SIGQUIT 107206
[root@bogon nginx]# ps -ef |grep nginx
root     106255      1  0 16:45 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
nginx    107401 106255  0 17:07 ?        00:00:00 nginx: worker process
nginx    107402 106255  0 17:07 ?        00:00:00 nginx: worker process
nginx    107403 106255  0 17:07 ?        00:00:00 nginx: worker process
nginx    107404 106255  0 17:07 ?        00:00:00 nginx: worker process
root     107440   1480  0 17:08 pts/0    00:00:00 grep --color=auto nginx

# 至此，nginx热升级回滚成功
```

<br>

<br>

## Nginx模块化设计机制

### 模块结构图

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.5piekf14yes0.webp" width="60%" />

<br/>

<br>

### 模块体系结构

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.2miqh0m6jdo0.webp" width="40%" />

<br/>

<br>

## Nginx编译安装的配置参数

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.7jfdxqlxm340.webp" width="60%"/>

<br>

<br>

## 定制编译安装Nginx

```bash
# （0）准备nginx的管理用户
[root@bogon install]# useradd nginx

# （1）准备安装文件
[root@localhost install]# wget https://nginx.org/download/nginx-1.22.1.tar.gz

[root@bogon install]# ll
总用量 3688
-rw-r--r--. 1 root root 1073322 5月  24 22:29 nginx-1.22.0.tar.gz
-rw-r--r--. 1 root root 2085854 10月  6 16:08 pcre-8.43.tar.gz
-rw-r--r--. 1 root root  607698 1月  16 2017 zlib-1.2.11.tar.gz


# （2）解压安装文件
[root@bogon install]# ll
总用量 16
drwxr-xr-x.  8 1001  1001  158 5月  24 07:59 nginx-1.22.0
drwxr-xr-x.  7 1169  1169 8192 2月  24 2019 pcre-8.43
drwxr-xr-x. 14  501 games 4096 1月  16 2017 zlib-1.2.11

# （3）解压后进入 nginx源码目录，查看相关编译参数
[root@bogon nginx-1.22.0]# ./configure --help

# （4）编译安装前下载相关依赖
[root@bogon nginx-1.22.0]# yum install -y gcc gcc-c++ pcre pcre-devel zlib zlib-devel openssl openssl-devel gd gd-devel

# （5）编译安装
[root@bogon nginx-1.22.0]# ./configure --prefix=/usr/local/nginx --conf-path=/usr/local/nginx/conf/nginx.conf --user=nginx --group=nginx --pid-path=/usr/local/nginx/pid/nginx.pid --error-log-path=/usr/local/nginx/logs/error.log --with-pcre=/install/pcre-8.43 --with-zlib=/install/zlib-1.2.11 --with-http_ssl_module --with-http_image_filter_module --with-http_stub_status_module --http-log-path=/usr/local/nginx/logs/access.log

[root@bogon nginx-1.22.0]# make

[root@bogon nginx-1.22.0]# make install

# 至次，nginx编译安装完成

[root@bogon nginx-1.22.0]# cd /usr/local/nginx/
[root@bogon nginx]# ll
总用量 4
drwxr-xr-x. 2 root root 4096 10月  6 16:16 conf
drwxr-xr-x. 2 root root   40 10月  6 16:16 html
drwxr-xr-x. 2 root root    6 10月  6 16:16 logs
drwxr-xr-x. 2 root root    6 10月  6 16:16 pid
drwxr-xr-x. 2 root root   19 10月  6 16:16 sbin

[root@bogon nginx]# sbin/nginx
[root@bogon nginx]# ps -ef | grep nginx
root     104771      1  0 16:18 ?        00:00:00 nginx: master process sbin/nginx
nginx    104772 104771  0 16:18 ?        00:00:00 nginx: worker process
root     104780   1480  0 16:18 pts/0    00:00:00 grep --color=auto nginx
```

<br>

<br>

## Nginx配置文件结构

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.69cd7t6e47k0.webp" width="65%"/>

<br>

<br>

## 虚拟主机的分类（三种）

- 基于多IP的虚拟主机
  - 多网卡多IP
  - 单网卡多IP
- 基于多端口的虚拟主机
- 基于域名的虚拟主机

<br>

### 基于多网卡的虚拟主机实现

- 这里我们使用多网卡多IP的方式

```nginx
server {
	listen	192.168.146.132:8080;
	server_name	localhost;
	location / {
		root	/usr/local/nginx/html/virtual_host_by_ip/132;
		index	index.html;
	}
}
server {
	listen	192.168.146.133:8080;
	server_name	localhost;
	location / {
		root	/usr/local/nginx/html/virtual_host_by_ip/133;
		index	index.html;
	}
}
server {
	listen	192.168.146.134:8080;
	server_name	localhost;
	location / {
		root	/usr/local/nginx/html/virtual_host_by_ip/134;
		index	index.html;
	}
}
```



<br>

### 基于端口的虚拟主机实现

```nginx
server {
	listen	90;
	server_name	localhost;
	location / {
		root	/usr/local/nginx/html/virtual_host_by_port/90;
		index	index.html;
	}
}
server {
	listen	91;
	server_name	localhost;
	location / {
		root	/usr/local/nginx/html/virtual_host_by_port/91;
		index	index.html;
	}
}
server {
	listen	92;
	server_name	localhost;
	location / {
		root	/usr/local/nginx/html/virtual_host_by_port/92;
		index	index.html;
	}
}
```

<br>

### 基于域名的虚拟主机实现

```nginx
server {
	listen	9090;
	server_name	test1.nginx.com;
	location / {
		root	/usr/local/nginx/html/virtual_host_by_domain/test1;
		index	index.html;
	}
}
server {
	listen	9090;
	server_name	test2.nginx.com;
	location / {
		root	/usr/local/nginx/html/virtual_host_by_domain/test2;
		index	index.html;
	}
}
server {
	listen	9090;
	server_name	test3.nginx.com;
	location / {
		root	/usr/local/nginx/html/virtual_host_by_domain/test3;
		index	index.html;
	}
}
```

<br>

<br>

<br>

# 核心指令-Nginx基础应用

## 配置文件main段核心参数用法

```nginx
# main段核心参数:
user USERNAME [GROUP]
    解释：指定运行nginx的worker子进程的属主和属组，其中属组可以不指定 
	示例：
		user nginx nginx;

pid DIR
    解释：指定运行nginx的master主进程的pid文件存放路径 
	示例：
		pid /ropt/nginx/logs/nginx.pid;

worker_rlimit_nofile number
   	解春：指定worker子进程可以打开的最大文件句柄数 
	示例：
         worker_rlimit_nofile 20480;

worker_rlimit_core size
 	一 解释：指定worker子进程异常终止后的core文件，用于记录分析问题 
	示例：
         worker_rlimit_core 50M;
         working_directory /opt/nginx/tmp;

worker_processes number | auto
   解释：指定nginx启动的worker子进程数量
   示例：
         worker_processes 4;
         worker_processes auto;

worker_cpu_affinity cpumaskl cpumask2...
   解霹：将每个worker子进程与我们的CPU物理核心绑定。
   示例：
         worker_cpu_affinity 0001 0010 0100 1000； #
         4个物理核心，4个worker子进程
         worker_cpu_affinity 00000001 00000010 00000100 00001000 00010000
         00100060 01000000 10000000; # 8物理核心，8个worker子进程
         worker_cpu_affinity 01 10 01 10;         # 2个物理核心，4个子进程
  备注：将每个worker子进程与特定CPU物理核心绑定，优势在于：避免同个worker子进程
  在不同的CPU核心上切换，缓存失效，降低性能；其并不能真正的避免进程切换

worker_priority number
   解释：指定worker子进程的nice值，以调整运行nginx的优先级，通常设定为负值，以优先 调用nginx
   示例：
        worker_priority -10;
   备注：Linux默认进程的优先级值是120,值越小越优先；nice设定范围为-20到+19

worker_shutdown_timeout time
   解春：指定证WOrker子进程优雅退出时的超时时间
   示例：
        worker_shutdown_timeout 5s;

timer_resolution time
   解释：worker子进程内部使用的计时器精度，调整时间间隔越大，系统调用越少，有利于性 能提升；反之，系统调用越多，性能下降
   示例：
        worker_resolution 100ms;

daemon on|off
  解释：设定nginx的运行方式，前台还是后台，前台用户调试，后台用于生产 示例：
       daemon off;

lock_file DIR
  解释：负载均衡互斥锁文件存放路径
       lock_file logs/nginx.lock
	
```

<br>

## 配置文件events段核心参数用法

|        参数        |                 含义                 |
| :----------------: | :----------------------------------: |
|        use         |      nginx使用何种事件驱动模型       |
| worker_connections | worker子进程能够处理的最大并发连接数 |
|    accept_mutex    |        是否打开负载均衡互斥锁        |
| accept_mutex_delay |  新连接分配给worker子进程的超时时间  |
|    muti_accept     |   worker子进程可以接收的新连接个数   |



|        参数        |           语法            |                      可选值                       |         默认配置         |                     推荐配置                     |
| :----------------: | :-----------------------: | :-----------------------------------------------: | :----------------------: | :----------------------------------------------: |
|        use         |        use method         | select、poll、kqueue、epoll、/dev/poll、eventport |            无            |             不指定，让nginx自己选择              |
| worker_connections | worker_connections number |                                                   | worker_connections 1024  | worker_connections 65535/worker_processes\|65535 |
|    accept_mutex    |   accept_mutex on\|off    |                      on、off                      |     accept_mutex off     |                 accept_mutex on                  |
| accept_mutex_delay |  accept_mutex_delay time  |                                                   | accept_mutex_delay 500ms |             accept_mutex_delay 200ms             |
|    muti_accept     |    muti_accept on\|off    |                      on、off                      |     muti_accept off      |                  muti_accept on                  |

<br>

## 配置文件http段核心参数用法

### server_name指令

```shell
# 可以写多个server name，可以使用正则表达式，通配符，也可以是ip 具体的域名。
server_name name1 name2 name3;
```

- 匹配优先级
  - 精确匹配 > 左侧通配符匹配 > 右侧通配符匹配 > 正则表达式匹配

<br>

### root和alias

|       |    语法     |         上下文          |       共同点        |           区别            |
| :---: | :---------: | :---------------------: | :-----------------: | :-----------------------: |
| root  | root path;  | http server location if | URI到磁盘文件的映射 | root会将定义路径与URI叠加 |
| alias | alias path; |        location         | URI到磁盘文件的映射 |     alias只取定义路径     |

<br>

### location的基础用法

```shell
# 语法：
	localtion [ = | ~ | ~* | ^~ ] uri {...}
	
# 上下文：
	server location
```

|   匹配规则   |          含义          |               示例               |
| :----------: | :--------------------: | :------------------------------: |
|      =       |        精确匹配        |   `location = /images/ {...}`    |
|      ~       |  正则匹配，区分大小写  | `location ~ \.(jpg|gif)$ {...}`  |
|      ~*      | 正则匹配，不区分大小写 | `location ~* \.(jpg|gif)$ {...}` |
|      ^~      |    匹配到即停止搜索    |   `location ^~ /images/ {...}`   |
| 不带任何符号 |                        |        `location / {...}`        |

- 匹配规则优先级

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.4dfzxu6vkws0.webp" width="45%" />

<br>

### 理解location中url结尾的反斜线

```shell
location /some-dir {

}

location /some-dir/ {

}
```

- 如果URL的结构是`https://domain.com/some-dir/`。尾部如果缺少`/`将导致重定向。因为根据约定，URL尾部的`/`表示目录，没有`/`表示文件。
  - 所以访问`/some-dir/`时，服务器会自动去该目录下找对应的默认文件。
  - 如果访问`/some-dir`的话，服务器会先去找`some-dir`文件，找不到的话会将`some-dir`当成目录，重定向到`/some-dir/`，去该目录下找默认文件。

<br>

### stub_status模块用法

- `stub_status`的使用，需要Nginx编译进去

```shell
# 语法结构
	stub_status;
# 低于1.7.5 版本：
	stub_status on;

# 上下文：
	server location
	
# 配置示例
	location /uri {
		stub_status;
	}
```

- 状态项

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.98fiesmzwzk.webp" width="65%" />

<br>

- 内嵌变量

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.5h4bf27xt4c0.webp" width="65%" />

<br>

<br>

<br>

# HTTP核心模块

## connection & request

- connection 是连接，即常说的tcp连接，三次握手，状态机
- request是请求，例如http请求，无状态的协议
- request是必须建立在connection之上的









# 学习备注

> 1. linux信号量这块的管理机制可以再抽时间深入理解一下
> 2. 第三章、4 的相关内容算是比较简单，后续实践一下
> 3. 需要理解请求过程，连接、请求
> 4. 其实第二章的内容也很重要，要对Nginx有总体上的认知，特点、优势等等，需要说出个所以然来
> 5. 多进程和多线程相关的知识还需要深入理解一下
> 6. Nginx的模块整体还需要有一个把握
> 7. location这块的内容 还需要进一步熟悉一下

<br>

<img src="" width="65%" />

<br>

<br>

