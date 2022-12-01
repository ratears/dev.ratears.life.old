---
title: Redis study notes
author: ratears
date: 2022-12-01 02:31:29
updated: 2022-12-01 02:31:29
categories:
	- [Database,Redis]
tags:
	- Redis
	- Cache
	- Database
---



# 第1章 Redis 概述

## Redis 简介

- Redis，Remote Dictionary Server，远程字典服务，由意大利人 Salvatore Sanfilippo（又名 Antirez）开发，是一个使用 ANSI C 语言编写、支持网络、可基于内存亦可持久化的日志型、NoSQL 开源内存数据库，其提供多种语言的 API。从 2010 年 3 月 15 日起，Redis 的开发工作由 VMware 主持。从2013 年 5 月开始，Redis 的开发由 Pivotal 赞助。
- Redis 这个名字的意思是 Remote Dictionary Server。[[6\]](https://en.wikipedia.org/wiki/Redis#cite_note-RedisFAQ-6) Redis 项目始于 Salvatore Sanfilippo（绰号*antirez*），Redis 的原始开发人员试图提高其意大利初创公司的可扩展性，开发了一个[实时](https://en.wikipedia.org/wiki/Real-time_computing)Web 日志分析器。在使用传统数据库系统扩展某些类型的工作负载时遇到重大问题后，Sanfilippo 于 2009 年开始在[Tcl](https://en.wikipedia.org/wiki/Tcl)中制作第一个 Redis 概念验证版本的原型。[[12\]](https://en.wikipedia.org/wiki/Redis#cite_note-12)后来 Sanfilippo 将该原型翻译成 C 语言并实现了第一种数据类型，即列表。在内部成功使用该项目几周后，Sanfilippo 决定将其开源

- Redis 之所以称之为字典服务，是因为 Redis 是一个 key-value 存储系统。支持存储的 value类型很多，包括 String(字符串)、List(链表)、Set(集合)、Zset(sorted set --有序集合)和 Hash（哈希类型）等。

<br>

## NoSQL 简介

- NoSQL（“non-relational”， “Not Only SQL”），泛指非关系型的数据库。随着互联网 web2.0网站的兴起，传统的关系数据库在处理 web2.0 网站，特别是超大规模和高并发的 SNS 类型的 web2.0 纯动态网站已经显得力不从心，出现了很多难以克服的问题，而非关系型的数据库则由于其本身的特点得到了非常迅速的发展。**NoSQL 数据库的产生就是为了解决大规模数据集合多重数据种类带来的挑战，特别是大数据应用难题。**



（1）键值存储数据库

- 就像 Map 一样的 key-value 对。典型代表就是 Redis。

（2）列存储数据库

- 关系型数据库是典型的行存储数据库。其存在的问题是，按行存储的数据在物理层面占用的是连续存储空间，不适合海量数据存储。而按列存储则可实现分布式存储，适合海量存储。典型代表是 HBase。

（3）文档型数据库

- 其是 NoSQL 与关系型数据的结合，最像关系型数据库的 NoSQL。典型代表是 MongoDB。

（4）图形(Graph)数据库

- 用于存放一个节点关系的数据库，例如描述不同人间的关系。典型代表是 Neo4J。

<br>

## Redis 的用途

- Redis 在生产中使用最多的场景就是做数据缓存。即客户端从 DBMS 中查询出的数据首先写入到 Redis 中，后续无论哪个客户端再需要访问该数据，直接读取 Redis 中的即可，不仅减小了 RT，而且降低了 DBMS 的压力。

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.cdgu076yuww.webp" width="60%">

<br>

- 根据 Redis 缓存的数据与 DBMS 中数据的同步性划分，缓存一般可划分为两类：**实时同步缓存**，与**阶段性同步缓存**。

> 实时同步缓存是指，DBMS 中数据更新后，Redis 缓存中的存放的相关数据会被立即清除，以促使再有对该数据的访问请求到来时，必须先从 DBMS 中查询获取到最新数据，然后再写入到 Redis。

> 阶段性同步缓存是指，Redis 缓存中的数据允许在一段时间内与 DBMS 中的数据不完全一致。而这个时间段就是这个缓存数据的过期时间。

<br>

## Redis 特性

- 能够做缓存的技术、中间件很多，例如，MyBatis 自带的二级缓存、Memched 等。只所以在生产中做缓存的产品几乎无一例外的会选择 Redis，是因为它有很多其它产品所不具备的特性。
- **性能极高**：Redis 读的速度可以达到 11w 次/s，写的速度可以达到 8w 次/s。只所以具有这么高的性能，因为以下几点原因：
  - （1）Redis 的所有操作都是在内存中发生的。
  - （2）Redis 是用 C 语言开发的。
  - （3）Redis 源码非常精细（集性能与优雅于一身）。
- **简单稳定**：Redis 源码很少。早期版本只有 2w 行左右。从 3.0 版本开始，增加了集群功能，代码变为了 5w 行左右。
- **持久化**：Redis 内存中的数据可以进行持久化，其有两种方式：RDB 与 AOF。
- **高可用集群**：Redis 提供了高可用的主从集群功能，可以确保系统的安全性。
- **丰富的数据类型**：Redis 是一个 key-value 存储系统。支持存储的 value 类型很多，包括String(字符串)、List(链表)、Set(集合)、Zset(sorted set --有序集合)和 Hash（哈希类型）等，还有 BitMap、HyperLogLog、Geospatial 类型。
  - BitMap：一般用于大数据量的二值性统计。
  - HyperLogLog：其是 Hyperlog Log，用于对数据量超级庞大的日志做去重统计。
  - Geospatial：地理空间，其主要用于地理位置相关的计算。
- **强大的功能**：Redis 提供了数据过期功能、发布/订阅功能、简单事务功能，还支持 Lua脚本扩展功能。
- **客户端语言广泛**：Redis 提供了简单的 TCP 通信协议，编程语言可以方便地的接入 Redis。所以，有很多的开源社区、大公司等开发出了很多语言的 Redis 客户端。
- **支持 ACL 权限控制**：之前的权限控制非常笨拙。从 Redis6 开始引入了 ACL 模块，可以为不同用户定制不同的用户权限。

> ACL，Access Control List，访问控制列表，是一种细粒度的权限管理策略，可以针对任意用户与组进行权限控制。目前大多数 Unix 系统与 Linux 2.6 版本已经支持 ACL 了。Zookeeper 早已支持 ACL 了。
>
> Unix 与 Linux 系统默认使用是 UGO（User、Group、Other）权限控制策略，其是一种粗粒度的权限管理策略。

- **支持多线程 IO 模型**：Redis 之前版本采用的是单线程模型，从 6.0 版本开始支持了多线程模型。

<br>

## Redis 的 IO 模型

- Redis 处理客户端请求所采用的处理架构，称为 Redis 的 IO 模型。不同版本的 Redis 采用的 IO 模型是不同的。

<br>

### 单线程模型

- 对于 Redis 3.0 及其以前版本，Redis 的 IO 模型采用的是<font color=red>纯粹的单线程模型</font>。即所有客户端的请求全部由一个线程处理。

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.72e1zah9knw0.webp" width="75%">

<br>

- Redis 的单线程模型采用了<font color=red>多路复用技术</font>。

> 对于多路复用器的<font color=red>多路选择算法</font>常见的有三种：select 模型、poll 模型、epoll 模型。
>
> - poll 模型的选择算法：采用的是轮询算法。该模型对客户端的就绪处理是有延迟的。
> - epoll 模型的选择算法：采用的是回调方式。根据就绪事件发生后的处理方式的不同，又可分为 LT 模型与 ET 模型。



> 每个客户端若要向 Redis 提交请求，都需要与 Redis 建立一个 socket 连接，并向事件分发器注册一个事件。一旦该事件发生就表明该连接已经就绪。而一旦连接就绪，事件分发器就会感知到，然后获取客户端通过该连接发送的请求，并将由该事件分发器所绑定的这个唯一的线程来处理。如果该线程还在处理多个任务，则将该任务写入到任务队列等待线程处理。
>
> 只所以称为事件分发器，是因为它会根据不同的就绪事件，将任务交由不同的事件处理器去处理。

<br>

### 混合线程模型

- 从 Redis 4.0 版本开始，Redis 中就开始加入了多线程元素。处理客户端请求的仍是单线程模型，但对于一些比较耗时但又不影响对客户端的响应的操作，就由后台其它线程来处理。例如，持久化、对 AOF 的 rewrite、对失效连接的清理等。

<br>

### 多线程模型

- Redis 6.0 版本，才是真正意义上的多线程模型。因为其对于客户端请求的处理采用的是多线程模型。

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.8h232my5ero.webp" width="75%">

<br>

- 多线程 IO 模型中的“多线程”仅用于接受、解析客户端的请求，然后将解析出的请求写入到任务队列。而对具体任务（命令）的处理，仍是由主线程处理。这样做使得用户无需考虑线程安全问题，无需考虑事务控制，无需考虑像 LPUSH/LPOP 等命令的执行顺序问题。

<br>

### 优缺点总结

#### 单线程模型

- 优点：可维护性高，性能高。不存在并发读写情况，所以也就不存在执行顺序的不确定性，不存在线程切换开销，不存在死锁问题，不存在为了数据安全而进行的加锁/解锁开销。

- 缺点：性能会受到影响，且由于单线程只能使用一个处理器，所以会形成处理器浪费。

<br>

#### 多线程模型

- 优点：其结合了多线程与单线程的优点，避开了它们的所有不足
- 缺点：该模型没有显示不足。如果非要找其不足的话就是，其并非是一个真正意义上的“多线程”，因为真正处理“任务”的线程仍是单线程。所以，其对性能也是有些影响的。

<br>

<br>

<br>

# 第2章 Redis 的安装与配置

## Redis 的安装

### 安装前的准备工作

> 由于 Redis 是由 C/C++语言编写的，而从官网下载的 Redis 安装包是需要编译后才可安装的，所以对其进行编译就必须要使用相关编译器。对于 C/C++语言的编译器，使用最多的是gcc 与 gcc-c++，而这两款编译器在 CentOS7 中是没有安装的，所以首先要安装这两款编译器。
>
> GCC，GNU Compiler Collection，GNU 编译器集合。



- 安装 gcc：`yum -y install gcc gcc-c++`

- 下载 Redis。将下载好的压缩包上传到 Linux 的/install/ 目录中。



<br>

### 安装 Redis

（1） 解压 Redis

```bash
# 将 Redis 解压到 /install 目录中。
tar -zxvf redis-7.0.5.tar.gz -C /install/
```



（2） 编译

```bash
# 进入到解压目录中，然后执行编译命令 make。
cd /install/redis-7.0.5/
make
```

> 编译过程是根据 Makefile 文件进行的，而 Redis 解压包中已经存在该文件了。所以可以直接进行编译了。



（3） 安装：

```bash
# 在 Linux 中对于编译过的安装包执行 make install 进行安装。
[root@bogon redis-7.0.5]# make install
cd src && make install
which: no python3 in (/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin)
make[1]: 进入目录“/install/redis-7.0.5/src”
    CC Makefile.dep
make[1]: 离开目录“/install/redis-7.0.5/src”
which: no python3 in (/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin)
make[1]: 进入目录“/install/redis-7.0.5/src”

Hint: It's a good idea to run 'make test' ;)

    INSTALL redis-server
    INSTALL redis-benchmark
    INSTALL redis-cli
make[1]: 离开目录“/install/redis-7.0.5/src”
[root@bogon redis-7.0.5]#
# 可以看到，共安装了三个组件：redis 服务器、客户端与一个性能测试工具 benchmark。

# 安装完成后，打开/usr/local/bin 目录，可以看到出现了很多的文件。
[root@bogon redis-7.0.5]# ll /usr/local/bin/
总用量 21500
-rwxr-xr-x. 1 root root  5197848 11月 15 00:29 redis-benchmark
lrwxrwxrwx. 1 root root       12 11月 15 00:29 redis-check-aof -> redis-server
lrwxrwxrwx. 1 root root       12 11月 15 00:29 redis-check-rdb -> redis-server
-rwxr-xr-x. 1 root root  5411040 11月 15 00:29 redis-cli
lrwxrwxrwx. 1 root root       12 11月 15 00:29 redis-sentinel -> redis-server
-rwxr-xr-x. 1 root root 11398056 11月 15 00:29 redis-server

# 通过 echo $PATH 可以看到，/usr/local/bin 目录是存在于该系统变量中的，这样这些命令就可以在任意目录中执行了。
[root@bogon redis-7.0.5]# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
```

<br>

### Redis 启动与停止

#### 前台启动

```bash
# 在任意目录执行 redis-server 命令即可启动 Redis。这种启动方式会占用当前命令行窗口。
redis-server

# 再开启一个会话窗口，可以查看到当前的 Redis 进程，默认端口号为 6379。
[root@bogon ~]# ps -ef |grep redis
root      23230  18578  0 00:35 pts/1    00:00:00 redis-server *:6379
root      23237  18614  0 00:36 pts/2    00:00:00 grep --color=auto redis


# 通过 Ctrl + C 命令可以停止 Redis。
```

<br>

#### 命令式后台启动

- 使用 nohup 命令，最后再添加一个&符，可以使要启动的程序在后台以守护进程方式运行。这样的好处是，进程启动后不会占用一个会话窗口，且其还会在当前目录，即运行启动命令的当前目录中创建一个 nohup.out 文件用于记录 Redis 的操作日志。

```bash
[root@bogon install]# nohup redis-server &
[1] 23241
[root@bogon install]# nohup: 忽略输入并把输出追加到"nohup.out"

[root@bogon install]# ll
总用量 2940
-rw-------. 1 root root    1195 11月 15 00:40 nohup.out
```

<br>

#### Redis 的停止

```shell
# 通过 redis-cli shutdown 命令可以停止 Redis。
[root@bogon install]# redis-cli shutdown
[1]+  完成                  nohup redis-server
```

<br>

#### 配置式后台启动

- 使用 nohup 命令可以使 Redis 后台启动，但每次都要键入 nohup 与&符，比较麻烦。可以通过修改 Linux 中 Redis 的核心配置文件 redis.conf 达到后台启动的目的。redis.conf 文件在Redis 的安装目录根下。

```bash
cp /install/redis-7.0.5/redis.conf /usr/local/bin/

vim /usr/local/bin/redis.conf

# 将 daemonize 属性值由 no 改为 yes，使 Redis 进程以守护进程方式运行。
daemonize yes
```

- 修改后再启动 Redis，就无需再键入 nohup 与&符了，但必须要指定启动所使用的 Redis配置文件。

> 使用 nohup redis-server &命令启动 Redis 时，启动项中已经设置好了 Redis 各个参数的默认值，Redis 会按照这些设置的参数进行启动。但这些参数是可以在配置文件中进行修改的，修改后，需要在启动命令中指定要加载的配置文件，这样，配置文件中的参数值将覆盖原默认值。

> Redis 已经给我们提供好了配置文件模板，是 Redis 安装目录的根目录下的 redis.conf 文件。由于刚刚对 redis.conf 配置文件做了修改，所以在开启 Redis 时需要显示指出要加载的配置文件。配置文件应紧跟在 redis-server 的后面。



```bash
redis-server /usr/local/bin/redis.conf
```

<br>

### 连接前的配置

- 若要使远程主机上的客户端能够连接并访问到服务端的 Redis，则服务端首先要做如下配置。

<br>

#### 绑定客户端 IP

- Redis 通过修改配置文件来限定可以访问自己的客户端 IP。

```bash
# 默认配置如下，只允许当前主机访问当前的 Redis，其它主机均不可访问。所以，如果不想限定访问的客户端，只需要将该行注释掉即可。

# bind 127.0.0.1 -::1
```

<br>

#### 关闭保护模式

```bash
# 默认保护模式是开启的。其只允许本机的客户端访问，即只允许自己访问自己。但生产中应该关闭，以确保其它客户端可以连接 Redis。

protected-mode no
```

<br>

#### 设置访问密码

- 为 Redis 设置访问密码，可以对要读/写 Redis 的用户进行身份验证。没有密码的用户可以登录 Redis，但无法访问。



（1）密码设置

```bash
# 访问密码的设置位置在 redis.conf 配置文件中。默认是被注释掉的，没有密码。修改后需要重启redis生效

requirepass redis@2022
```



（2）使用密码

```shell
127.0.0.1:6379> auth redis@2022
OK
```

```bash
redis-cli -a redis@2022
```

```shell
redis-cli -a redis@2022 shutdown
```

<br>

#### 禁止/重命名命令

- 两个非常危险的命令：flushall与 flushdb。它们都是用于直接删除整个 Redis数据库的。若让用户可以随便使用它们，可能会危及数据安全。Redis 可以通过修改配置文件来禁止使用这些命令，或重命名这些命令。以下配置，禁用了 flushall 与 flushdb 命令。

```shell
rename-command flushall ""
rename-command flushdb ""
```

<br>

#### 重启redis

- 修改相关配置后，需要重启redis，使其配置生效

<br>

## 	Redis 客户端分类

### 命令行客户端

```bash
# -h：指定要连接的 Redis 服务器的 IP
# -p：指定要连接的 Redis 的端口号
# 若连接的是本机 Redis，且端口号没有改变，保持默认的 6379，则-h 与-p 选项可以省略不写

redis-cli -h 127.0.0.1 -p 6379
```

<br>

### 图形界面客户端

- Redis Desktop Manager
  - 官网为：https://resp.app/（原来是 http://redisdesktop.com）。
  - Redis 的图形界面客户端很多，其中较出名的是 Redis Desktop Manager 的客户端。不过，该软件原来是免费软件，从 0.8.8 版本后变为了商业化收费软件。
- RedisPlus
  - RedisPlus 的官网地址为 https://gitee.com/MaxBill/RedisPlus。
- Java 代码客户端
  - 所谓 Java 代码客户端就是一套操作 Redis 的 API，其作用就像 JDBC 一样，所以 Java 代码客户端其实就是一个或多个 Jar 包，提供了对 Redis 的操作接口。
  - 对 Redis 操作的 API 很多，例如 jdbc-redis、jredis 等，但最常用也是最有名的是 Jedis。

<br>

<br>

<br>

# 第3章 Redis 配置文件详解







<br>

<br>

<br>

# 第4章 Redis 命令

- Redis 根据命令所操作对象的不同，可以分为三大类：
  - 对 Redis 进行基础性操作的命令
  - 对 Key 的操作命令
  - 对 Value 的操作命令







<br>

<br>

<br>

# 第5章 Redis 持久化

























<br>

<br>

<br>

# 学习备注

> 1

```html
&emsp;&emsp;
<font color=red></font>
```

<br>

<br>

<br>

<img src="" width="60%">

<br>

