---
title: 《Kafka多维度系统精讲，从入门到熟练掌握》study notes
author: ratears
categories:
	- [MQ,Kafka]
tags:
  - Kafka
date: 2022-11-26 22:00:57
updated: 2022-11-26 22:00:57
---



# 第1章 课程导学与学习指南





<br>

<br>

<br>

# 第2章 Kafka入门——开发环境准备



<br>

<br>

<br>

# 第3章 Kafka入门——Kafka基础操作

## Kafka介绍

- 一个分布式流处理平台
- Kafka是基于zookeeper的分布式消息系统
- Kafka具有高吞吐率、高性能、实时及高可靠等特点

<br>

## Kafka安装

- 安装准备
  - jdk-8u181-linux-x64.tar.gz（因为Kafka是scala开发的，scala是基于jdk的，固需需要安装jdk）
  - apache-zookeeper-3.5.7-bin.tar.gz
  - kafka_2.11-2.4.0.tgz

<br>

### step1 安装jdk

```shell
# 准备jdk安装包，并解压到 /usr/local/ 目录下
tar -zxvf jdk-8u181-linux-x64.tar.gz -C /usr/local/

# 创建软连接
ln -s jdk-8u181-linux-x64 jdk1.8

#配置jdk环境变量
vim /etc/profile

#####################################################
# 文档末尾追加如下内容
export JAVA_HOME=/usr/local/jdk1.8
export PATH=$PATH:$JAVA_HOME/bin
#####################################################

source /etc/profile

# 验证jdk环境
java -version
```

<br>

### step2 安装zookeeper

```shell
# 准备zookeeper安装包，并解压到 /usr/local/ 目录下
tar -zxvf apache-zookeeper-3.5.7-bin.tar.gz -C /usr/local/

# 创建软连接
ln -s apache-zookeeper-3.5.7-bin zookeeper

# 准备配置文件
cd /usr/local/zookeeper/conf/

cp zoo_sample.cfg zoo.cfg

vim zoo.cfg

#####################################################
# 修改 dataDir 然后保存（生产环境应该把dataDir设置成磁盘比较大的目录）
dataDir=/usr/local/zookeeper/data
#####################################################

# 创建data目录
mkdir -p /usr/local/zookeeper/data

# 启动zookeeper
cd /usr/local/zookeeper/bin/
./zkServer.sh start

# 使用zookeeper 自带的客户端连接 zookeeper
cd /usr/local/zookeeper/bin/
./zkCli.sh

```

<br>

### step3 安装Kafka

```shell
# 准备zookeeper安装包，并解压到 /usr/local/ 目录下
tar -zxvf kafka_2.11-2.4.0.tar.gz -C /usr/local/

# 创建软连接
ln -s kafka_2.11-2.4.0 kafka

# 修改配置文件
cd /usr/local/kafka/config/

vim server.properties

#####################################################
listeners=PLAINTEXT://192.168.146.135:9092

advertised.listeners=PLAINTEXT://192.168.146.135:9092

log.dirs=/usr/local/kafka/kafka-logs

# 注意生产环境该配置会变化，目前我们就使用如下配置，保持不变
zookeeper.connect=localhost:2181
#####################################################

# 创建logs目录
mkdir -p /usr/local/kafka/kafka-logs
```

<br>

## Kafka常用命令

```shell
# 注意执行命令需要在kafka的根目录下

# 1、启动Kafka
bin/kafka-server-start.sh config/server.properties &

# 2、停止Kafka
bin/kafka-server-stop.sh

# 3、创建Topic
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic jiangzh-topic

# 4、查看已经创建的Topic信息
bin/kafka-topics.sh --list --zookeeper localhost:2181

# 5、发送消息
bin/kafka-console-producer.sh --broker-list 192.168.146.135:9092 --topic jiangzh-topic

# 6、接收消息
bin/kafka-console-consumer.sh --bootstrap-server 192.168.146.135:9092 --topic jiangzh-topic --from-beginning
```

<br>

## Kafka基本概念

- Topic：一个虚拟的概念，由1到多个Partitions组成
- Partition：实际消息存储单位
- Producer：消息生产者
- Consumer：消息消费者

<br>

<br>

<br>

# 第4章 Kafka核心API——Kafka客户端操作 

## 五类Kafka客户端 API

- Producer API: 向主题(一个或多个)发布消息；
- Consumer API: 订阅主题(一个或多个)，拉取这些主题上发布的消息；
- Stream API: 作为流处理器，从主题消费消息，向主题发布消息，把输出流转换为输入流；
- Connect API: 作为下游或上游，把主题连接到应用程序或数据系统(比如关系数据库)，通常不需要直接使用这些API，而是使用 现成的连接器；
- AdminClient API: 管理(或巡查) topic, brokers, 或其他 kafka 对象；

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.32hkgl5c6x20.webp" width="60%">

<br>

## AdminClient客户端建立

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.3dkg228f3zm0.webp" width="70%">

<br>

```java
public class AdminSimple {
    public static void main(String[] args) {
        AdminClient adminClient = adminClient();
        System.out.println("adminClient:"+adminClient);
    }

    public static AdminClient adminClient(){
        Properties properties =new Properties();
        properties.setProperty(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG,"192.168.146.135:9092");
        AdminClient adminClient = AdminClient.create(properties);
        return adminClient;
    }
}
```

<br>

## 创建Topic演示

```java
public class AdminSimple {
    public static void main(String[] args) {
//        AdminClient adminClient = adminClient();
//        System.out.println("adminClient:"+adminClient);
        createTopic();
    }

    public static void createTopic() {
        AdminClient adminClient = adminClient();
        short rs = 10;
        NewTopic newTopic = new NewTopic("demo_topic",10,rs);
        CreateTopicsResult topics = adminClient.createTopics(Arrays.asList(newTopic));
        System.out.println("CreateTopicsResult :"+topics.toString());
    }

    public static AdminClient adminClient(){
        Properties properties =new Properties();
        properties.setProperty(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG,"192.168.146.135:9092");
        AdminClient adminClient = AdminClient.create(properties);
        return adminClient;
    }
}
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