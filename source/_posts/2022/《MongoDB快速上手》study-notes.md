---
title: 《MongoDB快速上手》study notes
author: ratears
date: 2022-12-04 06:14:34
updated: 2022-12-04 06:14:34
categories:
	- [Database,NoSQL,MongoDB]
tags:
	- MongoDB
	- NoSQL
	- Database
---



# 目标

- 理解MongoDB的业务场景、熟悉MongoDB的简介、特点和体系结构、数据类型等
- 能够在Windows和Linux下安装和启动MongoDB、图形化管理界面Compass的安装使用
- 掌握MongoDB基本常用命令实现数据的CRUD
- 掌握MongoDB的索引类型、索引管理、执行计划
- 使用Spring Data MongoDB完成文章评论业务的开发

<br>

<br>

<br>

# MongoDB相关概念

## 业务应用场景

- 传统的关系型数据库（如MySQL），在数据操作的“三高”需求以及应对Web2.0的网站需求面前，显得力不从心。

> - “三高”需求：
>   - High performance - 对数据库高并发读写的需求。
>   - Huge Storage - 对海量数据的高效率存储和访问的需求。
>   -  High Scalability && High Availability- 对数据库的高可扩展性和高可用性的需求

- **而MongoDB可应对“三高”需求**

- 具体的应用场景如：

> 1）社交场景，使用 MongoDB 存储存储用户信息，以及用户发表的朋友圈信息，通过地理位置索引实现附近的人、地点等功能。
>
> 2）游戏场景，使用 MongoDB 存储游戏用户信息，用户的装备、积分等直接以内嵌文档的形式存储，方便查询、高效率存储和访问。
>
> 3）物流场景，使用 MongoDB 存储订单信息，订单状态在运送过程中会不断更新，以 MongoDB 内嵌数组的形式来存储，一次查询就能将订单所有的变更读取出来。
>
> 4）物联网场景，使用 MongoDB 存储所有接入的智能设备信息，以及设备汇报的日志信息，并对这些信息进行多维度的分析。
>
> 5）视频直播，使用 MongoDB 存储用户信息、点赞互动信息等。

- 这些应用场景中，数据操作方面的共同特点是：
  - （1）数据量大
  - （2）写入操作频繁（读写都很频繁）
  - （3）价值较低的数据，对事务性要求不高

> 对于这样的数据，我们更适合使用MongoDB来实现数据的存储。

<br>

## 什么时候选择MongoDB

- 应用不需要事务及复杂 join 支持
- 新应用，需求会变，数据模型无法确定，想快速迭代开发
- 应用需要2000-3000以上的读写QPS（更高也可以）
- 应用需要TB甚至 PB 级别数据存储
- 应用发展迅速，需要能快速水平扩展
- 应用要求存储的数据不丢失
- 应用需要99.999%高可用
- 应用需要大量的地理位置查询、文本查询

> 如果上述有1个符合，可以考虑 MongoDB，2个及以上的符合，选择 MongoDB 绝不会后悔

- **相对MySQL，可以以更低的成本解决问题（包括学习、开发、运维等成本）**

<br>

## MongoDB简介

- MongoDB是一个开源、高性能、无模式的文档型数据库，当初的设计就是用于简化开发和方便扩展，是NoSQL数据库产品中的一种。是最像关系型数据库（MySQL）的非关系型数据库。
- 它支持的数据结构非常松散，是一种类似于 JSON 的 格式叫BSON，所以它既可以存储比较复杂的数据类型，又相当的灵活。
- MongoDB中的记录是一个文档，它是一个由字段和值对（field:value）组成的数据结构。MongoDB文档类似于JSON对象，即一个文档认为就是一个对象。字段的数据类型是字符型，它的值除了使用基本的一些类型外，还可以包括其他文档、普通数组和文档数组。

<br>

##  体系结构

### MySQL和MongoDB对比

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.yanmy0ggv5s.webp" width="60%">

<br>

| SQL术语/概念 | MongoDB术语/概念 |              解释/说明              |
| :----------: | :--------------: | :---------------------------------: |
|   database   |     database     |               数据库                |
|    table     |    collection    |            数据库表/集合            |
|     row      |     document     |           数据记录行/文档           |
|    column    |      field       |             数据字段/域             |
|    index     |      index       |                索引                 |
| table joins  |                  |        表连接,MongoDB不支持         |
|              |     嵌入文档     | MongoDB通过嵌入式文档来替代多表连接 |
| primary key  |   primary key    | 主键,MongoDB自动将_id字段设置为主键 |

<br>

## 数据模型

### 概述

- MongoDB的最小存储单位就是文档(document)对象。文档(document)对象对应于关系型数据库的行。数据在MongoDB中以
  BSON（Binary-JSON）文档的格式存储在磁盘上
- BSON（Binary Serialized Document Format）是一种类json的一种二进制形式的存储格式，简称Binary JSON。BSON和JSON一样，支持内嵌的文档对象和数组对象，但是BSON有JSON没有的一些数据类型，如Date和BinData类型。

- BSON采用了类似于 C 语言结构体的名称、对表示方法，支持内嵌的文档对象和数组对象，具有轻量性、可遍历性、高效性的三个特点，可以有效描述非结构化数据和结构化数据。这种格式的优点是灵活性高，但它的缺点是空间利用率不是很理想。
- Bson中，除了基本的JSON类型：string,integer,boolean,double,null,array和object，mongo还使用了特殊的数据类型。这些类型包括date,object id,binary data,regular expression 和code。每一个驱动都以特定语言的方式实现了这些类型，查看你的驱动的文档来获取详细信息。

<br>

### BSON数据类型参考列表

| 数据类型      | 描述                                                         | 举例                                                 |
| :------------ | ------------------------------------------------------------ | ---------------------------------------------------- |
| 字符串        | UTF-8字符串都可表示为字符串类型的数据                        | {"x" : "foobar"}                                     |
| 对象id        | 对象id是文档的12字节的唯一 ID                                | {"X" :ObjectId() }                                   |
| 布尔值        | 真或者假：true或者false                                      | {"x":true}+                                          |
| 数组          | 值的集合或者列表可以表示成数组                               | {"x" ： ["a", "b", "c"]}                             |
| 32位整数      | 类型不可用。JavaScript仅支持64位浮点数，所以32位整数会被自动转换。 | shell是不支持该类型的，shell中默认会转换成64位浮点数 |
| 64位整数      | 不支持这个类型。shell会使用一个特殊的内嵌文档来显示64位整数  | shell是不支持该类型的，shell中默认会转换成64位浮点数 |
| 64位浮点数    | shell中的数字就是这一种类型                                  | {"x"：3.14159，"y"：3}                               |
| null          | 表示空值或者未定义的对象                                     | {"x":null}                                           |
| undefined     | 文档中也可以使用未定义类型                                   | {"x":undefined}                                      |
| 符号          | shell不支持，shell会将数据库中的符号类型的数据自动转换成字符串 |                                                      |
| 正则表达式    | 文档中可以包含正则表达式，采用JavaScript的正则表达式语法     | {"x" ： /foobar/i}                                   |
| 代码          | 文档中还可以包含JavaScript代码                               | {"x" ： function() { /* …… */ }}                     |
| 二进制数据    | 二进制数据可以由任意字节的串组成，不过shell中无法使用        |                                                      |
| 最大值/最小值 | BSON包括一个特殊类型，表示可能的最大值。shell中没有这个类型。 |                                                      |

> shell默认使用64位浮点型数值。{“x”：3.14}或{“x”：3}。对于整型值，可以使用NumberInt（4字节符号整数）或NumberLong（8字节符号整数），{“x”:NumberInt(“3”)}{“x”:NumberLong(“3”)}

<br>

## MongoDB的特点

- MongoDB主要有如下特点：



- （1）高性能：

> MongoDB提供高性能的数据持久性。特别是,对嵌入式数据模型的支持减少了数据库系统上的I/O活动。
>
> 索引支持更快的查询，并且可以包含来自嵌入式文档和数组的键。（文本索引解决搜索的需求、TTL索引解决历史数据自动过期的需求、地理位置索引可用于构建各种 O2O 应用）
>
> mmapv1、wiredtiger、mongorocks（rocksdb）、in-memory 等多引擎支持满足各种场景需求
>
> Gridfs解决文件存储的需求

- （2）高可用性：

> MongoDB的复制工具称为副本集（replica set），它可提供自动故障转移和数据冗余。

- （3）高扩展性：

> MongoDB提供了水平可扩展性作为其核心功能的一部分。
>
> 分片将数据分布在一组集群的机器上。（海量数据存储，服务能力水平扩展）
>
> 从3.4开始，MongoDB支持基于片键创建数据区域。在一个平衡的集群中，MongoDB将一个区域所覆盖的读写只定向到该区域内的那些片。

- （4）丰富的查询支持：

> MongoDB支持丰富的查询语言，支持读和写操作(CRUD)，比如数据聚合、文本搜索和地理空间查询等。

- （5）其他特点：如无模式（动态模式）、灵活的文档模型

<br>

<br>

<br>

# 单机部署

## Linux系统中的安装启动和连接

### 版本的选择

- MongoDB的版本命名规范如：x.y.z；
  - y为奇数时表示当前版本为开发版，如：1.5.2、4.1.13；
  - y为偶数时表示当前版本为稳定版，如：1.6.3、4.0.10；
  - z是修正版本号，数字越大越好。

<br>

## 下载安装启动

（1）先到官网下载压缩包 mongodb-linux-x86_64-rhel70-4.0.28.tgz 。

（2）上传压缩包到Linux中，解压到 `/usr/local`

（3）创建软连接

```bash
ln -s mongodb-linux-x86_64-rhel70-4.0.28/ mongodb
```

（4）新建几个目录，分别用来存储数据和日志：

```bash
#数据存储目录
mkdir -p /mongodb/single/data/db
#日志存储目录
mkdir -p /mongodb/single/log
```

（5）新建并修改配置文件

```shell
vi /mongodb/single/mongod.conf
```

- 配置文件的内容如下：

```shell
#数据库路径
dbpath=/mongodb/single/data/db
#日志输出文件路径
logpath=/mongodb/single/log/mongod.log
#错误日志采用追加模式
logappend=true
#启用日志文件，默认启用
journal=true
#这个选项可以过滤掉一些无用的日志信息，若需要调试使用请设置为false
quiet=true
#端口号 默认为27017
port=27017
#允许远程访问（你的服务器局域网ip）
bind_ip=192.168.146.136,localhost
#开启子进程
fork=true
#开启认证，必选先添加用户，先不开启（不用验证账号密码）
#auth=true
```

（6）启动MongoDB服务

```bash
[root@localhost local]# /usr/local/mongodb/bin/mongod -f /mongodb/single/mongod.conf
about to fork child process, waiting until server is ready for connections.
forked process: 18809
child process started successfully, parent exiting
```

通过进程来查看服务是否启动了：

```bash
[root@localhost local]# ps -ef |grep mongod
root      18809      1  2 00:40 ?        00:00:00 /usr/local/mongodb/bin/mongod -f /mongodb/single/mongod.conf
root      18844   8794  0 00:41 pts/1    00:00:00 grep --color=auto mongod
```

（7）分别使用mongo命令和compass工具来连接测试。

- 需要配置防火墙放行，或直接关闭linux防火墙

（8）停止关闭服务

- （一）快速关闭方法（快速，简单，数据可能会出错）

```bash
# 目标：通过系统的kill命令直接杀死进程：杀完要检查一下，避免有的没有杀掉。

#通过进程编号关闭节点
kill -2 54410
```

【补充】

如果一旦是因为数据损坏，则需要进行如下操作（了解）：

1）删除lock文件：

```bash
rm -f /mongodb/single/data/db/*.lock
```

2）修复数据：

```bash
/usr/local/mongdb/bin/mongod --repair --dbpath=/mongodb/single/data/db
```



（二）标准的关闭方法（数据不容易出错，但麻烦）：

```bash
# 目标：通过mongo客户端中的shutdownServer命令来关闭服务
# 主要的操作步骤参考如下：

//客户端登录服务，注意，这里通过localhost登录，如果需要远程登录，必须先登录认证才行。
mongo --port 27017
//#切换到admin库
use admin
//关闭服务
db.shutdownServer()
```

<br>

<br>

<br>

# 基本常用命令

## 案例需求

- 存放文章评论的数据存放到MongoDB中，数据结构参考如下：
  - 数据库：articledb

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.4cm5w9rgfkw0.webp" width="80%">

<br>

## 数据库操作

### 选择和创建数据库

```bash
# 选择和创建数据库的语法格式：
use 数据库名称

# 如果数据库不存在则自动创建，例如，以下语句创建 articledb 数据库：
use articledb

# 查看有权限查看的所有的数据库命令
show dbs
# 或
show databases
```

> 注意: 在 MongoDB 中，集合只有在内容插入后才会创建! 就是说，创建集合(数据表)后要再插入一个文档(记录)，集合才会真正创建。

<br>

<br>

<br>

# 查看当前正在使用的数据库命令



























<br>

<br>

<br>

# 学习备注

> 1. 课件中的某些内容已经过时了或有问题，后续再深入了解一下。目前先熟悉使用再说

```html
&emsp;&emsp;
```

<br>

<br>

<br>

<img src="" width="60%">

<br>