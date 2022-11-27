---
title: 《MySQL 8.0详解与实战》study notes
author: ratears
categories:
  - [database,MySQL]
tags:
  - MySQL
  - database
date: 2022-10-24 11:00:34
updated: 2022-10-24 11:00:34
---





# 数据库选型

## SQL VS NOSQL

| 类型  |                    示例                    | 特点                                                         | 适用场景                                                     |
| :---: | :----------------------------------------: | :----------------------------------------------------------- | :----------------------------------------------------------- |
|  SQL  | MySQL<br>Oracle<br>SQLServer<br>PostGreSQL | 数据结构化存储在二维表格中。<br/>支持事务的 ACID 特性。<br/>支持使用SQL语言对存储在其中的数据进行操作 | 数据之间存在着一定关系，需要关联查询数据的场景。<br/>需要事务支持的业务场景。<br/>需要使用SQL语言灵活操作数据的场景。 |
| NOSQL |    HBase<br>MongoDB<br>Redis<br>Hadoop     | 存储结构灵活，没有固定的存储结构<br/>对事务的支持比较弱，但对数据的并发处理性能高<br/>大多不使用SQL语言操作数据 | 数据结构不固定的场景<br/>对事务要求不高，但读写并发比较大的场景。<br/>对数据处理操作比较简单的场景 |

<br>

<br>

## 关系数据库选型原则

- 数据库使用的广泛性

  <img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.4hxh96wd6lc0.webp" width="80%">

- 数据库的可扩展性

  - 支持基于二进制日志的逻辑复制
  - 存在多种第三方数据库中间件，支持读写分离，分库分表

- 数据库的安全性和稳定性

  - MySQL主从复制集群可达到99%的可用性
  - 配合主从复制高可用架构，可达到99%的可用性
  - 支持对存储在MySQL的数据，进行分级安全控制

- 数据库所支持的系统

  - 支持Linux操作系统
  - 支持windows操作系统

- 数据库的使用成本

  - 社区版本免费
  - 使用人员众多，社区活跃，可以方便获取技术支持

<br>

<br>

## MySQL 8.x 的安装

- 服务器环境：CentOS 7 

```shell
# （1）下载mysql的安装包
[root@localhost ~]# cd /install/
[root@localhost install]# wget https://cdn.mysql.com//Downloads/MySQL-8.0/mysql-8.0.31-el7-x86_64.tar.gz

# （2）解压
[root@localhost install]# tar -zxvf mysql-8.0.31-el7-x86_64.tar.gz

# （3）移动mysql的位置，创建软连接（便于管理）
[root@localhost install]# mv mysql-8.0.31-el7-x86_64 /usr/local/
[root@localhost install]# cd /usr/local/
[root@localhost local]# ln -s mysql-8.0.31-el7-x86_64/ mysql

# （4）配置mysql
# 先备份原 /etc/my.cnf 文件
[root@localhost mysql]# cp /etc/my.cnf /etc/my.cnf.bak
[root@localhost mysql]# vim /etc/my.cnf
```

```mysql
[client]
port            = 3306
socket          = /usr/local/mysql/data/mysql.sock
[mysqld]
# Skip #
skip_name_resolve              = 1
skip_external_locking          = 1 
skip_symbolic_links     = 1
# GENERAL #
user = mysql
default_storage_engine = InnoDB
character-set-server = utf8
socket  = /usr/local/mysql/data/mysql.sock
pid_file = /usr/local/mysql/data/mysqld.pid
basedir = /usr/local/mysql
port = 3306
bind-address = 0.0.0.0
explicit_defaults_for_timestamp = off
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
#read_only=on
# MyISAM #
key_buffer_size                = 32M
#myisam_recover                 = FORCE,BACKUP

# undo log #
innodb_undo_directory = /usr/local/mysql/undo
innodb_undo_tablespaces = 8

# SAFETY #
max_allowed_packet             = 100M
max_connect_errors             = 1000000
sysdate_is_now                 = 1
#innodb = FORCE
#innodb_strict_mode = 1
secure-file-priv='/tmp'
default_authentication_plugin='mysql_native_password'
# Replice #
 server-id = 1001
 relay_log = mysqld-relay-bin
 gtid_mode = on
 enforce-gtid-consistency
 log-slave-updates = on
 master_info_repository =TABLE
 relay_log_info_repository =TABLE


# DATA STORAGE #
 datadir = /usr/local/mysql/data/
 tmpdir = /tmp
 
# BINARY LOGGING #
 log_bin = /usr/local/mysql/sql_log/mysql-bin
 max_binlog_size = 1000M
 binlog_format = row
 binlog_expire_logs_seconds=86400
# sync_binlog = 1

 # CACHES AND LIMITS #
 tmp_table_size                 = 32M
 max_heap_table_size            = 32M
 max_connections                = 4000
 thread_cache_size              = 2048
 open_files_limit               = 65535
 table_definition_cache         = 4096
 table_open_cache               = 4096
 sort_buffer_size               = 2M
 read_buffer_size               = 2M
 read_rnd_buffer_size           = 2M
# thread_concurrency             = 24
 join_buffer_size = 1M
# table_cache = 32768
 thread_stack = 512k
 max_length_for_sort_data = 16k


 # INNODB #
 innodb_flush_method            = O_DIRECT
 innodb_log_buffer_size = 16M
 innodb_flush_log_at_trx_commit = 2
 innodb_file_per_table          = 1
 innodb_buffer_pool_size        = 256M
 #innodb_buffer_pool_instances = 8
 innodb_stats_on_metadata = off
 innodb_open_files = 8192
 innodb_read_io_threads = 16
 innodb_write_io_threads = 16
 innodb_io_capacity = 20000
 innodb_thread_concurrency = 0
 innodb_lock_wait_timeout = 60
 innodb_old_blocks_time=1000
 innodb_use_native_aio = 1
 innodb_purge_threads=1
 innodb_change_buffering=all
 innodb_log_file_size = 64M
 innodb_log_files_in_group = 2
 innodb_data_file_path  = ibdata1:256M:autoextend
 
 innodb_rollback_on_timeout=on
 # LOGGING #
 log_error                      = /usr/local/mysql/sql_log/mysql-error.log
 # log_queries_not_using_indexes  = 1
 # slow_query_log                 = 1
  slow_query_log_file            = /usr/local/mysql/sql_log/slowlog.log

 # TimeOut #
 #interactive_timeout = 30
 #wait_timeout        = 30
 #net_read_timeout = 60

[mysqldump]
quick
max_allowed_packet = 100M

[mysql]
no-auto-rehash
# Remove the next comment character if you are not familiar with SQL
#safe-updates

[myisamchk]
key_buffer_size = 256M
sort_buffer_size = 256M
read_buffer = 2M
write_buffer = 2M

[mysqlhotcopy]
interactive-timeout
```

```shell
# （5）创建管理mysql的用户mysql
[root@localhost mysql]# useradd mysql

# （6）创建mysql相关数据目录，并修改文件权限
[root@localhost mysql]# mkdir data sql_log undo
[root@localhost mysql]# chown -R mysql:mysql data sql_log undo
[root@localhost mysql]# ll
total 300
drwxr-xr-x.  2  7161 31415   4096 Sep 14 01:50 bin
drwxr-xr-x.  2 mysql mysql      6 Oct 24 13:43 data
drwxr-xr-x.  2  7161 31415     55 Sep 14 01:50 docs
drwxr-xr-x.  3  7161 31415   4096 Sep 14 01:50 include
drwxr-xr-x.  6  7161 31415    201 Sep 14 01:50 lib
-rw-r--r--.  1  7161 31415 287627 Sep 14 00:15 LICENSE
drwxr-xr-x.  4  7161 31415     30 Sep 14 01:50 man
-rw-r--r--.  1  7161 31415    666 Sep 14 00:15 README
drwxr-xr-x. 28  7161 31415   4096 Sep 14 01:50 share
drwxr-xr-x.  2 mysql mysql      6 Oct 24 13:43 sql_log
drwxr-xr-x.  2  7161 31415     77 Sep 14 01:50 support-files
drwxr-xr-x.  2 mysql mysql      6 Oct 24 13:43 undo

# （7）配置mysql的环境变量
[root@localhost mysql]# vim /etc/profile
# 在 /etc/profile 文件末尾添加如下内容
export PATH=$PATH:/usr/local/mysql/bin

[root@localhost mysql]# source /etc/profile

# （8）初始化mysql数据库
[root@localhost mysql]# mysqld --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data
# 验证：数据库初始化成功后，会在data目录下有相关文件生成

# （9）拷贝数据库启动脚本
[root@localhost mysql]# cp support-files/mysql.server /etc/init.d/mysqld

# 至此，mysql数据库安装完成
```

<br>

<br>

## MySQL 启动

```mysql
[root@localhost mysql]# /etc/init.d/mysqld start
Starting MySQL.. SUCCESS!

# 获取mysql的初始密码
[root@localhost mysql]# grep password sql_log/mysql-error.log

# 登录mysql后，修改mysql的密码
mysql> alter user user() identified by 'root';
Query OK, 0 rows affected (0.00 sec)
```

<br>

<br>

<br>

# 数据库设计

- 业务分析	=》	逻辑设计	=》	数据类型	=》	对象命名	=》 建立库表



## 逻辑设计

### 宽表模式存在的问题

- 宽表模式：指把所有需求字段设计在一张表中

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.3dkqourzzp40.webp" width="70%">

<br>

<br>

#### 数据冗余

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.693sty7axdk0.webp" width="80%">

<br>

- 相同数据在一个表中出现了多次（增加数据占用空间）
- 需要对数据进行多次维护（如果讲师的职位发生了变化，就需要对多条数据进行维护，如果有数据没有维护，就会导致数据不一致）

<br>

<br>

#### 宽表模式的应用场景

- 配合列存储的数据报表应用

<br>

<br>

## 数据库设计范式

- 第一范式：要求有主键，并且要求每一个字段原子性不可再分
- 第二范式：表中必须存在业务主键，并且非主键全部依赖于业务主键
- 第三范式：表中的非主键列之间不能相互依赖

<br>

<br>

- 反反范式化设计
  - 范式化设计存在的问题：为了满足范式化设计要求，我们对表拆分的很细。但是在查询时候关联的表会很多，影响性能
  - 为了提高查询性能，我们需要反范式化设计

<br>

<br>

## 物理设计

### MySQL常见的存储引擎

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.2pqx87bjieq0.webp" width="80%">

<br>

<br>

### InnoDB存储引擎的特点

- 事物型存储引擎支持ACID
- 数据按主键聚集存储
- 支持行级锁及MVCC
- 支持Btree和自适应Hash索引
- 支持全文和空间索引

<br>

<br>

### 整数类型

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.75n135ulfk00.webp" width="80%">

<br>

<br>

### 实数类型

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.4d7mn2i88vq0.webp" width="80%">

<br>

<br>

### 常用时间类型

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.5e3m2d5s8yc0.webp" width="80%">

<br>

<br>

### 字符串类型

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.618wwafqys00.webp" width="80%">

<br>

<br>

### 选择合适的数据类型

- 优先选择符合存储数据需求的最小数据类型
- 谨慎使用ENUM,TEXT字符串类型
- 同财务相关的数据类型，必须使用decimal类型

<br>

<br>

### 数据库对象命名原则

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.6bx7vkwrtgc0.webp" width="60%">

<br>

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.58c8x2q3kq80.webp" width="60%">

<br>

<br>

<br>

# MySQL 访问数据库

## MySQL 访问异常处理

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.6si7f2uku6w0.webp" width="80%">

<br>

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.4zrntdxrjtk0.webp" width="80%">

<br>









<br>

<br>

<br>

# 学习备注

> 1. 数据类型这块还需要加强一下呢

```html
&emsp;&emsp;
```

<br>

<br>

<br>

<img src="" width="80%">

<br>

