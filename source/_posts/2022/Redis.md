---
title: Redis
author: sonzonzy
date: 2022-07-02 01:47:47
updated: 2022-07-02 01:47:47
categories:
  - [database,redis]
tags:
  - redis
---



# 初识Redis

## SQL与NoSQL

|          |                     SQL（关系型数据库）                      |                    NoSQL（非关系数据库）                     |
| :------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| 数据结构 |                            结构化                            |                           非结构化                           |
| 数据关联 |                            关联的                            |                           无关联的                           |
| 查询方式 |                           SQL查询                            |                            非SQL                             |
| 事务特性 |                             ACID                             |                             BASE                             |
| 存储方式 |                             磁盘                             |                             内存                             |
|  扩展性  |                             垂直                             |                             水平                             |
| 使用场景 | （1）数据结构固定<br/>（2）相关业务对数据安全性、一致性有较高要求 | （1）数据结构不固定<br/>（2）对一致性、安全性要求不高<br/>（3）对性能要求 |

<br/>

- 数据结构

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/d30b03101d4e1a06a2ed1e5be56700b.31vpj58sbty0.webp" width="80%"/>

<br/>

- 数据关联

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/e3f1c89793ff81d6babb11413fe2ebe.1smbrmpvghq8.webp" width="80%"/>

<br/>

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/1d6675040ee9bbcce9c49e3cc39747c.2fpd2jgc3xxc.webp" width="80%"/>

<br/>

- SQL查询

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/5cd0215e592bef1b35b0203d36b7a48.3x5xuvughi00.webp" width="80%" />

<br/>

### 常见NoSQL





## Redis简介

- Redis诞生于2009年全称是Remote Dictionary Server，远程词典服务器，是一个基于内存的键值型NoSQL数据库。



## Redis特征

- 键值（key-value）型，value支持多种不同数据结构，功能丰富
- 单线程，每个命令具备原子性
- 低延迟，速度快（基于内存、IO多路复用、良好的编码）
- 支持数据持久化
- 支持主从集群、分片集群
- 支持多语言客户端



## 单机安装Redis

```bash
# 安装依赖 （Redis是基于C语言编写的，因此首先需要安装Redis所需要的gcc依赖）
yum install -y gcc tcl

# 上传/下载 安装包并解压
cd /usr/local/src/
wget https://download.redis.io/releases/redis-6.2.7.tar.gz
tar -zxvf redis-6.2.7.tar.gz

# 编译安装
cd /usr/local/src/redis-6.2.7
make && make install

# 安装完成后。默认的安装路径是在 `/usr/local/bin`目录下

[root@bogon src]# cd /usr/local/bin/
[root@bogon bin]# ll
总用量 18924
-rwxr-xr-x. 1 root root 4830088 7月  25 05:51 redis-benchmark
lrwxrwxrwx. 1 root root      12 7月  25 05:51 redis-check-aof -> redis-server
lrwxrwxrwx. 1 root root      12 7月  25 05:51 redis-check-rdb -> redis-server
-rwxr-xr-x. 1 root root 5004192 7月  25 05:51 redis-cli
lrwxrwxrwx. 1 root root      12 7月  25 05:51 redis-sentinel -> redis-server
-rwxr-xr-x. 1 root root 9535928 7月  25 05:51 redis-server

# 该目录已经默认配置到环境变量，因此可以在任意目录下运行这些命令。其中：

redis-server
# 是redis的服务端启动脚本

redis-cli
# 是redis提供的命令行客户端

redis-sentinel
# 是redis的哨兵启动脚本
```

<br/>

## Redis启动

### 默认启动（前台启动）

- 这种启动属于`前台启动`，会阻塞整个会话窗口，窗口关闭或者按下`CTRL + C`则Redis停止。不推荐使用。

```bash
redis-server
```

<br/>

### 指定配置启动

- 如果要让Redis以`后台`方式启动，则必须修改Redis配置文件

```bash
# 我们先将这个配置文件备份一份：
cp redis.conf redis.conf.bck
```

- 然后修改redis.conf文件中的一些配置

```properties
# 允许访问的地址，默认是127.0.0.1，会导致只能在本地访问。修改为0.0.0.0则可以在任意IP访问，生产环境不要设置为0.0.0.0
bind 0.0.0.0

# 守护进程，修改为yes后即可后台运行
daemonize yes 

# 密码，设置后访问Redis必须输入密码
requirepass redis@2022
```

- Redis的其它常见配置

```properties
# 监听的端口
port 6379

# 工作目录，默认是当前目录，也就是运行redis-server时的命令，日志、持久化等文件会保存在这个目录
dir .

# 数据库数量，设置为1，代表只使用1个库，默认有16个库，编号0~15
databases 1

# 设置redis能够使用的最大内存
maxmemory 512mb

# 日志文件，默认为空，不记录日志，可以指定日志文件名
logfile "redis.6379.log"
```

- 指定配置启动

```bash
# 进入redis安装目录 
[root@localhost redis]# /usr/local/bin

[root@bogon bin]# ll
总用量 19112
-rw-r--r--. 1 root root   93916 7月  25 06:16 redis.6379.conf
-rw-r--r--. 1 root root    1232 7月  25 06:20 redis.6379.log
-rwxr-xr-x. 1 root root 4830088 7月  25 05:51 redis-benchmark
lrwxrwxrwx. 1 root root      12 7月  25 05:51 redis-check-aof -> redis-server
lrwxrwxrwx. 1 root root      12 7月  25 05:51 redis-check-rdb -> redis-server
-rwxr-xr-x. 1 root root 5004192 7月  25 05:51 redis-cli
-rw-r--r--. 1 root root   93849 7月  25 06:08 redis.conf.bak
lrwxrwxrwx. 1 root root      12 7月  25 05:51 redis-sentinel -> redis-server
-rwxr-xr-x. 1 root root 9535928 7月  25 05:51 redis-server
[root@bogon bin]#

[root@localhost redis]# redis-server redis.6379.conf
[root@localhost redis]#
[root@bogon bin]# ps -ef |grep redis
root       8126      1  0 06:20 ?        00:00:00 redis-server 0.0.0.0:6379
root       8194   1552  0 06:21 pts/1    00:00:00 grep --color=auto redis
```

<br/>

### 开机自启（把redis添加到系统服务）

- 我们也可以通过配置来实现开机自启，首先，新建一个系统服务文件

```bash
vim /etc/systemd/system/redis.service
```

- 添加如下内容

```bash
[Unit]
Description=redis-server
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/bin/redis-server /usr/local/bin/redis.6379.conf
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

- 然后重载系统服务

```bash
systemctl daemon-reload
```

- 现在，我们可以用下面这组命令来操作redis了

```bash
# 启动
systemctl start redis
# 停止
systemctl stop redis
# 重启
systemctl restart redis
# 查看状态
systemctl status redis
```

- 执行下面的命令，可以让redis开机自启

```bash
systemctl enable redis
```

<br/>

## Redis的其它常见配置

```bash
# 监听的端口
port 6379

# 工作目录，默认是当前目录，也就是运行redis-server时的命令，日志、持久化等文件会保存在这个目录
dir .

# 数据库数量，设置为1，代表只使用1个库，默认有16个库，编号0~15
databases 1

# 设置redis能够使用的最大内存
maxmemory 512mb

# 日志文件，默认为空，不记录日志，可以指定日志文件名
logfile "redis.6379.log"
```

<br/>

## Redis停止

```bash
# 利用redis-cli来执行 shutdown 命令，即可停止 Redis 服务，
# 因为之前配置了密码，因此需要通过 -a 来指定密码
redis-cli -a redis shutdown
```

<br/>

# Redis客户端

## Redis命令行客户端

```bash
redis-cli [options] [commonds]
```

其中常见的options有：

- `-h 127.0.0.1`：指定要连接的redis节点的IP地址，默认是127.0.0.1
- `-p 6379`：指定要连接的redis节点的端口，默认是6379
- `-a 123321`：指定redis的访问密码 

其中的commonds就是Redis的操作命令，例如：

- `ping`：与redis服务端做心跳测试，服务端正常会返回`pong`

不指定commond时，会进入`redis-cli`的交互控制台：

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image-20211211110439353.2aqqz6hz4m4g.webp" width="80%" />

<br/>

## 图形化桌面客户端

- https://github.com/lework/RedisDesktopManager-Windows/releases

<br/>

## Redis的Java客户端

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.6lpitp9r4g00.webp" width="80%" />

<br/>

### Jedis

#### Jedis使用的基本步骤

（1）引入依赖

```xml
<!--Redis依赖-->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>4.2.0</version>
</dependency>
```

（2）创建Jedis对象，建立连接

```java
	private Jedis jedis;

    @BeforeEach
    void testJedis() {
        jedis = new Jedis("192.168.163.200",6379);
        jedis.auth("redis");
        jedis.select(0);
    }
```

（3）使用Jedis，方法名与Redis命令一致
```java
    @Test
    void testString() {
        // 插入数据，方法名称就是redis命令名称，非常简单
        String result = jedis.set("name", "张三");
        System.out.println("result = " + result);
        // 获取数据
        String name = jedis.get("name");
        System.out.println("name = " + name);
    }
```


（4）释放资源
```bash
	@Deprecated
    void tearDown() {
        if (jedis != null) {
            jedis.close();
        }
    }
```

<br/>

#### Jedis连接池

- Jedis本身是线程不安全的，并且频繁的创建和销毁连接会有性能损耗，因此推荐大家使用Jedis连接池代替Jedis的直连方式

```java
public class JedisConnectionFactory {
    private static final JedisPool jedisPool;

    static {
        JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
        // 最大连接
        jedisPoolConfig.setMaxTotal(8);
        // 最大空闲连接
        jedisPoolConfig.setMaxIdle(8); 
        // 最小空闲连接
        jedisPoolConfig.setMinIdle(0);
        // 设置最长等待时间， ms
        jedisPoolConfig.setMaxWaitMillis(200);
        jedisPool = new JedisPool(jedisPoolConfig, "localhost", 6379,
                1000, "123321");
    }
    // 获取Jedis对象
    public static Jedis getJedis(){
        return jedisPool.getResource();
    }
}
```

<br/>

### SpringDataRedis

#### SpringDataRedis简介

- SpringData是Spring中数据操作的模块，包含对各种数据库的集成，其中对Redis的集成模块就叫做SpringDataRedis，官网地址：https://spring.io/projects/spring-data-redis
  - 提供了对不同Redis客户端的整合（Lettuce和Jedis）
  - 提供了RedisTemplate统一API来操作Redis
  - 支持Redis的发布订阅模型
  - 支持Redis哨兵和Redis集群
  - 支持基于Lettuce的响应式编程
  - 支持基于JDK、JSON、字符串、Spring对象的数据序列化及反序列化
  - 支持基于Redis的JDKCollection实现

- SpringDataRedis中提供了RedisTemplate工具类，其中封装了各种对Redis的操作。并且将不同数据类型的操作API封装到了不同的类型中：

|               API               |   返回值类型    |         说明          |
| :-----------------------------: | :-------------: | :-------------------: |
| **redisTemplate**.opsForValue() | ValueOperations |  操作String类型数据   |
| **redisTemplate**.opsForHash()  | HashOperations  |   操作Hash类型数据    |
| **redisTemplate**.opsForList()  | ListOperations  |   操作List类型数据    |
|  **redisTemplate**.opsForSet()  |  SetOperations  |    操作Set类型数据    |
| **redisTemplate**.opsForZSet()  | ZSetOperations  | 操作SortedSet类型数据 |
|        **redisTemplate**        |                 |      通用的命令       |

<br/>

#### SpringDataRedis快速入门

- SpringBoot已经提供了对SpringDataRedis的支持，使用非常简单

（1）引入spring-boot-starter-data-redis依赖

```xml
<!--Redis依赖-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<!--连接池依赖-->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```

（2）在application.yml配置Redis信息
```yaml
spring:
  redis:
    host: 192.168.163.200
    port: 6379
    password: redis
    lettuce:
      pool:
        max-active: 8 # 最大连接
        max-idle: 8 # 最大空闲连接
        min-idle: 0 # 最小空闲连接
        max-wait: 100 # 连接等待时间
```
（3）注入RedisTemplate
```xml
@Autowired
private RedisTemplate redisTemplate;
```

（4）编写测试

```java
@SpringBootTest
public class RedisTest {

    @Autowired
    private RedisTemplate redisTemplate;

    @Test
    void testString() { 
        // 插入一条string类型数据
        redisTemplate.opsForValue().set("name", "李四");
        // 读取一条string类型数据
        Object name = redisTemplate.opsForValue().get("name");
        System.out.println("name = " + name);
    }
}
```

<br/>

# Redis 数据类型与常见命令

## Redis 数据结构介绍

- Redis是一个key-value的数据库，key一般是String类型，不过value的类型多种多样

<img src="https://git.poker/sonzonzy/image-hosting/blob/main/blog-img-bed/image.43orz7ks9kq0.webp?raw=true" width="80%"/>

## Redis通用命令

- 通用指令是部分数据类型的，都可以使用的指令，常见的有：
  - KEYS：查看符合模板的所有key
  - DEL：删除一个指定的key
  - EXISTS：判断key是否存在
  - EXPIRE：给一个key设置有效期，有效期到期时该key会被自动删除
  - TTL：查看一个KEY的剩余有效期
  - 通过help [command] 可以查看一个命令的具体用法









<br/>

# Redis Serializer（Redis序列化）

## SpringDataRedis的序列化方式

- RedisTemplate可以接收任意Object作为值写入Redis，只不过写入前会把Object序列化为字节形式，默认是采用JDK序列化

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.4cko09vu3y8.webp" width="80%" />

- 缺点：
  - 可读性差
  - 内存占用较大

<br/>

## 自定义RedisTemplate的序列化方式

```java
@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        // 创建Template
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        // 设置连接工厂
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        // 设置序列化工具
        GenericJackson2JsonRedisSerializer jsonRedisSerializer =
                new GenericJackson2JsonRedisSerializer();
        // key和 hashKey采用 string序列化
        redisTemplate.setKeySerializer(RedisSerializer.string());
        redisTemplate.setHashKeySerializer(RedisSerializer.string());
        // value和 hashValue采用 JSON序列化
        redisTemplate.setValueSerializer(jsonRedisSerializer);
        redisTemplate.setHashValueSerializer(jsonRedisSerializer);
        return redisTemplate;
    }
}
```

```java
@SpringBootTest
public class RedisTest2 {

    @Autowired
    private RedisTemplate<String, Object> redisTemplate;

    @Test
    void testString() {
        // 插入一条string类型数据
        redisTemplate.opsForValue().set("name", "李四");
        // 读取一条string类型数据
        Object name = redisTemplate.opsForValue().get("name");
        System.out.println("name = " + name);
    }
    @Test
    void testSaveObject() {
        redisTemplate.opsForValue().set("user:100",new Person("果冻",28));
        Person person = (Person) redisTemplate.opsForValue().get("user:100");
        System.out.println(person);
    }
```

<br/>

## StringRedisTemplate（RedisTemplate的序列化方式优化）

- 尽管JSON的序列化方式可以满足我们的需求，但依然存在一些问题

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.2ajrax16lk74.webp"  width="80%" />

- 为了在反序列化时知道对象的类型，JSON序列化器会将类的class类型写入json结果中，存入Redis，会带来额外的内存开销

- 为了节省内存空间，我们并不会使用JSON序列化器来处理value，而是统一使用String序列化器，要求只能存储String类型的key和value。当需要存储Java对象时，手动完成对象的序列化和反序列化

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.35s7tory1l60.webp" width="80%" />

- Spring默认提供了一个StringRedisTemplate类，它的key和value的序列化方式默认就是String方式。省去了我们自定义RedisTemplate的过程：

```java
@SpringBootTest
public class StingRedisTemplateTests {

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    // JSON工具
    private static final ObjectMapper mapper = new ObjectMapper();

    @Test
    void testStringTemplate() throws JsonProcessingException {
        // 准备对象
        Person user = new Person("lily", 18);
        // 手动序列化
        String json = mapper.writeValueAsString(user);
        // 写入一条数据到redis
        stringRedisTemplate.opsForValue().set("user:200", json);

        // 读取数据
        String val = stringRedisTemplate.opsForValue().get("user:200");
        // 反序列化
        Person user1 = mapper.readValue(val, Person.class);
        System.out.println("user1 = " + user1);
    }
}
```

<br/>

## RedisTemplate序列化总结

### RedisTemplate的两种序列化实践方案：

- 方案一：

  - 自定义RedisTemplate
  - 修改RedisTemplate的序列化器为GenericJackson2JsonRedisSerializer

  

- 方案二：

  - 使用StringRedisTemplate
  - 写入Redis时，手动把对象序列化为JSON
  - 读取Redis时，手动把读取到的JSON反序列化为对象

<br/>

# Redis企业实战

## 商户点评

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.6uz0qkh3xio0.webp" width="80%" />

<br/>

### 目录

- 短信登录
- 商户查询缓存
- 优惠券秒杀
- 达人探店
- 好友关注
- 附近的商户
- 用户签到
- UV统计

<br/>

### 短信登录

#### （1）导入商户点评项目

- [项目下载地址](https://gitee.com/ratears/data-resource/tree/master/db/redis/heima-dianping-init_and_import)

> [hm-dianping.zip](https://gitee.com/ratears/data-resource/blob/master/db/redis/heima-dianping-init_and_import/hm-dianping.zip) (将其下载解压缩后复制到idea工作空间，然后利用idea打开即可)（修改自己的MySQL和Redis配置）
>
> - 启动项目后，在浏览器访问：http://localhost:8081/shop-type/list ，如果可以看到数据则证明运行没有问题
>
> [hmdp.sql](https://gitee.com/ratears/data-resource/blob/master/db/redis/heima-dianping-init_and_import/hmdp.sql)（导入SQL文件。Mysql的版本采用5.7及以上版本）（注意先创建数据库）
>
> [nginx-1.18.0.zip](https://gitee.com/ratears/data-resource/blob/master/db/redis/heima-dianping-init_and_import/nginx-1.18.0.zip)（windows版本。解压缩后启动即可）
>
> - 访问: [http://127.0.0.1:8080](http://127.0.0.1:8080/) ，即可看到页面

<br/>

<img src="https://git.poker/sonzonzy/image-hosting/blob/main/blog-img-bed/image.32ghgq3ldhe0.webp?raw=true" width="80%"/>

<br/>

（2）基于Session实现登录

<img src="https://git.poker/sonzonzy/image-hosting/blob/main/blog-img-bed/image.1jcturrchsbk.webp?raw=true" width="90%" />

- 集群的session共享问题
- 基于Redis实现共享session登录



















