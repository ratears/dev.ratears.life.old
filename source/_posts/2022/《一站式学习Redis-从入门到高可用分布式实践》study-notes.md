---
title: 《一站式学习Redis-从入门到高可用分布式实践》study notes
auth: ratears
date: 2022-08-31 02:12:50
update: 2022-08-31 02:12:50
categories:
	- [database,redis]
	- [nosql,redis]
tags:
	- redis
	- nosql
	- database
---



# Redis初识

## Redis是什么

- 开源
- 基于键值的存储服务系统
- 多种数据结构
- 高性能、功能丰富

<br>

## Redis的特性

- 速度快（10w OPS）
  - 数据存储在内存（速度快的主要原因）
  - 使用C语言编写
  - 单线程
- 持久化
  - Redis的所有数据保存在内存中，对数据的更新异步的保存在磁盘上
- 多种数据结构
  - 5中主要类型
- 支持多种编程语言
  - 主流编程语言都支持Redis
- 功能丰富
  - 发布订阅
  - 事物
  - lua脚本
  - pipeline
- 简单
  - 早期代码23000行
  - 不依赖外部库
  - 单线程模型
- 主从复制
- 高可用、分布式

<br>

## Redis典型使用场景

- 缓存系统
- 计数器
- 消息队列系统
- 排行榜
- 社交网络
- 事实系统

<br>

## Redis安装

- 安装前环境准备

```shell
yum -y install gcc gcc-c++ kernel-devel
```

- 下载安装

```shell
[root@localhost ~]# cd /install

[root@localhost install]# wget https://github.com/redis/redis/archive/7.0.4.tar.gz
[root@localhost install]# tar -zxvf redis-7.0.4.tar.gz
[root@localhost install]# mv redis-7.0.4 /usr/local/
[root@localhost install]# cd /usr/local/

[root@localhost local]# ln -s redis-7.0.4/ redis
[root@localhost local]# cd redis

# 直接 make 会失败报错 原因：建立redis时系统默认使用jemalloc作为内存管理工具，但是当前无可用jemalloc，切换为标准内存管理工具libc问题解决
[root@localhost redis]# make MALLOC=libc
[root@localhost redis]# make install
```





<br>

## Redis三种启动方式

- 最简启动

```shell
# 使用默认配置启动
redis-server
```

- 动态参数启动

```shell
redis-server --port 6380
```

- 配置文件启动（推荐）

```shell
redis-server configPath
```

- 启动方式比较

> 生产环境选择配置启动
>
> 单机多实例配置文件可以用端口区分开来

<br>

## 验证

```shell
ps -ef |grep redis

netstat -antpl |grep redis

redis-cli -h [ip] -p [port] ping 
```

<br>

## Redis可执行文件说明

- redis-server	-	redis服务器
- redis-cli	-	redis命令行客户端
- redis-benchmark	-	redis性能测试工具
- redis-check-aof	-	AOF文件修复工具
- redis-check-dump	-	RDB文件检查工具
- redis-sentinel	-	Sentinel服务器（2.8之后）

<br>

## Redis常用配置

```shell
# 是否以守护进程方式启动 [yes|no]
daemonize yes

# redis 对外端口
port 6380

# 配置日志名称
logfile "6380.log"

# redis 工作目录（包括日志文件、持久化文件存储位置）
dir "/usr/local/redis/data/"
```

<br>

# API的理解和使用

## 通用命令

```shell
# 遍历所有的key，可以使用通配符
# 复杂度 O(n) ，不建议在生产环境使用，除非数量特别小
keys *

# 内置的对键值 统计的计数器
dbsize

# 检查key是否存在，返回 1 或 0
exists

# 删除指定 key-value，返回 1 或 0
del key [key...]

# 设置 key 在 seconds 秒后过期
expire key seconds

# 查询key 还有多长时间过期，不过期则返回 -1
ttl key

# 去除过期时间，
persist key

# 不存在则返回 none
type key
```

<br>

## 数据结构和内部编码

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.3scu9pvrtk80.webp" width="75%" />

<br>

- redisObject

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.69a05a18h8s0.webp" width="75%"/>

<br>

## 单线程

### 单线程为什么这么快

- 纯内存（主要原因）
- 阻塞IO
- 避免线程切换和竞态消耗

<br>

### 注意事项

1. 一次只运行一条命令

2. 拒绝长（慢）命命令：`kesy` `flushall` `flushdb` `slow lua script` `mutil/exec` `operate big value(collection)`
3. 其实不是单线程：`fysnc file descriptor` `close file descriptor`

<br>

## String（字符串）

### 结构

- 可以是真的字符串，同时也可以是数字，二进制数字等等。大小限制 512MB

<img src ="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/4c36c4ac53ae6df147284b21e0fa921.lo8tp31mfhs.webp" width="70%" />

<br>

### 场景

- 缓存
- 计数器
- 分布式锁
- ......

<br>

### 命令

|    命令     |          举例          | 时间复杂度 |                             说明                             |
| :---------: | :--------------------: | :--------: | :----------------------------------------------------------: |
|     set     |     set hello word     |    O(1)    |             不管key是否存在，都设置。成功返回ok              |
|    setnx    |       setnx k v        |    O(1)    |                       key不存在才设置                        |
|   set xx    |       set k v xx       |    O(1)    |                 key存在才设置。不存在返回nil                 |
|    mset     |    mset k1 v1 k2 v2    |    O(n)    |                 批量设置key-value，原子操作                  |
|     del     |       del hello        |    O(1)    |                     成功返回1,失败返回0                      |
|     get     |       get hello        |    O(1)    |                 成功返回的value，失败返回nil                 |
|    mget     |       mget k1 k2       |    O(n)    |                 批量获取key-value，原子操作                  |
|    incr     |      incr counter      |    O(1)    | 自增1，并返回自增后的value值。如果key不存在，自增后get(key) = 1 |
|    decr     |      decr counter      |    O(1)    | 自减1，并返回自减后的value值。如果key不存在，自减后get(key) = -1 |
|   incrby    |     incrby view k      |    O(1)    | 自增k，并返回自增k后的value值。如果key不存在，自增后get(key) = k |
|   decrby    |     decrby view k      |    O(1)    | 自减k，并返回自减后的value值。如果key不存在，自减后get(key) = -k |
|   getset    |   getset k newvalue    |    O(1)    |              set key newValue，并返回旧的value               |
|    apend    |       apend k v        |    O(1)    |                    将value追加到旧的value                    |
|   strlen    |        strlen k        |    O(1)    |        返回字符串的长度[字节]（utf-8 中文占 2个字节）        |
| incrbyfloat |    incrbyfloat k v     |    O(1)    |                       增加指定的浮点数                       |
|  getrange   |  getrange k start end  |    O(1)    |                  获取字符串指定下标所有的值                  |
|  setrange   | setrange k index value |    O(1)    |                     设置指定下标对应的值                     |

<br>

### 实战

1. 记录网站每个用户的个人主页访问量

```shell
# 单线程：无竞争（并发不会出现计错数的情况）
incr userId:pageView
```



2. 缓存视频的基本信息（数据源在MySQL中）

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.2nspuoc7ly60.webp" width="70%" />

<br>

3. 分布式id生成器

<br>

### 查漏补缺

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/1d121aa9afdb06c7451e0fb8bb2d560.kias448eqho.webp" width="70%" />

<br>

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/4c1ea9bb24f88ec70a3aacc637a9183.4xykrj7thuw0.webp" width="70%" />







## hash（哈希）





<br>

# Redis客户端

## Java客户端Jedis

### Jedis简单使用

- maven依赖

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
    <type>jar</type>
    <scope>compile</scope>
</dependency>
```

- Jedis直连

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.1l6p9hnls4rk.webp" width="70%"/>

<br>

### JedisPool简单使用

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.4ehexcty0j20.webp" width = "70%"/>

<br>

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.3r9cudbxdi00.webp" width = "70%"/>

<br>

### Jedis 与 JedisPool比较

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/97b459332ea53d7e42b90bd2743dcda.396pfset03e0.webp" width="60%" />

<br>

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/7e5d97f719abbfe9b363126e682f86d.3g21qh3ltu20.webp" width="60%" />

<br>

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/41e863e25d7ae1dc0c3cebe8444b9c5.2tj6g97vyck.webp" width="60%" />

<br>





### Jedis配置优化

#### pool配置 - 资源数控制

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.21oqgp1jyxnk.webp" width="60%" />

<br>

#### pool配置 - 借还参数

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.7cwnd55qd1s0.webp" width="70%" />

<br>

#### 适合的 maxTotal

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.2mma31ou5nw0.webp" width="60%" />

<br>

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.6sejm8evfwc.webp" width="60%" />

<br>

#### 适合的maxIdle和minIdle

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.6lhcm72l9bo0.webp" width="60%" />

<br>

### 常见问题和解决思路

- 常见问题

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.75d32k2l47g0.webp" width="60%" />

<br>

- 解决思路

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.41af9urqt6a0.webp" width="60%" />

<br>

- 错误示例

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.31pwe9jqbig0.webp" width="60%" />

<br>

- 推荐写法

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.69xc8p0s24o0.webp" width="60%" />

<br>

# Redis其他功能

## slowlog（慢查询）

### 命令生命周期

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.76jbqyv85l80.webp" width="70%" />

<br>

- **（1）慢查询发生在第3阶段**
- **（2）客户端超时不一定是慢查询，但慢查询是客户端超时的一个可能因素**

<br>

### 配置

#### slowlog-max-len

0. `slowlog-max-len` 表示慢查询队列长度

1. 慢查询是一个先进先出的队列；如果在第3步执行过程中，被列入慢查询的范围内，就会进入一个队列（用redis的列表实现的）
2. 慢查询队列是固定长度的
3. 慢查询队列数据保存在内存中

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.44yrctjcttc0.webp" width="70%" />

<br>

#### slowlog-log-slower-than

1. `slowlog-log-slower-than` 表示慢查询命令执行时间阈值（单位：微秒，1ms=1000微秒），超过阈值会被加入慢查询队列中
2. `slowlog-log-slower-than = 0` ，记录所有命令
3. `slowlog-log-slower-than < 0` ，不记录任何命令

<br>

#### 默认值

```shell
127.0.0.1:6380> config get slowlog-max-len
1) "slowlog-max-len"
2) "128"

# 10000微秒 =》 10ms
127.0.0.1:6380> config get slowlog-log-slower-than
1) "slowlog-log-slower-than"
2) "10000"
```

<br>

#### 配置方法

1. 方法一：修改配置文件重启（一般在第一次启动redis前进行配置。但如果redis正在运行中，不推荐此方式）

2. 方法二：动态配置

```shell
config set slowlog-max-len 1000

config set slowlog-log-slower-than 1000
```

```shell
# 操作示例
127.0.0.1:6380> config set slowlog-max-len 1000
OK
127.0.0.1:6380> config set slowlog-log-slower-than 1000
OK
127.0.0.1:6380> config get slowlog-max-len
1) "slowlog-max-len"
2) "1000"
127.0.0.1:6380> config get slowlog-log-slower-than
1) "slowlog-log-slower-than"
2) "1000"
127.0.0.1:6380>
```

<br>

### 慢查询命令

|   慢查询命令    |          说明          |
| :-------------: | :--------------------: |
| slowlog get [n] | 获取慢查询队列指定条数 |
|   slowlog len   |   获取慢查询队列长度   |
|  slowlog reset  |     清空慢查询队列     |

<br>

### 运维经验

1. slowlog-max-len不要设置过大。默认10ms，通常设置1ms（实际情况要根据QPS来决定阈值大小，有可能1ms就已经对我们的QPS产生影响了）
2. slowlog-log-slower-than不要设置过小，通常设置1000左右
3. 理解命令生命周期，理解慢查询处于命令生命周期的位置。便于我们排错和优化（慢查询、阻塞、网络都可能成为客户端超时的原因）
4. **定期持久化查询（因为慢查询是存在内存中的，且当慢查询数量逐步增多，早前的慢查询就会丢掉。做好持久化，可以分析历史的慢查询问题）。可以通过其它手段或开源软件实现这个功能**

<br>

## pipeline（流水线）

### 网络命令通信模型

#### 1次网络命令通信模型

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.10peiok08sk0.webp" width="60%" />

<br>

#### 批量网络命令通信模型

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.yo8sxod81o0.webp" width="60%" />

<br>

### 什么是pipeline（流水线）

- 我们知道redis的命令执行是很快的，但是网络时间却不一定。使用pipeline可以帮我们节约大量网络时间

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.v0w62a1vlc0.webp" width="60%" />

<br>

### pipeline的作用

| 命令 |   N个命令操作   | 1次pipeline（N个命令） |
| :--: | :-------------: | :--------------------: |
| 时间 | n次网络+n次命令 |  1次网络时间+n次命令   |
| 数量 |     1条命令     |        n条命令         |

- 注意

1. Redis的命令时间是微秒级别
2. pipeline每次条数要控制（网络）



- 举例

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.2sv2n6wunz20.webp" width="60%" />

<br>

### pipeline的jedis实现

- 添加maven依赖

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
    <type>jar</type>
    <scope>compile</scope>
</dependency>
```



- 没有pipe-line

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.2cxbniglxcys.webp" width="70%" />

<br>

- 使用pipeline

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.41hpky36m4a0.webp" width="60%" />

<br>

### pipeline与mget/mset操作的对比

- 原生M操作

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.6m3lpwqzxqs0.webp" width="60%" />

<br>

- pipeline

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.7b0lz2zro400.webp" width="60%" />

<br>

**pipeline命令可拆分**

<br>

### pipeline使用建议

- **注意每次pipeline携带数量**
- **pipeline每次只能作用在一个Redis节点上**
- **注意pipeline与M操作的区别**



## 发布订阅

### 角色

- 发布者（publisher）
- 订阅者（subscriber）
- 频道（channel）

<br>

### 发布订阅模型

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.5u5skel4tjk0.webp" width="60%" />

<br>

- **新的订阅者订阅了一个频道，是无法收到之前的消息**（因为无法做消息堆积，因为redis不是一个真正的消息队列这样一个工具）

<br>

### 发布订阅API

#### publish

```shell
# 向频道发布消息
PUBLISH [channel_name] [message]

127.0.0.1:6380> publish sohu:tv "hello world"
(integer) 0
127.0.0.1:6380> publish sohu:auto "taxi"
(integer) 0
```

<br>

#### subscribe

```shell
# 订阅一个或多个频道
SUBSCRIBE [channel_name]...

127.0.0.1:6380> subscribe sohu:tv
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "sohu:tv"
3) (integer) 1
```

<br>

#### unsubscribe

```shell
# 订阅一个或多个频道
UNSUBSCRIBE [channel_name]...

127.0.0.1:6380> UNSUBSCRIBE sohu:tv
1) "unsubscribe"
2) "sohu:tv"
3) (integer) 0
```

<br>

#### 其它API

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.j7q4w7w0f5c.webp"  width="65%"/>



<br>

#### 发布订阅与消息队列

- Redis可以实现消息队列，消息队列是抢的模式
- 注意二者的区别与使用场景

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.33zfet508q00.webp" width="65%" />

<br>

## BItmap（位图）





## HyperLogLog

- Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的（基于HyperLogLog算法：极小空间完成独立数量统计）
- 本质还是字符串

<br>

### API（命令）

```shell
# Pfadd 命令将所有元素参数添加到 HyperLogLog 数据结构中
redis 127.0.0.1:6379> PFADD key element [element ...]

redis 127.0.0.1:6379> PFADD mykey a b c d e f g h i j
(integer) 1
redis 127.0.0.1:6379> PFCOUNT mykey
(integer) 10
```

```shell
# Pfcount 命令返回给定 HyperLogLog 的基数估算值
redis 127.0.0.1:6379> PFCOUNT key [key ...]

redis 127.0.0.1:6379> PFADD hll foo bar zap
(integer) 1
redis 127.0.0.1:6379> PFADD hll zap zap zap
(integer) 0
redis 127.0.0.1:6379> PFADD hll foo bar
(integer) 0
redis 127.0.0.1:6379> PFCOUNT hll
(integer) 3
redis 127.0.0.1:6379> PFADD some-other-hll 1 2 3
(integer) 1
redis 127.0.0.1:6379> PFCOUNT hll some-other-hll
(integer) 6
redis> 
```

```shell
#  PFMERGE 命令将多个 HyperLogLog 合并为一个 HyperLogLog ，合并后的 HyperLogLog 的基数估算值是通过对所有 给定 HyperLogLog 进行并集计算得出的
PFMERGE destkey sourcekey [sourcekey ...]

redis> PFADD hll1 foo bar zap a
(integer) 1
redis> PFADD hll2 a b c foo
(integer) 1
redis> PFMERGE hll3 hll1 hll2
"OK"
redis> PFCOUNT hll3
(integer) 6
redis>  
```

<br>

### 示例（百万独立用户-内存消耗）

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.540yx2nmmgw0.webp" width="60%" />

<br>

### 使用经验

- 是否能容忍错误？（错误率：0.81%）
- 是否需要单条数据？

<br>

# GEO

- Redis GEO 主要用于存储地理位置信息，并对存储的信息进行操作（存储经纬度，计算两地距离，范围计算等）
- 底层使用 zset 实现

<br>

### 应用场景

- 类似微信摇一摇（计算指定范围类的用户）
- 根据距离计算周围的酒店餐馆等

<br>

### API

- geoadd：添加地理位置的坐标。
- geopos：获取地理位置的坐标。
- geodist：计算两个位置之间的距离。
- georadius：根据用户给定的经纬度坐标来获取指定范围内的地理位置集合。
- georadiusbymember：根据储存在位置集合里面的某个地点获取指定范围内的地理位置集合。
- geohash：返回一个或多个位置对象的 geohash 值。

<br>

# Redis持久化的取舍和选择

## 持久化的作用

1. 什么是持久化

> redis所有的数据保存在内存中，对数据的更新将异步的保存在磁盘上

内存 =》（持久化）=》磁盘

内存 《=（恢复）《= 磁盘



2. 持久化的方式

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.urjf6b58j5c.webp" width="60%" />

<br>

## RDB（Redis DataBase）

### 什么是RDB

- RDB：在不同的时间点，将 redis 存储的数据生成快照并存储到磁盘等介质上

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.xtthna1x3gg.webp" width="60%" />

<br>

### RDB触发机制的三种方式

#### save（同步）

- 可能会造成阻塞

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.1mmseu4fo9og.webp" width="60%" />

<br>

```shell
127.0.0.1:6380>  save
OK
```

- 文件策略：生成临时的rdb文件，当save执行完后。如果存在老的rdb文件，临时文件变成新文件替换老文件
- 复杂度：O(n)

<br>

#### bgsave（异步）

- 客户端执行 `bgsave` redis会使用linux的 `fork()` 函数生成一个redis的子进程，由该子进程生成RDB文件
- 一般情况下， `bgsave` 不会阻塞到redis

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.1nkd5mhwgzcw.webp" width="60%" />

<br>

```shell
127.0.0.1:6380> bgsave
Background saving started
```

- 文件策略：生成临时的rdb文件，当save执行完后。如果存在老的rdb文件，临时文件变成新文件替换老文件
- 复杂度：O(n)

<br>

#### save 与 bgsave比较

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.7z4y2ta7f2w.webp" width="65%" />

<br>

#### 自动

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.42jf7bw3rfe0.webp" width="70%" />

<br>

- 说明：在 60s 中改变了10000 条数据（set，del），会自动做rdb的生成

<br>

##### 缺点

- 数据写入量无法控制，生成规则无法控制。如果文件非常大，或很频繁的做这样的操作，会对硬盘造成一定压力

<br>

##### 默认配置

```shell
################################ 快照  #################################  
#  
# Save the DB on disk:保存数据库到磁盘  
#  
#   save <秒> <更新>  
#  
#   如果指定的秒数和数据库写操作次数都满足了就将数据库保存。  
#  
#   下面是保存操作的实例：  
#   900秒（15分钟）内至少1个key值改变（则进行数据库保存--持久化）  
#   300秒（5分钟）内至少10个key值改变（则进行数据库保存--持久化）  
#   60秒（1分钟）内至少10000个key值改变（则进行数据库保存--持久化）  
#  
#   注释：注释掉“save”这一行配置项就可以让保存数据库功能失效。  
#  
#   你也可以通过增加一个只有一个空字符串的配置项（如下面的实例）来去掉前面的“save”配置。  
#  
#   save ""  
  
save 900 1  
save 300 10  
save 60 10000  
  
#在默认情况下，如果RDB快照持久化操作被激活（至少一个条件被激活）并且持久化操作失败，Redis则会停止接受更新操作。  
#这样会让用户了解到数据没有被正确的存储到磁盘上。否则没人会注意到这个问题，可能会造成灾难。  
#  
#如果后台存储（持久化）操作进程再次工作，Redis会自动允许更新操作。  
#  
#然而，如果你已经恰当的配置了对Redis服务器的监视和备份，你也许想关掉这项功能。  
#如此一来即使后台保存操作出错,redis也仍然可以继续像平常一样工作。  
stop-writes-on-bgsave-error yes  
  
#是否在导出.rdb数据库文件的时候采用LZF压缩字符串和对象？  
#默认情况下总是设置成‘yes’， 他看起来是一把双刃剑。  
#如果你想在存储的子进程中节省一些CPU就设置成'no'，  
#但是这样如果你的kye/value是可压缩的，你的到处数据接就会很大。  
rdbcompression yes  
  
#从版本RDB版本5开始，一个CRC64的校验就被放在了文件末尾。  
#这会让格式更加耐攻击，但是当存储或者加载rbd文件的时候会有一个10%左右的性能下降，  
#所以，为了达到性能的最大化，你可以关掉这个配置项。  
#  
#没有校验的RDB文件会有一个0校验位，来告诉加载代码跳过校验检查。  
rdbchecksum yes  
  
# 导出数据库的文件名称  
dbfilename dump.rdb  
  
# 工作目录  
#  
# 导出的数据库会被写入这个目录，文件名就是上面'dbfilename'配置项指定的文件名。  
#   
# 只增的文件也会在这个目录创建（这句话没看明白）  
#   
# 注意你一定要在这个配置一个工作目录，而不是文件名称。  
dir ./  
```

<br>

##### 最佳配置

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.23rdcl2poqrk.webp" width="70%" />

<br>

- 关闭自动配置

### 触发机制 - 不容忽略方式

1. 全量复制（主从复制时候，主会自动生成RDB）
2. debug reload（相当于不会将内存清空的重启，也会生成RDB）
3. shutdown

<br>

### RDB现存问题

- 耗时耗性能

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.b2ax18993q0.webp" width="50%" />

<br>

- 不可控，丢失数据

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.2hq85j7f4ni0.webp" width="50%" />

<br>

## AOF（Append Only File）

- 将 redis 执行过的所有写指令记录下来（它的写入是实时的），在下次 redis 重新启动时，只要把这些写指令从前到后再重复执行一遍，就可以实现数据恢复了

### AOF运行原理

- 创建

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.1i410l2830jk.webp" width="50%" />

<br>

- 恢复

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.2ode8n5q6b80.webp" width="60%" />

<br>

### AOF的三种策略

#### always

- 写入数据不会丢失

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.1nz1bf5i1ojk.webp" width="50%" />

<br>

#### everysec

- 是redis的配置默认值
- 可能会丢失1s的数据

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.5uvzghtr78s0.webp" width="50%" />

<br>

#### no

- 根据操作系统决定

<br>

### AOF的三种策略对比

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.7ezkvv0o0bg0.webp" width="50%" />

<br>

### AOF重写

- 减少磁盘占用量
- 加速恢复速度

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.3hr6crymmo0.webp" width="50%" />

<br>

#### AOF重写的两种方式

- BGREWRITEAOF （类似rdb的bgsave）
  - 将Redis中的数据进行回溯， 回溯成AOF文件

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.5j30ijjpor40.webp" width="50%" />

<br>

- AOF重写配置

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.72n439aom900.webp" width="50%" />

<br>

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.5wl4pwfy7ds0.webp" width="50%" />

<br>

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.yskrqzxtakw.webp" width="50%" />

<br>

#### AOF重写流程

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.2b1z2py9ols0.webp" width="50%" />

<br>

## RDB与AOF的抉择

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.2got5zb1qs20.webp" width="50%" />

<br>

### RDB最佳策略

- RDB

1. ”关闭“
2. 集中管理
3. 主从，从开

### AOF最佳策略

1. ”开“：缓存和存储
2. AOF集中管理
3. everysec

### 最佳策略

1. 小分片
2. 缓存或存储
3. 监控（硬盘、内存、负载、网络）
4. 足够的内存

<br>

# 常见的持久化开发运维问题

## fork操作

- 

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.3ofn5wyto2w0.webp" width="50%" />

<br>

- fork改善

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.25vvl1tdwgu8.webp" width="50%" />



<br>

## 子进程开销和优化

- 

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.5u5w40mtg9s0.webp" width="50%" />

<br>

## 硬盘优化

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.5lnrmjt1hw40.webp" width="50%" />

<br>

## AOF追加阻塞

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.6v89xx5ytl00.webp" width="50%" />

<br>

## AOF阻塞定位

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.3t06d4zqkpo0.webp" width="50%" />

<br>

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.3qdujdqyup60.webp" width="50%" />

<br>

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.60mk8wrt4lk0.webp" width="50%" />

<br>

# Redis复制的原理与优化

## 什么是主从复制

- 一个master可以有多个slave，但一个slave只能有一个master
- 数据流向必须是单向的。master -> slave
- 变成从节点前会把数据清楚



## 主从复制作用

- 一个数据提供了多个副本（成为高可用、分布式的基础）
- 扩展读性能（读写分离）



## 主从复制实现

### slaveof 命令

- 复制（slaveof 这个命令是异步的）

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.2pa32y5jfz80.webp" width="60%"/>

<br>

- 取消复制

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.7jnus8oq23k0.webp" width="60%"/>

<br>

### 配置

```shell
slaveof ip port
slave-read-only yes
```



### 主从复制-命令和配置的比较

| 方式 |    命令    |   配置   |
| :--: | :--------: | :------: |
| 优点 |  无需重启  | 统一配置 |
| 缺点 | 不便于管理 | 需要管理 |



### 主从配置操作

```shell
info replication
```



## 主从复制原理

### 全量复制过程原理

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.312gr8tguvm0.webp" width="50%" />

<br>

### 全量复制开销

1. bgsave时间
2. RDB文件网络传输时间
3. 从节点清空数据时间
4. 从节点加载RDB时间
5. 可能的AOF重写时间



### 部分复制过程原理

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.3vhs1nc1dzi0.webp" width="50%" />

<br>

## 主从复制中的故障处理与常见问题

- 故障不可避免
- 自动故障转移
- 故障分为master故障和slave故障



### 读写分离问题

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.a738ve0k3y8.webp" width="50%" />

<br>

### 配置不一致

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.nl2kmy53a0w.webp" width="50%" />

<br>

### 规避全量复制

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.6pe5jkk3nm80.webp" width="50%" />

<br>

### 规避复制风暴

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.361o49e7h2i0.webp" width="50%" />

<br>

# Redis Sentinel







# 第9章 初识Redis Cluster





<br>

# 学习备注

> 1. jedis 需要熟悉,有些代码还要手动过一遍才是
> 2. 生产环境普通用户后台启动redis
> 3. 部分图片内容是否应该转化为代码呢？
> 4. bitmap不太懂，还需要深入理解。还包括 hyperloglog、geo
> 5. RDB和AOF的恢复原理和过程是怎么样子的？
> 6. 主从复制操作虽然简单，但是最好是实践一下

<img src="" width="50%" />

<br>









