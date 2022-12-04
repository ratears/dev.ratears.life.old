---
title: Redis study notes
author: ratears
date: 2022-12-01 02:31:29
updated: 2022-12-01 02:31:29
categories:
	- [Database, Redis]
	- [study-notes] 
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

## Redis 基本命令

> 首先通过 redis-cli 命令进入到 Redis 命令行客户端，然后再运行下面的命令。

```bash
redis-cli -a redis@2022
```

<br>

### 心跳命令 ping

```bash
# 键入 ping 命令，会看到 PONG 响应，则说明该客户端与 Redis 的连接是正常的。该命令亦称为心跳命令。
127.0.0.1:6379> ping
PONG
```

<br>

### 读写键值命令

```bash
# set key value 会将指定 key-value 写入到 DB。get key 则会读取指定 key 的 value 值。
127.0.0.1:6379> set name lisi
OK
127.0.0.1:6379> get name
"lisi"
```

<br>

### DB 切换 select

```bash
# Redis 默认有 16 个数据库。默认使用的是 0 号 DB，可以通过 select db 索引来切换 DB。
127.0.0.1:6379> select 3
OK
127.0.0.1:6379[3]>
```

<br>

### 查看 key 数量 dbsize

```bash
# dbsize 命令可以查看当前数据库中 key 的数量。
127.0.0.1:6379[3]> dbsize
(integer) 1
127.0.0.1:6379[3]> select 0
OK
127.0.0.1:6379> dbsize
(integer) 2
```

<br>

### 删除当前库中数据 flushdb

```bash
# flushdb 命令仅仅删除的是当前数据库中的数据，不影响其它库。
# 如果使用了 禁止/重命名命令 则需要注意
```

<br>

### 删除所有库中数据命令 flushall

```bash
# flushall 命令可以删除所有库中的所有数据。所以该命令的使用一定要慎重。
# 如果使用了 禁止/重命名命令 则需要注意
```

<br>

### 退出客户端命令

```bash
# 使用 exit 或 quit 命令均可退出 Redis 命令行客户端。
127.0.0.1:6379> exit

127.0.0.1:6379> quit
```

<br>

## Key 操作命令

- Redis 中存储的数据整体是一个 Map，其 key 为 String 类型，而 value 则可以是 String、Hash 表、List、Set 等类型。



| Key 操作命令      | 格式               | 功能                                                         | 说明                                                         |
| ----------------- | :----------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| keys              | KEYS pattern       | 查找所有符合给定模式 pattern 的 key，pattern 为正则表达式    | KEYS 的速度非常快，但在一个大的数据库中使用它可能会阻塞当前服务器的服务。所以生产环境中一般不使用该命令，而使用 `scan` 命令代替 |
| exists            | EXISTS key         | 检查给定 key 是否存在                                        | 若 key 存在，返回 1 ，否则返回 0                             |
| del               | DEL key [key ...]  | 删除给定的一个或多个 key 。不存在的 key 会被忽略             | 返回被删除 key 的数量                                        |
| rename            | RENAME key newkey  | 将 key 改名为 newkey                                         | 当 key 和 newkey 相同，或者 key 不存在时，返回一个错误。当 newkey 已经存在时， RENAME 命令将覆盖旧值。改名成功时提示 OK ，失败时候返回一个错误 |
| move              | MOVE key db        | 将当前数据库的 key 移动到给定的数据库 db 当中                | 如果当前数据库(源数据库)和给定数据库(目标数据库)有相同名字的给定 key ，或者 key 不存在于当前数据库，那么 MOVE 没有任何效果。移动成功返回 1 ，失败则返回 0 。 |
| type              | TYPE key           | 返回 key 所储存的值的类型                                    | 返回值有以下六种<br/>none (key 不存在)<br/> string (字符串)<br/> list (列表)<br/> set (集合)<br/> zset (有序集)<br/> hash (哈希表) |
| expire 与 pexpire | EXPIRE key seconds | 为给定 key 设置生存时间。当 key 过期时(生存时间为 0)，它会被自动删除。expire 的时间单位为秒，pexpire 的时间单位为毫秒。在 Redis 中，带有生存时间的 key被称为“易失的”(volatile)。 | 生存时间设置成功返回 1。若 key 不存在时返回 0 。rename 操作不会改变 key的生存时间。 |
| ttl 与 pttl       | TTL key            | TTL, time to live，返回给定 key 的剩余生存时间               | 其返回值存在三种可能：<br/>（1）当 key 不存在时，返回 -2 。<br/>（2）当 key 存在但没有设置剩余生存时间时，返回 -1 。<br/>（3）否则，返回 key 的剩余生存时间。ttl 命令返回的时间单位为秒，而 pttl 命令返回的时间单位为毫秒。 |
| persist           | PERSIST key        | 去除给定 key 的生存时间，将这个 key 从“易失的”转换成“持久的” | 当生存时间移除成功时，返回 1；若 key 不存在或 key 没有设置生存时间，则返回 0。 |
| randomkey         | RANDOMKEY          | 从当前数据库中随机返回(不删除)一个 key。                     | 当数据库不为空时，返回一个 key。当数据库为空时，返回 nil     |



- **scan**
  - 格式：SCAN cursor [MATCH pattern] [COUNT count] [TYPE type]
  - 功能：用于迭代数据库中的数据库键。其各个选项的意义为：
    - cursor：本次迭代开始的游标。
    - pattern ：本次迭代要匹配的 key 的模式。 
    - count ：本次迭代要从数据集里返回多少元素，默认值为 10 。
    - type：本次迭代要返回的 value 的类型，默认为所有类型。

> SCAN 命令是一个基于游标 cursor 的迭代器：SCAN 命令每次被调用之后，都会向用户返回返回一个包含两个元素的数组， 第一个元素是用于进行下一次迭代的新游标，而第二个元素则是一个数组， 这个数组中包含了所有被迭代的元素。用户在下次迭代时需要使用这个新游标作为 SCAN 命令的游标参数，以此来延续之前的迭代过程。当SCAN 命令的游标参数被设置为 0 时，服务器将开始一次新的迭代。如果新游标返回 0表示迭代已结束。

- 说明：
  - 使用间断的、负数、超出范围或者其他非正常的游标来执行增量式迭代不会造成服务器崩溃。
  - 当数据量很大时，count 的数量的指定可能会不起作用，Redis 会自动调整每次的遍历数目。由于 scan 命令每次执行都只会返回少量元素，所以该命令可以用于生产环境，而不会出现像 KEYS 命令带来的服务器阻塞问题。
  - 增量式迭代命令所使用的算法只保证在数集的大小有界的情况下迭代才会停止，换句话说，如果被迭代数据集的大小不断地增长的话，增量式迭代命令可能永远也无法
    完成一次完整迭代。即当一个数据集不断地变大时，想要访问这个数据集中的所有元素就需要做越来越多的工作， 能否结束一个迭代取决于用户执行迭代的速度是否比数据集增长的速度更快。

- 相关命令：另外还有 3 个 scan 命令用于对三种类型的 value 进行遍历。
  - hscan：属于 Hash 型 Value 操作命令集合，用于遍历当前 db 中指定 Hash 表的所有 field-value 对。
  - sscan：属于 Set 型 Value 操作命令集合，用于遍历当前 db 中指定 set 集合的所有元素
  - zscan：属于 ZSet 型 Value 操作命令集合，用于遍历当前 db 中指定有序集合的所有元素（数值与元素值）

<br>

## String 型 Value 操作命令















<br>

<br>

<br>

# 第5章 Redis 持久化

## 概述

- Redis 是一个内存数据库，所以其运行效率非常高。但也存在一个问题：内存中的数据是不持久的，若主机宕机或 Redis 关机重启，则内存中的数据全部丢失。当然，这是不允许的。Redis 具有持久化功能，其会按照设置以快照或操作日志的形式将数据持久化到磁盘。
- 根据持久化使用技术的不同，Redis 的持久化分为两种：RDB 与 AOF。

<br>

## 持久化基本原理

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/1669979940444.ex1epa9vjog.webp" width="70%">

<br>

- Redis 持久化也称为钝化，是指将内存中数据库的状态描述信息保存到磁盘中。只不过是不同的持久化技术，对数据的状态描述信息是不同的，生成的持久化文件也是不同的。但它们的作用都是相同的：避免数据意外丢失。
- 通过手动方式，或自动定时方式，或自动条件触发方式，将内存中数据库的状态描述信息写入到指定的持久化文件中。当系统重新启动时，自动加载持久化文件，并根据文件中数据库状态描述信息将数据恢复到内存中，这个数据恢复过程也称为激活。这个钝化与激活的过程就是 Redis 持久化的基本原理。

- 对于 Redis 单机状态下，无论是手动方式，还是定时方式或条件触发方式，都存在数据丢失问题：在尚未手动/自动保存时发生了 Redis 宕机状况，那么从上次保存到宕机期间产生的数据就会丢失。不同的持久化方式，其数据的丢失率也是不同的。

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/1669980154363.74thoe883o80.webp" width="70%">

<br>

- RDB 是默认持久化方式，但 Redis 允许 RDB 与 AOF 两种持久化技术同时开启，此时系统会使用 AOF 方式做持久化，即 AOF 持久化技术的优先级要更高。同样的道理，两种技术同时开启状态下，系统启动时若两种持久化文件同时存在，则优先加载 AOF持久化文件。

<br>

## RDB 持久化

- **RDB，Redis DataBase，是指将内存中某一时刻的数据快照全量写入到指定的 rdb 文件的持久化技术。**

- RDB 持久化默认是开启的。当 Redis 启动时会自动读取 RDB 快照文件，将数据从硬盘载入到内存，以恢复 Redis 关机前的数据库状态。

<br>

### 持久化的执行

- **RDB 持久化的执行有三种方式：手动 save 命令、手动 bgsave 命令，与自动条件触发。**

<br>

#### 手动 save 命令

- 通过在 redis-cli 客户端中执行 save 命令可立即进行一次持久化保存。save 命令在执行期间会阻塞 redis-server 进程，直至持久化过程完毕。而在 redis-server 进程阻塞期间，Redis不能处理任何读写请求，无法对外提供服务。

```bash
127.0.0.1:6379[2]> save
OK
```

<br>

#### 手动 bgsave 命令

- 通过在 redis-cli 客户端中执行 bgsave 命令可立即进行一次持久化保存。不同于 save 命令的是，正如该命令的名称一样，background save，后台运行 save。bgsave 命令会使服务器进程 redis-server 生成一个子进程，由该子进程负责完成保存过程。在子进程进行保存过程中，不会阻塞 redis-server 进程对客户端读写请求的处理。

```bash
127.0.0.1:6379> bgsave
Background saving started
```

<br>

#### 自动条件触发

- 自动条件触发的本质仍是 bgsave 命令的执行。只不过是用户通过在配置文件中做相应的设置后，Redis 会根据设置信息自动调用 bgsave 命令执行

<br>

#### 查看持久化时间

- 通过 lastsave 命令可以查看最近一次执行持久化的时间，其返回的是一个 Unix 时间戳。

```bash
127.0.0.1:6379> lastsave
(integer) 1668522087
```

<br>

### RDB 优化配置

- RDB 相关的配置在 redis.conf 文件的 SNAPSHOTTING 部分。

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.5ip89bu9rsk0.webp" width="60%">

<br>

#### save

- 该配置用于设置快照的自动保存触发条件，即 save point，保存点。该触发条件是在指定时间段内发生了指定次数的写操作。除非另有规定，默认情况下持久化条件为 save 3600 1300 100 60 10000。其等价于以下三条
  - save 3600 1 # 在 3600 秒(1 小时)内发生 1 次写操作
  - save 300 100 # 在 300 秒(5 分钟)内发生 100 次写操作
  - save 60 10000 # 在 60 秒(1 分钟)内发生 1 万次写操作
- 如果不启用 RDB 持久化，只需设置 save 的参数为空串即可：save “”。

<br>

#### stop-write-on-bgsave-error

```shell
stop-writes-on-bgsave-error yes
```

- 默认情况下，如果 RDB 快照已启用（至少一个保存点），且最近的 bgsave 命令失败，Redis将停止接受写入。这样设置是为了让用户意识到数据没有正确地保存到磁盘上，否则很可能没有人会注意到，并会发生一些灾难。当然，如果 bgsave 命令后来可以正常工作了，Redis将自动允许再次写入

<br>

#### rdbcompression

```shell
rdbcompression yes
```

- 当进行持久化时启用 LZF 压缩字符串对象。虽然压缩 RDB 文件会消耗系统资源，降低性能，但可大幅降低文件的大小，方便保存到磁盘，加速主从集群中从节点的数据同步。

<br>

#### rdbchecksum

```shell
rdbchecksum yes	
```

- 从 RDB5 开始，RDB 文件的 CRC64 校验和就被放置在了文件末尾。这使格式更能抵抗 RDB文件的损坏，但在保存和加载 RDB 文件时，性能会受到影响（约 10%），因此可以设置为 no禁用校验和以获得最大性能。在禁用校验和的情况下创建的 RDB 文件的校验和为零，这将告诉加载代码跳过校验检查。默认为 yes，开启了校验功能

<br>

####  sanitize-dump-payload

- 该配置用于设置在加载 RDB 文件或进行持久化时是否开启对 zipList、listPack 等数据的全面安全检测。该检测可以降低命令处理时发生系统崩溃的可能。其可设置的值有三种选择：
  - no：不检测
  - yes：总是检测
  - clients：只有当客户端连接时检测。排除了加载 RDB 文件与进行持久化时的检测
- **默认值本应该是 clients，但其会影响 Redis 集群的工作，所以默认值为 no，不检测**

<br>

#### dbfilename

- 指定 RDB 文件的默认名称，默认为 dump.rdb

<br>

#### rdb-del-sync-files

- 主从复制时，是否删除用于同步的从机上的 RDB 文件。默认是 no，不删除。不过需要。**注意，只有当从机的 RDB 和 AOF 持久化功能都未开启时才生效。**

<br>

#### dir

- 指定 RDB 与 AOF 文件的生成目录。默认为 Redis 安装根目录

<br>

### RDB 文件结构

- RDB 持久化文件 dump.rdb 整体上有五部分构成：

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.11ghbcn60heo.webp" width="70%">

<br>

#### SOF

- SOF 是一个常量，一个字符串 REDIS，仅包含这五个字符，其长度为 5。用于标识 RDB文件的开始，以便在加载 RDB 文件时可以迅速判断出文件是否是 RDB 文件。

<br>

#### rdb_version

- 一个整数，长度为 4 字节，表示 RDB 文件的版本号

<br>

#### EOF

- EOF 是一个常量，占 1 个字节，用于标识 RDB 数据的结束，校验和的开始

<br>

#### check_sum

- 校验和 check_sum 用于判断 RDB 文件中的内容是否出现数据异常。其采用的是 CRC 校验算法。
- CRC 校验算法：

> - 在持久化时，先将 SOF、rdb_version 及内存数据库中的数据快照这三者的二进制数据拼接起来，形成一个二进制数（假设称为数 a），然后再使用这个 a 除以校验和 check_sum，此时可获取到一个余数 b，然后再将这个 b 拼接到 a 的后面，形成 databases。
>
> - 在加载时，需要先使用 check_sum 对 RDB 文件进行数据损坏验证。验证过程：只需将RDB 文件中除 EOF 与 check_sum 外的数据除以 check_sum。只要除得的余数不是 0，就说明文件发生损坏。当然，如果余数是 0，也不能肯定文件没有损坏。
> - 这种验证算法，是数据损坏校验，而不是数据没有损坏的校验。

<br>

#### databases

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.65mcvofo1ik0.webp" width="80%">

<br>

- databases 部分是 RDB 文件中最重要的数据部分，其可以包含任意多个非空数据库。而每个 database 又是由三部分构成：
  - SODB：是一个常量，占 1 个字节，用于标识一个数据库的开始。
  - db_number：数据库编号。
  - key_value_pairs：当前数据库中的键值对数据。（每个 key_value_pairs 又由很多个用于描述键值对的数据构成。）
    - VALUE_TYPE：是一个常量，占 1 个字节，用于标识该键值对中 value 的类型。
    - EXPIRETIME_UNIT：是一个常量，占 1 个字节，用于标识过期时间的单位是秒还是毫秒。
    - time：当前 key-value 的过期时间。

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.fn8asgxhol4.webp" width="100%">

<br>

### RDB 持久化过程

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.2c0fq381m9xc.webp" width="70%">

<br>

- 对于 Redis 默认的 RDB 持久化，在进行 bgsave 持久化时，redis-server 进程会 fork 出一个 bgsave 子进程，由该子进程以<font color=red>异步方式</font>负责完成持久化。而在持久化过程中，redis-server进程不会阻塞，其会继续接收并处理用户的读写请求。

- bgsave 子进程的详细工作原理如下：

> 由于子进程可以继承父进程的所有资源，且父进程不能拒绝子进程的继承权。所以，bgsave 子进程有权读取到 redis-server 进程写入到内存中的用户数据，使得将内存数据持久化到 dump.rdb 成为可能。
>
> bgsave 子进程在持久化时首先会将内存中的全量数据 copy 到磁盘中的一个 RDB 临时文件，copy 结束后，再将该文件 rename 为 dump.rdb，替换掉原来的同名文件。
>
> 不过，在进行持久化过程中，如果 redis-server 进程接收到了用户写请求，则系统会将内存中发生数据修改的物理块 copy 出一个副本。等内存中的全量数据 copy 结束后，会再将副本中的数据 copy 到 RDB 临时文件。这个副本的生成是由于 Linux 系统的写时复制技术（Copy-On-Write）实现的。

<img src="" width="60%">

<br>

> **写时复制技术是 Linux 系统的一种进程管理技术。**
>
> 原本在 Unix 系统中，当一个主进程通过 fork()系统调用创建子进程后，内核进程会复制主进程的整个内存空间中的数据，然后分配给子进程。这种方式存在的问题有以下几点：
>
> - 这个过程非常耗时
> - 这个过程降低了系统性能
> - 如果主进程修改了其内存数据，子进程副本中的数据是没有修改的。即出现了数据冗余，而冗余数据最大的问题是数据一致性无法保证。而写时复制则是在任何一方需要写入数据到共享内存时都会出现异常，此时内核进程就会将需要写入的数据 copy 出一个副本写入到另外一块非共享内存区域

<br>

## AOF 持久化

### 概述

- AOF，Append Only File，是指 Redis 将每一次的写操作都以日志的形式记录到一个 AOF文件中的持久化技术。当需要恢复内存数据时，将这些写操作重新执行一次，便会恢复到之前的内存数据状态。

<br>

### AOF 基础配置

#### AOF 的开启

- 默认情况下 AOF 持久化是没有开启的，通过修改配置文件中的 appendonly 属性为 yes 可以开启。

<br>

#### 文件名配置

- Redis 7 在这里发生了重大变化。原来只有一个 appendonly.aof 文件，现在具有了三类多个文件：
  - 基本文件：可以是 RDF 格式也可以是 AOF 格式。其存放的内容是由 RDB 转为 AOF 当时内存的快照数据。该文件可以有多个。
  - 增量文件：以操作日志形式记录转为 AOF 后的写入操作。该文件可以有多个。
  - 清单文件：用于维护 AOF 文件的创建顺序，保障激活时的应用顺序。该文件只有一个。

<br>

#### 混合式持久化开启

- 对于基本文件可以是 RDF 格式也可以是 AOF 格式。通过 aof-use-rdb-preamble 属性可以选择。其默认值为 yes，即默认 AOF 持久化的基本文件为 rdb 格式文件，也就是默认采用混合式持久化。

<br>

#### AOF 文件目录配置

- 为了方便管理，可以专门为 AOF 持久化文件指定存放目录。目录名由 appenddirname属性指定，存放在 redis.conf 配置文件的 dir 属性指定的目录，默认为 Redis 安装目录。

<br>

### AOF 文件格式

- AOF 文件包含三类文件：基本文件、增量文件与清单文件。其中基本文件一般为 rdb 格式，在前面已经研究过了。

<br>

#### Redis 协议

- 增量文件扩展名为.aof，采用 AOF 格式。AOF 格式其实就是 Redis 通讯协议格式，AOF持久化文件的本质就是基于 Redis 通讯协议的文本，将命令以纯文本的方式写入到文件中。
- Redis 协议规定，Redis 文本是以行来划分，每行以\r\n 行结束。每一行都有一个消息头，以表示消息类型。消息头由六种不同的符号表示，其意义如下：
  - (+) 表示一个正确的状态信息
  - (-) 表示一个错误信息
  - (*) 表示消息体总共有多少行，不包括当前行
  - ($) 表示下一行消息数据的长度，不包括换行符长度\r\n
  - (空) 表示一个消息数据
  - (:) 表示返回一个数值

<br>

#### 查看 AOF 文件

打开 appendonly.aof.1.incr.aof 文件，可以看到如下格式内容。

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.5w0csfakr4k0.webp" width="40%">

<br>

- 以上内容中框起来的是三条命令。一条数据库切换命令 SELECT 0，两条 set 命令。它们的意义如下：

> *2 -- 表示当前命令包含 2 个参数
> $6 -- 表示第 1 个参数包含 6 个字符
> SELECT -- 第 1 个参数
> $1 -- 表示第 2 个参数包含 1 个字符
> 0 -- 第 2 个参数
> *3 --表示当前命令包含 3 个参数
> $3 -- 表示第 1 个参数包含 3 个字符
> set -- 第 1 个参数
> $3 -- 表示第 2 个参数包含 3 个字符
> k11 -- 第 2 个参数
> $3 -- 表示第 3 个参数包含 2 个字符
> v11 -- 第 3 个参数
> *3

<br>

#### 清单文件

- 打开清单文件 appendonly.aof.manifest，查看其内容如下：

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.2uuk3oh389k0.webp" width="60%">

<br>

- 该文件首先会按照 seq 序号列举出所有基本文件，基本文件 type 类型为 b，然后再按照seq 序号再列举出所有增量文件，增量文件 type 类型为 i。

- 对于 Redis 启动时的数据恢复，也会按照该文件由上到下依次加载它们中的数据。

<br>

### Rewrite 机制

- 随着使用时间的推移，AOF 文件会越来越大。为了防止 AOF 文件由于太大而占用大量的磁盘空间，降低性能，Redis 引入了 Rewrite 机制来对 AOF 文件进行压缩。

<br>

#### 何为 rewrite

- 所谓 Rewrite 其实就是对 AOF 文件进行重写整理。当 Rewrite 开启后，主进程 redis-server创建出一个子进程 bgrewriteaof，由该子进程完成 rewrite 过程。其首先对现有 aof 文件进行rewrite 计算，将计算结果写入到一个临时文件，写入完毕后，再 rename 该临时文件为原 aof文件名，覆盖原有文件。

<br>

#### rewrite 计算

- rewrite 计算也称为 rewrite 策略。rewrite 计算遵循以下策略：
  - 读操作命令不写入文件
  - 无效命令不写入文件
  - 过期数据不写入文件
  - 多条命令合并写入文件

<br>

#### 手动开启 rewrite

- Rewrite 过程的执行有两种方式。一种是通过 bgrewriteaof 命令手动开启，一种是通过设置条件自动开启。

以下是手动开启方式：

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.5hbstfr9bgw0.webp" width="70%">

<br>

- 该命令会使主进程 redis-server 创建出一个子进程 bgrewriteaof，由该子进程完成 rewrite过程。而在 rewrite 期间，redis-server 仍是可以对外提供读写服务的。

<br>

#### 自动开启 rewrite

- 手动方式需要人办干预，所以一般采用自动方式。由于 Rewrite 过程是一个计算过程，需要消耗大量系统资源，会降低系统性能。所以，Rewrite 过程并不是随时随地任意开启的，而是通过设置一些条件，当满足条件后才会启动，以降低对性能的影响。
- **auto-aof-rewrite-percentage**：开启 rewrite 的增大比例，默认 100%。指定为 0，表示禁用自动 rewrite。
- **auto-aof-rewrite-min-size**：开启 rewrite 的 AOF 文件最小值，默认 64M。该值的设置主要是为了防止小 AOF 文件被 rewrite，从而导致性能下降。

> 自动重写 AOF 文件。当 AOF 日志文件大小增长到指定的百分比时，Redis 主进程redis-server 会 fork 出一个子进程 bgrewriteaof 来完成 rewrite 过程。
>
> 其工作原理如下：Redis 会记住最新 rewrite 后的 AOF 文件大小作为基本大小，如果从主机启动后就没有发生过重写，则基本大小就使用启动时 AOF 的大小。
>
> 如果当前 AOF 文件大于基本大小的配置文件中指定的百分比阈值，且当前 AOF 文件大
> 于配置文件中指定的最小阈值，则会触发 rewrite。

<br>

## AOF 优化配置

### appendfsync

- 当客户端提交写操作命令后，该命令就会写入到 aof_buf 中，而 aof_buf 中的数据持久化到磁盘 AOF 文件的过程称为数据同步。
- 何时将 aof_buf 中的数据同步到 AOF 文件？采用不同的数据同步策略，同时的时机是不同的，有三种策略：
  - always：写操作命令写入 aof_buf 后会立即调用 fsync()系统函数，将其追加到 AOF 文件。该策略效率较低，但相对比较安全，不会丢失太多数据。最多就是刚刚执行过的写操作在尚未同步时出现宕机或重启，将这一操作丢失。
  - no：写操作命令写入 aof_buf 后什么也不做，不会调用 fsync()函数。而将 aof_buf 中的数据同步磁盘的操作由操作系统负责。Linux 系统默认同步周期为 30 秒。效率较高。
  - everysec：默认策略。写操作命令写入 aof_buf 后并不直接调用 fsync()，而是每秒调用一次 fsync()系统函数来完成同步。该策略兼顾到了性能与安全，是一种折中方案。

<br>

### no-appendfsync-on-rewrite

- 该属性用于指定，当 AOF fsync 策略设置为 always 或 everysec，当主进程创建了子进程正在执行 bgsave 或 bgrewriteaof 时，主进程是否不调用 fsync()来做数据同步。设置为 no，双重否定即肯定，主进程会调用 fsync()做同步。而 yes 则不会调用 fsync()做数据同步
- 如果调用 fsync()，在需要同步的数据量非常大时，会阻塞主进程对外提供服务，即会存在延迟问题。如果不调用 fsync()，则 AOF fsync 策略相当于设置为了 no，可能会存在 30 秒数据丢失的风险。

<br>

### aof-rewrite-incremental-fsync

- 当 bgrewriteaof 在执行过程也是先将 rewrite 计算的结果写入到了 aof_rewrite_buf 缓存中，然后当缓存中数据达到一定量后就会调用 fsync()进行刷盘操作，即数据同步，将数据写入到临时文件。该属性用于控制 fsync()每次刷盘的数据量最大不超过 4MB。这样可以避免由于单次刷盘量过大而引发长时间阻塞

<br>

### aof-load-truncated

- 在进行 AOF 持久化过程中可能会出现系统突然宕机的情况，此时写入到 AOF 文件中的最后一条数据可能会不完整。当主机启动后，Redis 在 AOF 文件不完整的情况下是否可以启动，取决于属性 aof-load-truncated 的设置。其值为：
  - yes：AOF 文件最后不完整的数据直接从 AOF 文件中截断删除，不影响 Redis 的启动。
  - no：AOF 文件最后不完整的数据不可以被截断删除，Redis 无法启动。

<br>

### aof-timestamp-enabeld

- 该属性设置为 yes 则会开启在 AOF 文件中增加时间戳的显示功能，可方便按照时间对数据进行恢复。但该方式可能会与 AOF 解析器不兼容，所以默认值为 no，不开启。

<br>

## AOF 持久化过程

### AOF 详细的持久化过程如下：

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.6cqgnli756w0.webp" width="90%">

<br>

1)	Redis 接收到的写操作命令并不是直接追加到磁盘的 AOF 文件的，而是将每一条写命令按照 redis 通讯协议格式暂时添加到 AOF 缓冲区 aof_buf。

2)	根据设置的数据同步策略，当同步条件满足时，再将缓冲区中的数据一次性写入磁盘的AOF 文件，以减少磁盘 IO 次数，提高性能。

3)	当磁盘的 AOF 文件大小达到了 rewrite 条件时，redis-server 主进程会 fork 出一个子进程bgrewriteaof，由该子进程完成 rewrite 过程。

4)	子进程 bgrewriteaof 首先对该磁盘 AOF 文件进行 rewrite 计算，将计算结果写入到一个临时文件，全部写入完毕后，再 rename 该临时文件为磁盘文件的原名称，覆盖原文件。

5)	如果在 rewrite 过程中又有写操作命令追加，那么这些数据会暂时写入 aof_rewrite_buf缓冲区。等将全部 rewrite 计算结果写入临时文件后，会先将 aof_rewrite_buf 缓冲区中的数据写入临时文件，然后再 rename 为磁盘文件的原名称，覆盖原文件。

<br>

## RDB 与 AOF 对比

### RDB 优势与不足

#### RDB 优势

- RDB 文件较小
- 数据恢复较快

<br>

#### RDB 不足

- 数据安全性较差
- 写时复制会降低性能
- RDB 文件可读性较差

<br>

### AOF 优势与不足

#### AOF 优势

- 数据安全性高
- AOF 文件可读性强

<br>

#### AOF 不足

- AOF 文件较大
- 写操作会影响性能
- 数据恢复较慢

<br>

## 持久化技术选型

- 官方推荐使用 RDB 与 AOF 混合式持久化。
- 若对数据安全性要求不高，则推荐使用纯 RDB 持久化方式
- 不推荐使用纯 AOF 持久化方式。
- 若 Redis 仅用于缓存，则无需使用任何持久化技术。

<br>

<br>

<br>

# 第6章 Redis 主从集群

## 概述

- 为了避免 Redis 的单点故障问题，我们可以搭建一个 Redis 集群，将数据备份到集群中的其它节点上。若一个 Redis 节点宕机，则由集群中的其它节点顶上

<br>

## 主从集群搭建

- Redis 的主从集群是一个“一主多从”的读写分离集群。集群中的 Master 节点负责处理客户端的读写请求，而 Slave 节点仅能处理客户端的读请求。只所以要将集群搭建为读写分离模式，主要原因是，对于数据库集群，写操作压力一般都较小，压力大多数来自于读操作请求。所以，只有一个节点负责处理写操作请求即可。

<br>

## 伪集群搭建与配置

- 在采用单线程 IO 模型时，为了提高处理器的利用率，一般会在一个主机中安装多台 Redis，构建一个 Redis 主从伪集群。当然，搭建伪集群的另一个场景是，在学习 Redis，而学习用主机内存不足以创建多个虚拟机。
- 下面要搭建的读写分离伪集群包含一个 Master 与两个 Slave。它们的端口号分别是：6380、6381、6382。

<br>

### 复制 redis.conf

- 在 redis 安装目录中 mkdir 一个目录，名称随意。这里命名为 cluster。然后将 redis.conf文件复制到 cluster 目录中。该文件后面会被其它配置文件包含，所以该文件中需要设置每个 Redis 节点相同的公共的属性。

<br>

### 修改 redis.conf

> 在 redis.conf 中做如下几项修改：

- **masterauth**

> 因为我们要搭建主从集群，且每个主机都有可能会是 Master，所以最好不要设置密码验证属性 requirepass。如果真需要设置，一定要每个主机的密码都设置为相同的。此时每个配置文件中都要设置两个完全相同的属性：requirepass 与 masterauth。其中 requirepass 用于指定当前主机的访问密码，而 masterauth 用于指定当前 slave 访问 master 时向 master 提交的访问密码，用于让 master 验证自己身份是否合法。

- **repl-disable-tcp-nodelay**

> 该属性用于设置是否禁用 TCP 特性 tcp-nodelay。设置为 yes 则禁用 tcp-nodelay，此时master 与 slave 间的通信会产生延迟，但使用的 TCP 包数量会较少，占用的网络带宽会较小。相反，如果设置为 no，则网络延迟会变小，但使用的 TCP 包数量会较多，相应占用的网络带宽会大。

> tcp-nodelay：为了充分复用网络带宽，TCP 总是希望发送尽可能大的数据块。为了达到该目的，TCP 中使用了一个名为 Nagle 的算法
>
> Nagle 算法的工作原理是，网络在接收到要发送的数据后，并不直接发送，而是等待着数据量足够大（由 TCP 网络特性决定）时再一次性发送出去。这样，网络上传输的有效数据比例就得到了大大提升，无效数据传递量极大减少，于是就节省了网络带宽，缓解了网络压力。
>
> tcp-nodelay 则是 TCP 协议中 Nagle 算法的开头。

<br>

### 新建 redis6380.conf

- 新建一个 redis 配置文件 redis6380.conf，该配置文件中的 Redis 端口号为 6380。

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.7gbc2hre5u40.webp" width="60%">

<br>

### 再复制出两个 conf 文件

再使用 redis6380.conf 复制出两个 conf 文件：redis6381.conf 与 redis6382.conf。然后修改其中的内容。

- 修改 redis6381.conf 的内容如下：

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.xffys58raao.webp" width="60%">

<br>

- 修改 redis6382.conf 的内容如下：

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.5e6keop3ns00.webp" width="60%">

<br>

### 启动三台 Redis

- 分别使用 redis6380.conf、redis6381.conf 与 redis6382.conf 三个配置文件启动三台 Redis

<br>

### 设置主从关系

- 再打开三个会话框，分别使用客户端连接三台 Redis。然后通过 slaveof 命令，指定 6380的 Redis 为 Master。

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.4bf4uzhhovq0.webp" width="60%">

<br>

### 查看状态信息

- 通过 info replication 命令可查看当前连接的 Redis 的状态信息

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.5uupdo5hec00.webp" width="60%">

<br>

## 分级管理

- 若 Redis 主从集群中的 Slave 较多时，它们的数据同步过程会对 Master 形成较大的性能压力。此时可以对这些 Slave 进行分级管理。

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.2mzrpmmhd3q0.webp" width="70%">

<br>

- 设置方式很简单，只需要让低级别 Slave 指定其 slaveof 的主机为其上一级 Slave 即可。不过，上一级 Slave 的状态仍为 Slave，只不过，其是更上一级的 Slave。
- 例如，指定 6382 主机为 6381 主机的 Slave，而 6381 主机仍为真正的 Master 的 Slave。
- 此时会发现，Master 的 Slave 只有 6381 一个主机

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.jtz8pen64f4.webp" width="70%">

<br>

## 容灾冷处理

- 在 Master/Slave 的 Redis 集群中，若 Master 出现宕机怎么办呢？有两种处理方式，一种是通过手工角色调整，使 Slave 晋升为 Master 的冷处理；一种是使用哨兵模式，实现 Redis集群的高可用 HA，即热处理。

- 无论 Master 是否宕机，Slave 都可通过 slaveof no one 将自己由 Slave 晋升为 Master。如果其原本就有下一级的 Slave，那么，其就直接变为了这些 Slave 的真正的 Master 了。而原来的 Master 也会失去这个原来的 Slave。

<img src="" width="60%">

<br>

## 主从复制原理

### 主从复制过程

- 当一个 Redis 节点（slave 节点）接收到类似 slaveof 127.0.0.1 6380 的指令后直至其可以从 master 持续复制数据，大体经历了如下几个过程：

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/dd571f3f00e4d6950ede0c9190c73ef.5uuiaga2r2g0.webp" width="80%">

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/731d169b0deeb64901d4cc82ec49558.1c8izssggrz4.webp" width="80%">

<br>

















<br>

<br>

<br>

#  第7章 Redis 分布式系统











































<br>

<br>

<br>

# 学习备注

> 1. **scan** 还需要深入理解一下

```html
&emsp;&emsp;
<font color=red></font>
```

<br>

<br>

<br>

<img src="" width="60%">

<br>

