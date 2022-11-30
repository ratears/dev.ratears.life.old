---
title: 《heima-Mysql高级（4天）》study notes
author: ratears
categories:
  - [database,MySQL]
tags:
  - MySQL
  - database
date: 2022-10-24 14:29:42
updated: 2022-10-24 14:29:42
---



# MySQL高级课程简介

| 序号 | Day01              | Day02       | Day03          | Day04          |
| :--: | ------------------ | ----------- | -------------- | -------------- |
|  1   | Linux系统安装MySQL | 体系结构    | 应用优化       | MySQL 常用工具 |
|  2   | 索引               | 存储引擎    | 查询缓存优化   | MySQL 日志     |
|  3   | 视图               | 优化SQL步骤 | 内存管理及优化 | MySQL 主从复制 |
|  4   | 存储过程和函数     | 索引使用    | MySQL锁问题    | 综合案例       |
|  5   | 触发器             | SQL优化     | 常用SQL技巧    |                |

<br>

<br>

<br>

# Mysql高级-day01

## 安装MySQL 5.6（rpm方式）

```shell
#	1). 卸载 centos 中预安装的 mysql/mariadb,并删除相关文件
rpm -qa | grep -i mysql
rpm -e mysql-libs-5.1.71-1.el6.x86_64 --nodeps

#	2). 下载 mysql 的安装包
wget https://cdn.mysql.com/archives/mysql-5.6/MySQL-5.6.22-1.el7.x86_64.rpm-bundle.tar

#	3). 解压 mysql 的安装包
tar -xvf MySQL-5.6.22-1.el7.x86_64.rpm-bundle.tar

#	4). 安装依赖包
yum -y install libaio.so.1 libgcc_s.so.1 libstdc++.so.6 libncurses.so.5 --setopt=protected_multilib=false autoconf net-tools
yum update libstdc++-4.4.7-4.el6.x86_64

#	5). 安装 mysql-client
rpm -ivh MySQL-client-5.6.22-1.el7.x86_64.rpm


#	6). 安装 mysql-server
rpm -ivh MySQL-server-5.6.22-1.el7.x86_64.rpm

# 如果安装过程中缺少依赖则安装相关依赖。若安装失败，可以卸载后，删除mysql有关的文件，再重新安装
```

<br>

<br>

### 启动 MySQL 服务

```shell
service mysql start
service mysql stop
service mysql status
service mysql restart
```

<br>

<br>

### 登录MySQL

```shell
# mysql 安装完成之后, 会自动生成一个随机的密码, 并且保存在一个密码文件中 : /root/.mysql_secret

# 登录之后, 修改密码 :
set password = password('root');

# 授权远程访问 :
grant all privileges on *.* to 'root' @'%' identified by 'root';
flush privileges;
```

<br>

<br>

<br>

## 索引

### 索引概述

- MySQL官方对索引的定义为：索引（index）是帮助MySQL高效获取数据的数据结构（有序）。在数据之外，数据库系统还维护者满足特定查找算法的数据结构，这些数据结构以某种方式引用（指向）数据， 这样就可以在这些数据结构上实现高级查找算法，这种数据结构就是索引

<br>

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.6iks54eq7940.webp" width="80%">

<br>

> 左边是数据表，一共有两列七条记录，最左边的是数据记录的物理地址（注意逻辑上相邻的记录在磁盘上也并不是一定物理相邻的）。为了加快Col2的查找，可以维护一个右边所示的二叉查找树，每个节点分别包含索引键值和一个指向对应数据记录物理地址的指针，这样就可以运用二叉查找快速获取到相应数据。

> 一般来说索引本身也很大，不可能全部存储在内存中，因此索引往往以索引文件的形式存储在磁盘上。索引是数据库中用来提高性能的最常用的工具

<br>

<br>

### 索引优势劣势

- 优势
  - 1） 类似于书籍的目录索引，提高数据检索的效率，降低数据库的IO成本
  - 2） 通过索引列对数据进行排序，降低数据排序的成本，降低CPU的消耗
- 劣势
  - 1） 实际上索引也是一张表，该表中保存了主键与索引字段，并指向实体类的记录，所以索引列也是要占用空间
  - 2） 虽然索引大大提高了查询效率，同时却也降低更新表的速度，如对表进行INSERT、UPDATE、DELETE。因为更新表时，MySQL 不仅要保存数据，还要保存一下索引文件每次更新添加了索引列的字段，都会调整因为更新所带来的键值变化后的索引信息。

<br>

<br>

### 索引结构

- 索引是在MySQL的存储引擎层中实现的，而不是在服务器层实现的。所以每种存储引擎的索引都不一定完全相同，也不是所有的存储引擎都支持所有的索引类型的。MySQL目前提供了以下4种索引：
  - BTREE 索引 ： 最常见的索引类型，大部分索引都支持 B 树索引
  - HASH 索引：只有Memory引擎支持 ， 使用场景简单
  - R-tree 索引（空间索引）：空间索引是MyISAM引擎的一个特殊索引类型，主要用于地理空间数据类型，通常使用较少
  - Full-text （全文索引） ：全文索引也是MyISAM的一个特殊索引类型，主要用于全文索引，InnoDB从Mysql5.6版本开始支持全文索引



- **MyISAM、InnoDB、Memory三种存储引擎对各种索引类型的支持**

|    索引     |   InnoDB引擎    | MyISAM引擎 | Memory引擎 |
| :---------: | :-------------: | :--------: | :--------: |
|  BTREE索引  |      支持       |    支持    |    支持    |
|  HASH 索引  |     不支持      |   不支持   |    支持    |
| R-tree 索引 |     不支持      |    支持    |   不支持   |
|  Full-text  | 5.6版本之后支持 |    支持    |   不支持   |

- 我们平常所说的索引，如果没有特别指明，都是指B+树（多路搜索树，并不一定是二叉的）结构组织的索引。其中聚集索引、复合索引、前缀索引、唯一索引默认都是使用 B+tree 索引，统称为 索引。

<br>

#### BTREE 结构

- BTree又叫多路平衡搜索树，一颗m叉的BTree特性如下：
  - 树中每个节点最多包含m个孩子
  - 除根节点与叶子节点外，每个节点至少有[ceil(m/2)]个孩子
  - 若根节点不是叶子节点，则至少有两个孩子
  - 所有的叶子节点都在同一层
  - 每个非叶子节点由n个key与n+1个指针组成，其中[ceil(m/2)-1] <= n <= m-1

> 以5叉BTree为例，key的数量：公式推导[ceil(m/2)-1] <= n <= m-1。所以 2 <= n <=4 。当n>4时，中间节点分裂到父节点，两边节点分裂

> BTREE树 和 二叉树 相比， 查询数据的效率更高， 因为对于相同的数据量来说，BTREE的层级结构比二叉树小，因此搜索速度快

<br>

#### B+TREE 结构

- B+Tree为BTree的变种，B+Tree与BTree的区别为：
  - 1). n叉B+Tree最多含有n个key，而BTree最多含有n-1个key
  - 2). B+Tree的叶子节点保存所有的key信息，依key大小顺序排列
  - 3). 所有的非叶子节点都可以看作是key的索引部分

<br>

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.6jme7w6ntz40.webp" width="80%">

<br>

- 由于B+Tree只有叶子节点保存key信息，查询任何key都要从root走到叶子。所以B+Tree的查询效率更加稳定

<br>

### MySQL中的B+Tree

- MySql索引数据结构对经典的B+Tree进行了优化。在原B+Tree的基础上，增加一个指向相邻叶子节点的链表指针，就形成了带有顺序指针的B+Tree，提高区间访问的性能



- MySQL中的 B+Tree 索引结构示意图:

<br>

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.7bv4u1b5wk40.webp" width="100%">

<br>

<br>

### 索引分类

- 1） 单值索引 ：即一个索引只包含单个列，一个表可以有多个单列索引
- 2） 唯一索引 ：索引列的值必须唯一，但允许有空值
- 3） 复合索引 ：即一个索引包含多个列

<br>

<br>

### 索引语法

- 索引在创建表的时候，可以同时创建， 也可以随时增加新的索引

准备环境:

```mysql
create databse demo_01 default charset=uft8mb4;

use demo_01;

create table `city` (
	`city_id` int(11) not null auto_increment,
    `city_name` varchar(50) not null,
    `country_id` int(11) not null,

)
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