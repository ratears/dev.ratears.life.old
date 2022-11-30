---
title: 姜承尧 MySQL DBA实战 study notes
author: sonzonzy
date: 2022-05-18 01:33:04
updated: 2022-05-18 01:33:04
categories:
  - [database,mysql]
tags:
  mysql
---





# 第01天_Mysql简介与前景



<br>

<br>

<br>

# 第02天_MySQL安装与基本命令

## MySQL安装步骤

- 卸载操作系统自带的mariadb，下载mysql安装的的依赖
- 创建mysql用户组和用户
- 配置`/etc/my.cnf`
- 官网下载mysql数据库
- 创建目录，设置权限。初始化mysql元数据库
- 将mysql添加到启动项

<br>

<br>

## MySQL 5.6 安装过程

- 官网MySQL 5.6安装参考手册：[MySQL 5.6 Reference Manual-Installing MySQL on Unix/Linux Using Generic Binaries](https://dev.mysql.com/doc/refman/5.6/en/binary-installation.html)



- （1）卸载操作系统自带的 mariadb/mysql 数据库

```shell
[root@localhost ~]# rpm -qa | grep mariadb | xargs yum remove -y
```

- （2）创建数据目录

```shell
[root@localhost ~]# mkdir -p /install /backup /data
```

- （3）创建mysql用户和用户组

```shell
[root@localhost ~]# useradd mysql
```

- （4）下载安装过程中可能需要的依赖

```shell
[root@localhost ~]# yum install -y libaio autoconf
```

- （5）新建/编辑 `/etc/my.cnf` 文件（需要先配置好这个文件，否则，安装后启动报错）

`vim /etc/my.cnf`

```shell
[client]
# 所有客户端登录进去后可以配置的参数

[mysql]
# mysql这个命名登录进去后配置的参数


[mysqld]
# mysql服务器启动时候的参数
port = 3306
user = mysql
basedir = /usr/local/mysql
datadir = /data
```

- （6）下载mysql5.6的安装包，并解压到 `/usr/local/` 目录下，创建软连接（便于管理mysql）

```shell
[root@localhost install]# wget wget https://downloads.mysql.com/archives/get/p/23/file/mysql-5.6.51-linux-glibc2.12-x86_64.tar.gz

# 校验下载的数据库压缩包MD5值，和官网进行比较，辨别是否被篡改过
[root@localhost install]# md5sum mysql-5.6.51-linux-glibc2.12-x86_64.tar.gz

[root@localhost install]# tar -zxvf mysql-5.6.51-linux-glibc2.12-x86_64.tar.gz -C /usr/local/

[root@localhost local]# cd /usr/local/
[root@localhost local]# ln -s mysql-5.6.51-linux-glibc2.12-x86_64/ mysql
```

- （7）设置目录权限

```shell
[root@localhost local]# chown -R mysql.mysql /data/
[root@localhost local]# chmod 750 /data/
```

- （8）初始化mysql

```shell
[root@localhost local]# cd /usr/local/mysql

# mysql初始化数据库的语句，5.6 和 5.7 是不一样的，需要注意
[root@localhost mysql]# scripts/mysql_install_db --user=mysql

[root@localhost mysql]# bin/mysqld_safe --user=mysql &
# 拷贝启动脚本（服务）到指定位置，便于管理、使用
[root@localhost mysql]# cp support-files/mysql.server /etc/init.d/mysql.server
```

<br>

<br>

## MySQL 5.7 安装过程

- 官网MySQL 5.7安装参考手册：[MySQL 5.7 Reference Manual-Installing MySQL on Unix/Linux Using Generic Binaries](https://dev.mysql.com/doc/refman/5.7/en/binary-installation.html)



- （1）卸载操作系统自带的 mariadb/mysql 数据库

```shell
[root@localhost ~]# rpm -qa | grep mariadb | xargs yum remove -y
```

- （2）创建数据目录

```shell
[root@localhost ~]# mkdir -p /install /backup /data
```

- （3）创建mysql用户和用户组

```shell
[root@localhost ~]# useradd mysql
```

- （4）下载安装过程中可能需要的依赖

```shell
[root@localhost ~]# yum install -y libaio autoconf
```

- （5）新建/编辑 `/etc/my.cnf` 文件（需要先配置好这个文件，否则，安装后启动报错）

`vim /etc/my.cnf`

```shell
[client]
# 所有客户端登录进去后可以配置的参数

[mysql]
# mysql这个命令登录进去后配置的参数


[mysqld]
# mysql服务器启动时候的参数
port = 3306
user = mysql
basedir = /usr/local/mysql
datadir = /data
```

- （6）上传mysql5.7的安装包，并解压到 `/usr/local/` 目录下，创建软连接（便于管理mysql）

```shell
[root@localhost install]# tar -zxvf mysql-5.7.39-linux-glibc2.12-x86_64.tar.gz -C /usr/local/

# 校验下载的数据库压缩包MD5值，和官网进行比较，辨别是否被篡改过
[root@localhost install]# md5sum mysql-5.7.39-linux-glibc2.12-x86_64.tar.gz

[root@localhost local]# cd /usr/local/
[root@localhost local]# ln -s mysql-5.7.39-linux-glibc2.12-x86_64/ mysql
```

- （7）设置目录权限

```shell
[root@localhost local]# chown -R mysql.mysql /data/
[root@localhost local]# chmod 750 /data/
```

- （8）初始化mysql

```shell
[root@localhost local]# cd /usr/local/mysql

# mysql初始化数据库的语句，5.6 和 5.7 是不一样的，需要注意
[root@localhost mysql]# bin/mysqld --initialize --user=mysql

[root@localhost mysql]# bin/mysqld_safe --user=mysql &
# 拷贝启动脚本（服务）到指定位置，便于管理、使用
[root@localhost mysql]# cp support-files/mysql.server /etc/init.d/mysql.server
```

<br>

<br>

## MySQL安装成功后进行基础配置

- （1）设置MySQL开机自启动

```bash
# 把 mysql 的服务加入开机自启动项目
[root@localhost ~]# chkconfig --add mysql.server
[root@localhost ~]# chkconfig --list
```

- （2）配置MySQL环境变量

`vim /etc/profile`

```bash
export PATH=/usr/local/mysql/bin:$PATH
```

`source /etc/profile`

- （3）添加日志文件配置

`vim /etc/my.cnf`

```mysql
[mysqld]
log_error = error.log
```

- （4）修改初始密码

> 5.7

```mysql
mysql> set password='root';
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

> 5.6

```mysql
mysql> set password=password("root");
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

<br>

<br>

## MySQL的其它配置

### 免密登录

`vim /etc/my.cnf`

```mysql
[client]
#  mysql这个命令每次启动的时候，会去读配置文件，读到[client]标签，会自动把用户名和密码添加到后面。所以输入 mysql 后回车便可直接登录
user = root
password = root
```

<br>

### 定制MySQL终端提示符

`vim /etc/my.cnf`

```mysql
[mysql]
prompt = (\u@\h) [\d]>\_
```

<br>

### 允许MySQL远程登录

```shell
(root@localhost) [(none)]> grant all on *.* to root@'%' identified by 'root' with grant option;
Query OK, 0 rows affected (0.01 sec)

(root@localhost) [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

<br>

<br>

## MySQL 5.6与5.7 安装上的差异

- （1）5.6 安装完没有密码，而5.7初始化完成会有密码

  > 如果希望MySQL5.7安装初始化的时候表现跟MySQL5.6一样-没有密码，则可以在初始化数据库的语句中	`bin/mysqld --initialize --user=mysql`，把	`--initialize`	替换成	`--initialize-insecure`。（可以通过	 `mysqld --verbose --help | less	`查看到两个参数的说明）




- （2）初始化命令不一样
  - 5.6：`scripts/mysql_install_db --user=mysql`
  - 5.7：`bin/mysqld --initialize --user=mysql`




- （3）设置密码的方式不一样
  - 5.6：`set password = password("mysql");`
  - 5.7：`set password='mysql';`

<br>

<br>

## 查看MySQL 安装后的默认参数

```mysql
mysqld --verbose --help
```

<br>

<br>

## 作业

> 熟练安装 mysql 5.6、5.7
>
> mysql 5.6、5.7 自动安装的脚本。

<br>

<br>

## MySQL 5.6 升级到 5.7







# 第03天_MySQL客户端连接与权限管理

##  mysql最优配置文件参考

[ mysql_best_configuration](https://github.com/jdaaaaaavid/mysql_best_configuration/blob/master/my.cnf)

<br>

<br>

## MySQL可以有多个配置文件

```mysql
# mysql 在启动的时候会依次去读取下面的文件
[root@localhost ~]# mysql --verbose --help | grep my.cnf
                      order of preference, my.cnf, $MYSQL_TCP_PORT,
/etc/my.cnf /etc/mysql/my.cnf /usr/local/mysql/etc/my.cnf ~/.my.cnf
```

- 遵循参数替换原则。后面的参数会替换前面的参数

<br>

<br>

## MySQL的配置文件参数

```mysql
[client]
# 所有客户端登录进去后可以配置的参数

[mysql]
# mysql这个命令登录进去后配置的参数

[mysqld]
# mysql服务器启动时候的参数
port = 3306
user = mysql
datadir = /data
log_error = error.log
```

- 查看mysql的参数，当前配置的值

```mysql
# 默认省略了session
show variables;

show session variables;
```

```mysql
show variables like 'log_error';

show global variables like '%log%';
```

```mysql
# 对已经创建的连接不生效，只对新创建的连接生效
set global long_query_time=5;
```



|                             命令                             |             说明             |
| :----------------------------------------------------------: | :--------------------------: |
| SHOW [GLOBAL \|SESSION] VARIABLES [LIKE  'pattern' \|WHERE expr] | 查看变量，可根据like进行过滤 |
|            SET [GLOBAL\|SESSION] VARIABLES = xxx             |  修改global或session的参数   |



## MySQL的参数分类

- 从作用域上可分为global和session
  - seesion：只对当前这一个连接有效
  - global：全局有效
- 从类型上又可分为可修改和只读参数
  - 用户可在线修改非只读参数
  - 只读参数只能通过配置文件修改并重启
  - 所有参数的修改都不持久化

### 当前会话下，查看其它连接的 variables

- mysql5.7开始，多了一个数据库	`performance_schema`

```mysql
use performance_schema;


show tables like '%variables%';


(root@localhost) [performance_schema]> select * from variables_by_thread where variable_name = 'long_query_time';
+-----------+-----------------+----------------+
| THREAD_ID | VARIABLE_NAME   | VARIABLE_VALUE |
+-----------+-----------------+----------------+
|        31 | long_query_time | 10.000000      |
|        33 | long_query_time | 10.000000      |
+-----------+-----------------+----------------+
2 rows in set (0.00 sec)


# 查看当前连接到mysql的线程列表，以及它在做什么事情；
(root@localhost) [performance_schema]> show processlist;
+----+------+-----------+--------------------+---------+------+----------+------------------+
| Id | User | Host      | db                 | Command | Time | State    | Info             |
+----+------+-----------+--------------------+---------+------+----------+------------------+
|  6 | root | localhost | performance_schema | Sleep   |  766 |          | NULL             |
|  8 | root | localhost | performance_schema | Query   |    0 | starting | show processlist |
+----+------+-----------+--------------------+---------+------+----------+------------------+
2 rows in set (0.00 sec)


# 查看当前连接的id，这个id在processlist 里面
(root@localhost) [performance_schema]> select connection_id();
+-----------------+
| connection_id() |
+-----------------+
|               8 |
+-----------------+
1 row in set (0.00 sec)



# 这张表里 有 thread_id 和 processlist_id，通过这张表，做关联查询，则可以查询其它会话的变量值
(root@localhost) [performance_schema]> select * from threads limit 1\G
*************************** 1. row ***************************
          THREAD_ID: 1
               NAME: thread/sql/main
               TYPE: BACKGROUND
     PROCESSLIST_ID: NULL
   PROCESSLIST_USER: NULL
   PROCESSLIST_HOST: NULL
     PROCESSLIST_DB: NULL
PROCESSLIST_COMMAND: NULL
   PROCESSLIST_TIME: 1967
  PROCESSLIST_STATE: NULL
   PROCESSLIST_INFO: NULL
   PARENT_THREAD_ID: NULL
               ROLE: NULL
       INSTRUMENTED: YES
            HISTORY: YES
    CONNECTION_TYPE: NULL
       THREAD_OS_ID: 3562
1 row in set (0.00 sec)
```

<br>

<br>

## 权限管理

- mysql的权限认证分成：用户名、密码、IP

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/1652975371(1).66aoeywjsa80.webp" />

- mysql的权限分成：全局（所有库）、指定库、指定表、指定列
- 常用的权限
  - SQL语句：SELECT、INSERT、UPDATE、DELETE、INDEX
  - 存储过程：CREATE ROUTINE、ALTER ROUTINE、EXECUTE、TRIGGER
  - 管理权限：SUPER、RELOAD、SHOW DATABASE、SHUTDOWN、GRANT OPTION

- 常用权限、用户相关sql语句

```mysql
create user 'sao'@'192.168.%.%' identified by 'sao';

drop user 'sao'@'192.168.%.%';

#显示当前用户的权限 这三个是同一个意思
show grants;
show grants for current_user;
show grants for current_user();

grant select,update,delete,insert on mysql.* to 'sao'@'192.168.%.%';

show grants for 'sao'@'192.168.%.%';


# 不推荐 使用grant 同时创建权限和用户。
(root@localhost) [(none)]> grant select,update,delete,insert on mysql.* to 'sa'@'192.168.%.%' identified by 'sa';
Query OK, 0 rows affected, 1 warning (0.00 sec)

(root@localhost) [(none)]> show warnings;
+---------+------+------------------------------------------------------------------------------------------------------------------------------------+
| Level   | Code | Message                                                                                                                            |
+---------+------+------------------------------------------------------------------------------------------------------------------------------------+
| Warning | 1287 | Using GRANT for creating new user is deprecated and will be removed in future release. Create new user with CREATE USER statement. |
+---------+------+------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)


alter user 'sao'@'192.168.%.%' identified by 'saosao';

revoke select on mysql.* from 'sao'@'192.168.%.%';

grant create,index on mysql.* to 'sao'@'192.168.%.%' with grant option;
```

```mysql
# mysql 下的 mysql库中的权限表
# user表中存放全局权限
user
db
tables_priv 
columns_priv
```

<font color="red">**注意：不要通过修改权限表来更改用户权限**</font>

```mysql
# mysql 5.7 密码加密使用的是 password()这个函数
select password('456');
```

## 资源管理

- MAX_QUERIES_PER_HOUR *count*
- MAX_UPDATES_PER_HOUR *count*
- MAX_CONNECTIONS_PER_HOUR *count*
- MAX_USER_CONNECTIONS *count*

```mysql
# 修改用户的最大连接数
alter user 'sao'@'192.168.%.%' with MAX_USER_CONNECTIONS 2;
```

<br>

<br>

<br>

# 第04天_SSL加密连接与密码插件

## 连接MySQL实例

### 通过本地*socket*进行连接

```mysql
# mysql服务器本地 ，默认使用socket方式连接。 默认省略 -S/tmp/mysql.socket
mysql -uroot -p -S/tmp/mysql.socket
```

```mysql
(root@localhost) [(none)]> show variables like '%socket%'
    -> ;
+-----------------------------------------+-----------------+
| Variable_name                           | Value           |
+-----------------------------------------+-----------------+
| performance_schema_max_socket_classes   | 10              |
| performance_schema_max_socket_instances | -1              |
| socket                                  | /tmp/mysql.sock |
+-----------------------------------------+-----------------+
3 rows in set (0.01 sec)
```

- 查看当前连接的状态

```mysql

(root@localhost) [mysql]> \s
--------------
mysql  Ver 14.14 Distrib 5.7.37, for linux-glibc2.12 (x86_64) using  EditLine wrapper

Connection id:          2
Current database:       mysql
Current user:           root@localhost
SSL:                    Not in use
Current pager:          stdout
Using outfile:          ''
Using delimiter:        ;
Server version:         5.7.37 MySQL Community Server (GPL)
Protocol version:       10
Connection:             Localhost via UNIX socket
Server characterset:    latin1
Db     characterset:    latin1
Client characterset:    utf8
Conn.  characterset:    utf8
UNIX socket:            /tmp/mysql.sock
Uptime:                 35 sec

Threads: 1  Questions: 41  Slow queries: 0  Opens: 136  Flush tables: 1  Open tables: 129  Queries per second avg: 1.171
--------------

(root@localhost) [mysql]> status;
--------------
mysql  Ver 14.14 Distrib 5.7.37, for linux-glibc2.12 (x86_64) using  EditLine wrapper

Connection id:          2
Current database:       mysql
Current user:           root@localhost
SSL:                    Not in use
Current pager:          stdout
Using outfile:          ''
Using delimiter:        ;
Server version:         5.7.37 MySQL Community Server (GPL)
Protocol version:       10
Connection:             Localhost via UNIX socket
Server characterset:    latin1
Db     characterset:    latin1
Client characterset:    utf8
Conn.  characterset:    utf8
UNIX socket:            /tmp/mysql.sock
Uptime:                 39 sec

Threads: 1  Questions: 44  Slow queries: 0  Opens: 136  Flush tables: 1  Open tables: 129  Queries per second avg: 1.128
--------------

(root@localhost) [mysql]>

```



<br/>

### 通过TCP/IP协议远程连接

```mysql
mysql -uroot -h192.168.163.200 -P3306 -p
```

### SSL

- mysql 5.7开始 客户端默认使用ssl通信
  - 只有通过 tcp/ip 的连接的方式 ，ssl才生效，socket方式不会生效

```mysql
# 开启ssl后，默认使用none
mysql -uroot -h192.168.163.200 -P3306 -p --ssl-mode=none

mysql -uroot -h192.168.163.200 -P3306 -p --ssl-mode=DISABLED

alter user 'any'@'%' require ssl;

```

- ssl 加密 对性能有影响（影响较小）
- ssl的另一种方式 x509
  - 要求：用户名 + 密码 + 启用ssl+公钥 

## 作业

> 了解5.6如何使用ssl

## 密码插件

```mysql
show plugins;
```

- mysql 5.6 开始就有了密码插件 `validate_password`

### validate_password

- `validate_password`插件安装方式有以下两种

```mysql
# （1）在my.cnf文件中添加如下配置，再重启mysql
[mysqld]
plugin-load-add=validate_password.so

# （2）执行如下命令 
INSTALL PLUGIN validate_password SONAME 'validate_password.so';
```

```mysql
# 安装完成插件后，对密码就有要求了
(root@localhost) [mysql]> alter user 'andy'@'%' identified by 'andy';
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements


(root@localhost) [(none)]> show variables like '%validate%';
+--------------------------------------+--------+
| Variable_name                        | Value  |
+--------------------------------------+--------+
| query_cache_wlock_invalidate         | OFF    |
| validate_password_check_user_name    | OFF    |
| validate_password_dictionary_file    |        |
| validate_password_length             | 8      |
| validate_password_mixed_case_count   | 1      |
| validate_password_number_count       | 1      |
| validate_password_policy             | MEDIUM |
| validate_password_special_char_count | 1      |
+--------------------------------------+--------+
8 rows in set (0.00 sec)
```

- 建议生产环境使用该插件
  - 如果使用该插件，则需要在配置文件中做如下配置 

```mysql
# 默认mysql启动就使用该插件
[mysqld]
plugin-load = validate_password.so
```

| Policy          | Tests Performed                                              |
| :-------------- | :----------------------------------------------------------- |
| `0` or `LOW`    | Length                                                       |
| `1` or `MEDIUM` | Length; numeric, lowercase/uppercase, and special characters |
| `2` or `STRONG` | Length; numeric, lowercase/uppercase, and special characters; dictionary file |

- Strong中有一个 dictionary file 。该dictionary file 表示，设置的密码中不能包含 dictionary file 文件中的字符串（即dictionary file 中的字符串不能作为密码的一部分）。如果要启动这个策略，那么  `validate_password_policy ` 要设置为 `Strong`

```mysql
(root@localhost) [(none)]> set global validate_password_dictionary_file = '/mysql_data/dic.file';
Query OK, 0 rows affected (0.00 sec)

(root@localhost) [(none)]> exit
Bye
[root@localhost mysql_data]# vim /mysql_data/dic.file
[root@localhost mysql_data]# ll /mysql_data/
总用量 123008
-rw-r-----. 1 mysql mysql       56 5月  19 00:45 auto.cnf
-rw-r-----. 1 mysql mysql        5 5月  20 00:06 bogon.pid
-rw-------. 1 mysql mysql     1680 5月  19 00:45 ca-key.pem
-rw-r--r--. 1 mysql mysql     1112 5月  19 00:45 ca.pem
-rw-r--r--. 1 mysql mysql     1112 5月  19 00:45 client-cert.pem
-rw-------. 1 mysql mysql     1676 5月  19 00:45 client-key.pem
-rw-r--r--. 1 root  root        13 5月  21 21:51 dic.file
-rw-r-----. 1 mysql mysql    47519 5月  21 21:50 error.log
-rw-r-----. 1 mysql mysql      333 5月  21 21:40 ib_buffer_pool
-rw-r-----. 1 mysql mysql 12582912 5月  21 21:40 ibdata1
-rw-r-----. 1 mysql mysql 50331648 5月  21 21:40 ib_logfile0
-rw-r-----. 1 mysql mysql 50331648 5月  19 00:45 ib_logfile1
-rw-r-----. 1 mysql mysql 12582912 5月  21 21:40 ibtmp1
-rw-r-----. 1 mysql mysql        5 5月  21 21:40 localhost.pid
drwxr-x---. 2 mysql mysql     4096 5月  19 00:45 mysql
drwxr-x---. 2 mysql mysql     8192 5月  19 00:45 performance_schema
-rw-------. 1 mysql mysql     1680 5月  19 00:45 private_key.pem
-rw-r--r--. 1 mysql mysql      452 5月  19 00:45 public_key.pem
-rw-r--r--. 1 mysql mysql     1112 5月  19 00:45 server-cert.pem
-rw-------. 1 mysql mysql     1680 5月  19 00:45 server-key.pem
drwxr-x---. 2 mysql mysql     8192 5月  19 00:45 sys
[root@localhost mysql_data]# chown -R mysql:mysql dic.file

```

- `validate_password_check_user_name ` 这个开启后，表示密码中不能使用用户名，逆序也不行
- 作为了解。可以看看大厂实际生产环境，如何使用密码的。

## 多实例安装

- 一台服务器上安装多个实例，充分利用硬件资源
- mysql 原生就支持多实例安装
- 通过 `mysqld_multi` 实现

### 多实例安装操作流程

- 在配置文件 `my.cnf` 中新增加如下内容

```mysql
[mysqld1]
port = 3307
user = mysql
datadir = /mysql_data1
socket = /tmp/mysql1.sock
```

- 执行 `mysqld --initialize --datadir=/mysql_data1` 会发现生成了 `/mysql_data1/`目录，初始密码在 `error.log`中

- 在配置文件 `my.cnf` 中新增 `mysqld_multi`，并增加如下内容

```mysql
[mysqld_multi]
mysqld = /usr/local/mysql/bin/mysqld_safe
mysqladmin = /usr/local/mysql/bin/mysqladmin
log = /usr/local/mysql/mysqld_multi.log
```

- 执行 ` mysqld_multi report`

```mysql
[root@localhost mysql]# mysqld_multi report
Reporting MySQL servers
MySQL server from group: mysqld1 is not running
```

- 查看运行状态与启停、登录

```mysql
[root@localhost mysql]# mysqld_multi start 1

[root@localhost mysql]# mysqld_multi report
Reporting MySQL servers
MySQL server from group: mysqld1 is running

[root@localhost mysql_data1]# mysql -uroot -p'huah5jqpdJ?+' -S/tmp/mysql1.sock
```

```linux
# 可以查看到3307 端口已经被占用
[root@localhost mysql_data1]# netstat -ntl
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN
tcp6       0      0 :::22                   :::*                    LISTEN
tcp6       0      0 :::3306                 :::*                    LISTEN
tcp6       0      0 :::3307                 :::*                    LISTEN
[root@localhost mysql_data1]#
```

- 注意：
  - 最开始安装的 mysql 实例 也可以使用 mysql_multi start 进行启动，配置好便可
  - 默认 error.log 在各自的mysql data 目录中，所以不需要配置

<br/>

# 第05天_MySQL启动与关闭 & 多实例安装不同版本数据库

## MySQL 的启动与关闭

```mysql
/etc/init.d/mysql.server start


[root@localhost bin]# mysqld --defaults-file=/etc/my.cnf

# 守护进程方式启动mysql，当mysql进程挂了以后，会自动把mysql进程拉起来
[root@localhost bin]# mysqld_safe --user=mysql &
```

```mysql
# 需要密码
(root@localhost) [(none)]> shutdown;
Query OK, 0 rows affected (0.00 sec)

# 不提供用户名和密码
/etc/init.d/mysql.server stop



# 需要密码
[root@localhost bin]# mysqladmin -uroot -proot shutdown

```

- 为什么mysql不需要密码就可以关闭呢？

```mysql
# 是安全关闭的
/etc/init.d/mysql.server stop

# 是一个shell执行脚本
[root@localhost bin]# file /etc/init.d/mysql.server
/etc/init.d/mysql.server: POSIX shell script, ASCII text executable

# mysql利用了linux中的信号机制 ,查看 /etc/init.d/mysql.server 脚本，可以看到 kill -0 [进程号]

     if (kill -0 $mysqld_pid 2>/dev/null)
      then
        echo $echo_n "Shutting down MySQL"
        kill $mysqld_pid

# kill -0 [进程号] 表示 发送一个信号给这个进程。 然后 kill 掉这个进程，所以其是安全关闭的
```

```mysql

[mysqld_multi]
# 使用哪个程序来启动mysql
mysqld = /usr/local/mysql/bin/mysqld_safe
# 通过mysqladmin 这个命令来关闭数据库
mysqladmin = /usr/local/mysql/bin/mysqladmin
log = /usr/local/mysql/mysqld_multi.log
user=root
pass=Root1_2022
```

<br/>

## MySQL 忘记密码了怎么办

```mysql
# 编辑 mysql配置文件 my.cnf，添加如下内容

[mysqld]
skip-grant-tables

# 然后重启数据库

# 此时登录mysql不需要密码,use mysql 然后修改user表的密码字段即可
update mysql set authentication_string = password('root') where user='root' and host = 'localhost';

(root@localhost) [mysql]> flush privileges;

# 再去掉参数，重启数据库即可
```

## MySQL5.7 的新参数

- `default_password_lifetime`

```mysql
ALTER USER 'jeffrey'@'localhost' PASSWORD EXPIRE;
```

```mysql
[mysqld]
default_password_lifetime=180

[mysqld]
default_password_lifetime=0

SET GLOBAL default_password_lifetime = 180;
SET GLOBAL default_password_lifetime = 0;
```









# 第07天_数据类型、常用函数、编码













# 学习备注

> 1. 第一天的内容，最后还需要认真学习一遍，做好笔记
> 2. 权限管理这块，还需要实践，继续熟悉



<br>

<img src="" width="80%"/>

<br>



































































































































































































































# 备注

> 熟悉mysql官网
>
> hexo new post "MySQL - 简介、版本特性与选择、安装与初始化配置、升级、启停"
