---
title: KingbaseES KCA study notes
auth: ratears
date: 2022-09-03 17:19:20
update: 2022-09-03 17:19:20
categories:
	- [database,kingbase]
tags:
	- kingbase
	- database
---



# Kingbase安装与卸载

## KES安装

### 环境要求

|        环境         |                      要求                      |
| :-----------------: | :--------------------------------------------: |
|      操作系统       |                   CentOS 7.x                   |
| 机器规格 - 内存大小 |                   3GB 及以上                   |
| 机器规格 - 磁盘空间 |                  20GB 及以上                   |
|   KES 安装包版本    | KingbaseES_V008R006C006B0013_Lin64_install.iso |
|      jdk 版本       |                      1.8+                      |



### 安装前准备工作

1. 服务器安装jdk1.8+版本并配置环境变量

```shell
[root@localhost ~]# yum list |grep jdk

[root@localhost ~]# yum -y install java-1.8.0-openjdk

[root@localhost ~]# java -version
openjdk version "1.8.0_342"
OpenJDK Runtime Environment (build 1.8.0_342-b07)
OpenJDK 64-Bit Server VM (build 25.342-b07, mixed mode)
```



2. 创建用于管理Kingbase的用户

```shell
[root@localhost ~]# useradd kingbase
[root@localhost ~]# 
[root@localhost ~]# id kingbase
uid=1000(kingbase) gid=1000(kingbase) 组=1000(kingbase)

[root@localhost install]# passwd kingbase
```



3. 按实施规范创建目录，设置权限

- 为了利于数据库的日常运维、持续使用、存储扩容等，我们在安装前须做好选项、存储目录规划

|      选项      |                             设置                             |
| :------------: | :----------------------------------------------------------: |
|      目录      | 安装软件存储目录：/install<br/>备份目录：/backup<br/>归档目录：/archive<br/>数据存储目录：/data<br/>KES 安装目录：/KingbaseES/V8 |
|      端口      |                            54321                             |
|  SYSTEM 密码   |                            system                            |
| 数据库编码格式 |                             UTF8                             |
| 大小写是否敏感 |    ENABLE_CI，默认为 off，表示大小写敏感（根据需求选择）     |



```shell
# 使用 root 用户，创建目录，设置权限

# 创建目录
[root@localhost ~]# mkdir -p /install /KingbaseES/V8 /backup /data /archive

# 修改目录属组、属主、权限
[root@localhost ~]# chown -R kingbase:kingbase /install /KingbaseES /backup /archive /data
[root@localhost ~]# chmod -R 775 /install /KingbaseES /backup /archive
[root@localhost ~]# chmod -R 700 /data


[root@localhost ~]# ls -l / |grep kingbase
drwxrwxr-x.   2 kingbase kingbase    6 9月   4 22:39 archive
drwxrwxr-x.   2 kingbase kingbase    6 9月   4 22:39 backup
drwx------.   2 kingbase kingbase    6 9月   4 22:39 data
drwxrwxr-x.   2 kingbase kingbase    6 9月   4 22:39 install
drwxrwxr-x.   3 kingbase kingbase   16 9月   4 22:39 KingbaseES
```

<br>

### 上传安装包、授权文件、检查 md5、解压

```shell
# 使用 root 用户将安装文件上传到/install 下
[root@localhost install]# ll
-rw-r--r--. 1 root root 2180431872 9月   4 22:56 KingbaseES_V008R006C006B0013_Lin64_install.iso

# 使用 root 用户将授权文件上传到/install。设置授权文件的属主为kingbase和权限并验证
[root@localhost install]# ll
-rw-rw-r--. 1 kingbase kingbase       3534 4月  26 11:51 license_18720_0.dat

# 检查和校验安装文件的 md5 值
[root@localhost install]# md5sum KingbaseES_V008R006C006B0013_Lin64_install.iso
c1410ba7062fbaff3308c1453797ce3e  KingbaseES_V008R006C006B0013_Lin64_install.iso

# 使用 root 用户挂载 KES 安装虚拟光盘文件
[root@localhost install]# mount -o loop /install/KingbaseES_V008R006C006B0013_Lin64_install.iso /mnt/
mount: /dev/loop0 写保护，将以只读方式挂载
[root@localhost install]# ll /mnt/
总用量 6
dr-xr-xr-x. 2 root root 2048 6月  14 01:09 setup
-r-xr-xr-x. 1 root root 3820 6月  14 01:09 setup.sh

# 使用 kingbase 用户复制挂载后的安装文件到/install 下
[kingbase@bogon ~]$ ll /mnt/
总用量 6
dr-xr-xr-x. 2 root root 2048 6月  14 01:09 setup
-r-xr-xr-x. 1 root root 3820 6月  14 01:09 setup.sh
[kingbase@bogon ~]$ mkdir -p /install/KES_V8R6CB13_INSTALL
[kingbase@bogon ~]$ cp -r /mnt/* /install/KES_V8R6CB13_INSTALL/
[kingbase@bogon ~]$ ll /install/KES_V8R6CB13_INSTALL/
总用量 4
dr-xr-xr-x. 2 kingbase kingbase   54 9月   4 23:14 setup
-r-xr-xr-x. 1 kingbase kingbase 3820 9月   4 23:14 setup.sh

[kingbase@bogon ~]$ du -sm /mnt/
2080    /mnt/
[kingbase@bogon ~]$ du -sm /install/KES_V8R6CB13_INSTALL/
2080    /install/KES_V8R6CB13_INSTALL/
```

<br>

### 字符界面安装 KES 程序

#### 启动字符界面安装向导进入“简介”界面

```shell
[kingbase@bogon ~]$ cd /install/KES_V8R6CB13_INSTALL/

[kingbase@bogon KES_V8R6CB13_INSTALL]$ bash ./setup.sh -i console

# 安装目录输入: /KingbaseES/V8
```



####  创建和初始化数据库集簇

```shell
# 数据存储目录输入: /data

# 直接回车接受参数的默认值或根据规划要求指定参数的值：
（1）端口：54321（生产环境可以根据需求，自定义为其它端口）。
（2）输入 SYSTEM 超级管理员的密码（本实验环境密码设置为 system）。
（3）设置数据库编码格式（推荐设置为“UTF8”）。
（4）选择数据库模式（默认为 oracle 模式）
```



#### 将 KES 服务注册为 linux 系统服务

```shell
# 使用 root 用户执行安装程序提供的脚本/KingbaseES/V8/Scripts/root.sh

[root@localhost ~]# /KingbaseES/V8/install/script/root.sh
Starting KingbaseES V8:
waiting for server to start.... done
server started
KingbaseES V8 started successfully
```



####  重启 linux 确认 KES 服务自动启动

```shell
[kingbase@bogon ~]$ ps -xf |grep -v grep |grep -i 'kingbase'
  2116 ?        S      0:00 sshd: kingbase@notty
  2092 ?        S      0:00 sshd: kingbase@pts/1
  4919 ?        Ss     0:00 /KingbaseES/V8/KESRealPro/V008R006C006B0013/Server/bin/kingbase -D /data
  4920 ?        Ss     0:00  \_ kingbase: logger
  4922 ?        Ss     0:00  \_ kingbase: checkpointer
  4923 ?        Ss     0:00  \_ kingbase: background writer
  4924 ?        Ss     0:00  \_ kingbase: walwriter
  4925 ?        Ss     0:00  \_ kingbase: autovacuum launcher
  4926 ?        Ss     0:00  \_ kingbase: stats collector
  4927 ?        Ss     0:00  \_ kingbase: ksh writer
  4928 ?        Ss     0:00  \_ kingbase: ksh collector
  4929 ?        Ss     0:00  \_ kingbase: kwr collector
  4930 ?        Ss     0:00  \_ kingbase: logical replication launcher
```

<br>

## 确认 KES 是否已正确安装

- 可以使用以下几个角度确认 KES 是否已正确安装或启动

1. 查看安装过程日志，确认没有错误记录

```shell
[kingbase@bogon ~]$ cd /install/KES_V8R6CB13_INSTALL/

[kingbase@bogon KES_V8R6CB13_INSTALL]$ ls -al *.log
ls: 无法访问*.log: 没有那个文件或目录
```



2. 查看开始菜单中是否已成功安装相关程序



3. 查看相关进程是否启动

```shell
[kingbase@bogon ~]$ ps -xf |grep -v grep |grep -i 'kingbase'
  2116 ?        S      0:00 sshd: kingbase@notty
  2092 ?        S      0:00 sshd: kingbase@pts/1
  4919 ?        Ss     0:00 /KingbaseES/V8/KESRealPro/V008R006C006B0013/Server/bin/kingbase -D /data
  4920 ?        Ss     0:00  \_ kingbase: logger
  4922 ?        Ss     0:00  \_ kingbase: checkpointer
  4923 ?        Ss     0:00  \_ kingbase: background writer
  4924 ?        Ss     0:00  \_ kingbase: walwriter
  4925 ?        Ss     0:00  \_ kingbase: autovacuum launcher
  4926 ?        Ss     0:00  \_ kingbase: stats collector
  4927 ?        Ss     0:00  \_ kingbase: ksh writer
  4928 ?        Ss     0:00  \_ kingbase: ksh collector
  4929 ?        Ss     0:00  \_ kingbase: kwr collector
  4930 ?        Ss     0:00  \_ kingbase: logical replication launcher
```



4. 验证数据库连接是否正常

```shell
# 用 ksql 工具测试能否连接到数据库
[kingbase@bogon ~]$ /KingbaseES/V8/Server/bin/ksql test system
ksql (V8.0)
输入 "help" 来获取帮助信息.

test=#

# 在桌面环境中启动“数据库对象管理工具”测试能否连接数据库
```



5. 查看服务是否已设为开机自启

```shell
[kingbase@bogon ~]$ systemctl list-dependencies |grep kingbase
● ├─kingbase8d.service
[kingbase@bogon ~]$ chkconfig --list |grep kingbase

注：该输出结果只显示 SysV 服务，并不包含
原生 systemd 服务。SysV 配置数据
可能被原生 systemd 配置覆盖。

      要列出 systemd 服务，请执行 'systemctl list-unit-files'。
      查看在具体 target 启用的服务请执行
      'systemctl list-dependencies [target]'。

kingbase8d      0:关    1:关    2:开    3:开    4:开    5:开    6:关
```

<br>

## 启停 KES 服务

### root用户管理KES服务

- 以 root 用户身份登录

#### root 用户将 KES 注册为 linux 开机自启服务

```shell
# 使用 root 用户执行安装程序提供的脚本/KingbaseES/V8/Scripts/root.sh 来注册系统服务，这样开机时会自己启动 KES 数据库服务

[root@localhost ~]# /KingbaseES/V8/install/script/root.sh
Starting KingbaseES V8:
waiting for server to start.... done
server started
KingbaseES V8 started successfully
```

<br>

#### systemctl 管理 KES 服务

```shell
# root 用户使用 systemctl 管理 KES 服务

# 1、确认 KES 服务状态。
systemctl status kingbase8d.service

# 2、停止 KES 服务。
systemctl status kingbase8d.service

# 3、启动 KES 服务。
systemctl start kingbase8d.service

# 4、重启 KES 服务。
systemctl restart kingbase8d.service
```

<br>

#### service 管理 KES 服务

```shell
# root 用户使用 service 管理 KES 服务

service kingbase8d status

service kingbase8d start

service kingbase8d restart

service kingbase8d stop
```

<br>

### kingbase用户管理KES服务

- 使用 kingbase 用户登录后执行 sys_ctl 命令

#### 使用金仓 sys_ctl 命令管理 KES 服务

##### sys_ctl 长命令格式

```shell
/KingbaseES/V8/Server/bin/sys_ctl status -D /data

/KingbaseES/V8/Server/bin/sys_ctl stop -D /data

/KingbaseES/V8/Server/bin/sys_ctl start -D /data

/KingbaseES/V8/Server/bin/sys_ctl restart -D /data
```

<br>

##### sys_ctl 语法大纲

```shell
sys_ctl start 	[-D DATADIR] [-l FILENAME] [-W] [-t SECS] [-s]
				[-o OPTIONS] [-p PATH] [-c]
				
sys_ctl stop 	[-D DATADIR] [-m SHUTDOWN-MODE] [-W] [-t SECS] [-s]

sys_ctl restart [-D DATADIR] [-m SHUTDOWN-MODE] [-W] [-t SECS] [-s]
				[-o OPTIONS] [-c]
				
sys_ctl reload 	[-D DATADIR] [-s]

sys_ctl status 	[-D DATADIR]

sys_ctl promote [-D DATADIR] [-W] [-t SECS] [-s]

sys_ctl logrotate [-D DATADIR] [-s]

sys_ctl kill SIGNALNAME PID
```

<br>

#### 使用 kingbase 命令启动 KES 服务

```shell
/KingbaseES/V8/Server/bin/kingbase -D /data >log1 2>&1 &
```

<br>

## 环境变量对相关命令的影响

1. 定位金仓 sys_ctl 的路径

```shell
[kingbase@bogon bin]$ find /KingbaseES/ -name sys_ctl
/KingbaseES/V8/KESRealPro/V008R006C006B0013/Server/bin/sys_ctl
```



2. 定位主数据目录

```shell
[kingbase@bogon bin]$ ps -ef|grep '\ -D\ '
kingbase   6723   2093  0 00:27 pts/1    00:00:00 /KingbaseES/V8/Server/bin/kingbase -D /data

[root@localhost ~]# find / -name kingbase.conf
/data/kingbase.conf
```



3. 修改环境变量

（1）KINGBASE_DATA 变量

- 该环境变量指向 KES 主数据目录，此变量名是金仓程序员指定的命名，不要修改
- 未指定该变量时，sys_ctl 工具在执行时需要加-D 参数来给定主数据目录位置

（2）修改 shell 的 profile

- 把【/KingbaseES/V8/Server/bin】加到$PATH 变量里面
-  把/data 赋值给$KINGBASE_DATA

```shell
# 编辑 kingbase 的环境变量文件，添加相应路径
vi /home/kingbase/.bashrc

export PATH=/KingbaseES/V8/Server/bin:$PATH
export KINGBASE_DATA=/data

# 让环境变量生效
source /home/kingbase/.bashrc
```

```shell
# 设置变量后命令行比较简捷
sys_ctl stop -D $KINGBASE_DATA 
kingbase -D /data >log1 2>&1 &
```

<br>

## 授权文件license.dat

### 查看license有效天数

```shell
test=# select get_license_validdays();
 get_license_validdays
-----------------------
                    89
(1 行记录)

test=#
```

<br>

###　license过期后的故障现象

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.41abq8t29n80.webp" />

<br>

### license过期故障解决

- 使用 kingbase 用户登录，上传新的 lincense.dat 到 kingbase的安装目录 `/KingbaseES/V8`下——替换原旧的 license.dat。然后重新加载数据库（或者重新启动）

```shell
# 注意 lincese.dat 的 属组和属组 必须是 kingbase，授权文件名称 必须是 lincese.dat

[kingbase@bogon ~]$ sys_ctl reload -D $KINGBASE_DATA
server signaled
```

<br>

## KES卸载 

1. 使用 root 用户执行 rootuninstall.sh 脚本移除 KES 开机自启服务

```shell
su - root

/KingbaseES/V8/install/script/rootuninstall.sh
```



2. 以 kingbase 用户执行 bash /KingbaseES/V8/Uninstall/Uninstaller -i console

```shell
[kingbase@bogon ~]$ bash /KingbaseES/V8/Uninstall/Uninstaller -i console

# 卸载完后，执行`echo $?`的 结果为 0，则 Uninstaller -i console 执行成功
```



3. 清除安装目录下残留文件

- 方法一：执行 `rm –fr /KingbaseES/V8` 直接删除安装目录
- 方法二：执行 `mv /KingbaseES/V8 /KingbaseES/V8.bak` 将安装目录改名

<br>

# 数据库对象管理工具

- 参考培训文档，做实验，以及做好记录



<br>

# 命令行工具KSQL

## KSQL简介

- KSQL 是金仓提供给 DBA 的与 KES 数据库交互的命令行客户端程序（部分工作场景是无法使用图形界面工具来工作的；还有部分场景使用SQL效率更高）。熟练使用 KSQL 工具可以帮助 DBA 快速的操作和维护数据库

### 查看 KSQL 工具的帮助

```shell
[kingbase@bogon ~]$ ksql --help
```

- 部分参数解析

1. 连接参数

| 参数 |                             简介                             |
| :--: | :----------------------------------------------------------: |
|  -h  | 连接服务器的监听 IP 或主机名(-h 缺省时为 localhost 方式登录) |
|  -p  | 连接服务器的监听端口号<br>当为端口号为默认值 54321 时可缺省-p<br/>设置了 KINGBASE_PORT 环境变量时也可缺省-p |
|  -U  |                      连接服务器的用户名                      |
|  -W  |                         强制输入密码                         |



2. 通用参数

| 参数 |                             简介                             |
| :--: | :----------------------------------------------------------: |
|  -c  | 指定连接数据库后执行的单行命令，执行完成后自动退出数据库连接 |
|  -d  |                    指定连接时登录的数据库                    |
|  -f  |   指定连接数据库时执行的脚本，执行完成后自动退出数据库连接   |
|  -l  |                        打印数据库列表                        |
|  -V  |                      打印数据库版本信息                      |
|  -?  |                   打印 ksql 命令的帮助信息                   |



3. 输入输出参数

| 参数 |               简介               |
| :--: | :------------------------------: |
|  -H  |     以 html 格式展示输出结果     |
|  -E  |      展示元命令所执行的 sql      |
|  -t  |           不输出字段名           |
|  -x  |      调整查询结果为纵向展示      |
|  -q  |        不输出登录提示信息        |
|  -o  | 将命令输出结果保存到指定的文件中 |

<br>

### 查看标准 SQL 命令的帮助

1. 使用 `\h` 列出所有的 SQL 命令清单

```sql
test=# \h
```



2. 使用 `\h <sql 命令>` 列出某个 SQL 命令的语法大纲

```sql
test=# \h delete
Command:     DELETE
Description: 删除数据表中的数据列
Syntax:
[ WITH [ RECURSIVE ] with查询语句(with_query) [, ...] ]
DELETE FROM [ ONLY ] 表名 [ * ] [ [ AS ] 别名 ]
    [ USING USING列表(using_list) ]
    [ WHERE 条件 | WHERE CURRENT OF 游标名称 ]
    [ RETURNING * | 输出表达式 [ [ AS ] 输出名称 ] [, ...] ]
```

<br>

### 查看 KSQL 元命令的帮助

1. 元命令介绍

> （1）ksql 提供了一组以“\”开头的快捷命令，称之为 ksql 元命令。
> （2）搭配通配符“*”或“?”提高查询效率。
> （3）使用选项“S”显示系统对象。
> （4）使用选项“+”显示更加丰富的信息。



2. 常用元命令介绍

|      参数      |                             简介                             |
| :------------: | :----------------------------------------------------------: |
|     \d[S+]     | 列出表,视图和序列,其中 S 表示包含系统对象，+表示列出详细信息 |
|  \d[S+] 名称   |                  描述表，视图，序列，或索引                  |
| \db[+] [模式]  |                          列出表空间                          |
| \di[S+] [模式] |                           列出索引                           |
|   \dp [模式]   |           列出表，视图和序列的访问权限(\z 和相同)            |
| \ds[S+] [模式] |                           列出序列                           |
|     \du[+]     |                           列出角色                           |
|     \l[+]      |                       列出所有的数据库                       |



### KSQL 连接到数据库

#### 使用 SOCKET 方式登录数据库

```shell
[kingbase@localhost ~]$ ksql -U system -d test
ksql (V8.0)
输入 "help" 来获取帮助信息.

test=# 
```



#### 使用 TCP/IP 方式登录数据库

```shell
[kingbase@localhost ~]$ ksql -h 192.168.146.129 -p 54321 -U system -d test
用户 system 的口令：
ksql (V8.0)
输入 "help" 来获取帮助信息.

test=# 
```



#### 在 KSQL 中切换登录用户和数据库

```shell
[kingbase@bogon ~]$ ksql -Usystem -d test
ksql (V8.0)
输入 "help" 来获取帮助信息.

test=# \c tfsdb
您现在已经连接到数据库 "tfsdb",用户 "system".

tfsdb=# \c - tfsdb
您现在已经连接到数据库 "tfsdb",用户 "tfsdb".

tfsdb=# \c test system
您现在已经连接到数据库 "test",用户 "system".
```



#### KSQL 引用环境变量进行快速登录

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.3xiqo6d2q3g0.webp" width="60%"/>

<br>



### 执行 SQL 的几种方式

1. 交互方式执行 SQL

```shell
# 登录ksql
tfsdb=# select * from pa_user;
```



2. 非交互方式执行 SQL (单条 SQL 语句)

```shell
# 使用“-c”选项登录 tfsdb 数据库查看

[kingbase@bogon ~]$ ksql -U tfsdb -d tfsdb -c 'select * from pa_user;'
```



3. 非交互方式执行 SQL (成批的 SQL 语句、SQL文件)

```shell
[kingbase@bogon ~]$ ll
总用量 8
-rw-rw-r--  1 kingbase kingbase  23 9月   9 18:00 demo.sql
-rw-------. 1 kingbase kingbase 712 8月  20 19:13 restart.log

[kingbase@bogon ~]$ cat demo.sql
select * from pa_user;

[kingbase@bogon ~]$ ksql -U tfsdb -d tfsdb -f /home/kingbase/demo.sql

```



### KSQL 元命令介绍

- 略（查看官网）



### 使用元命令实现异构数据库数据交换

```shell
# 导出表数据库到 csv 文件
tfsdb=# \copy tfsdb.pa_user to /home/kingbase/pa_user.csv csv
COPY 16

# 查看 csv
tfsdb=# \! cat /home/kingbase/pa_user.csv

# 将 csv 文件导入数据库表中
tfsdb=# \copy tfsdb.pa_user from /home/kingbase/pa_user.csv csv
COPY 16

```



# 用户与角色

- 用户和角色是数据库管理的基础
- 本章主要介绍如何在 KES 数据库中创建用户和角色，以及利用“角色”对多个用户批量授权，使 KES 管理体系更加清晰、简单



## 数据库用户

### 用户管理概述

1. 数据库用户代表数据库的使用者
2. 应该为每个使用者创建用户
3. 尽量避免多人使用同一个数据库用户



### 用户管理

- 增删改查 略（参考官网）

- 当待删除用户是部分对象的拥有者时，因对象依赖会导致删除用户失败



## 数据库角色

### 角色的概念

1. 将一组具有相同权限的用户组织在一起，这一组具有相同权限的用户就称为角色（Role）
2. 角色在生产系统中一般被视作用户组，利用角色对用户执行批量授权



### 角色管理

- 增删改查 略（参考官网）
- 当待删除角色是部分对象的拥有者时，因对象依赖会导致删除角色失败
- 当待删除角色被显式授予对象权限时，因权限依赖会导致删除角色失败



# 对象的访问权限入门

- 数据库的表、索引、视图、图表、缺省值、规则、触发器、语法等等，在数据库中的一切，都称为数据库对象，对象分为如下两类：

1. 模式（SCHEMA）对象：可视为一个表的集合，可以理解为一个存储目录，包含视图、索引、数据类型、函数和操作符等
2. 非模式对象：其他的数据库对象，如数据库、表空间、用户、权限。



- 用户或角色访问模式对象或非模式对象的能力称之为对象权限







<br>

<br>

<br>

# 简单巡检

## 查看 KES 版本信息

```shell
# 使用 sys_ctl 查看版本
[kingbase@localhost ~]$ sys_ctl -V
sys_ctl (Kingbase) 12.1
[kingbase@localhost ~]$ sys_ctl --version
sys_ctl (Kingbase) 12.1

# 使用 version 函数查看版本
test=# select version();
                                                       version
----------------------------------------------------------------------------------------------------------------------
 KingbaseES V008R006C006B0013 on x86_64-pc-linux-gnu, compiled by gcc (GCC) 4.1.2 20080704 (Red Hat 4.1.2-46), 64-bit
(1 行记录)

```

<br>

<br>

## 查看 license 有效期

```shell
test=# select get_license_validdays();
 get_license_validdays
-----------------------
                    66
(1 行记录)
```

<br>

<br>

## 查看 KES 实例启动时间和运行时长

```shell
# 查看数据库实例启动时间
test=# select sys_postmaster_start_time();
   sys_postmaster_start_time
-------------------------------
 2022-09-25 08:04:56.261041+08
(1 行记录)

# 查看 KES 无故障运行时长
test=# select date_trunc('second',current_timestamp - sys_postmaster_start_time()) as uptime;
         uptime
------------------------
 2 days 16:28:15.000000
(1 行记录)

```

<br>

<br>

## 查看数据库列表

```shell
# 使用 ksql 的-l 参数或元命令\l
[kingbase@localhost ~]$ ksql -Usystem -d test -l
                                           数据库列表
    名称     |   拥有者    | 字元编码 | 校对规则 |    Ctype    |            存取权限
-------------+-------------+----------+----------+-------------+--------------------------------
 security    | system      | UTF8     | ci_x_icu | zh_CN.UTF-8 |
 template0   | system      | UTF8     | ci_x_icu | zh_CN.UTF-8 | =c/system                     +
             |             |          |          |             | system=CTc/system
 template1   | system      | UTF8     | ci_x_icu | zh_CN.UTF-8 | =c/system                     +
             |             |          |          |             | system=CTc/system
 test        | system      | UTF8     | ci_x_icu | zh_CN.UTF-8 |
 xjnxdb_test | xjnxdb_test | UTF8     | ci_x_icu | zh_CN.UTF-8 | =Tc/xjnxdb_test               +
             |             |          |          |             | xjnxdb_test=C*T*c*/xjnxdb_test
(5 行记录)

test=# \l
                                           数据库列表
    名称     |   拥有者    | 字元编码 | 校对规则 |    Ctype    |            存取权限
-------------+-------------+----------+----------+-------------+--------------------------------
 security    | system      | UTF8     | ci_x_icu | zh_CN.UTF-8 |
 template0   | system      | UTF8     | ci_x_icu | zh_CN.UTF-8 | =c/system                     +
             |             |          |          |             | system=CTc/system
 template1   | system      | UTF8     | ci_x_icu | zh_CN.UTF-8 | =c/system                     +
             |             |          |          |             | system=CTc/system
 test        | system      | UTF8     | ci_x_icu | zh_CN.UTF-8 |
 xjnxdb_test | xjnxdb_test | UTF8     | ci_x_icu | zh_CN.UTF-8 | =Tc/xjnxdb_test               +
             |             |          |          |             | xjnxdb_test=C*T*c*/xjnxdb_test
(5 行记录)

# 使用数据字典
test=# select datname from sys_database;
   datname
-------------
 test
 security
 template1
 template0
 xjnxdb_test
(5 行记录)
```

<br>

<br>

## 查看数据库占用的磁盘空间

```shell
# 统计当前数据库占用的磁盘空间
test=# select sys_database_size(current_database())/1024/1024 || 'MB' MB;
  MB
------
 12MB
(1 行记录)

# 统计所有数据库占用的磁盘空间总量
xjnxdb_test=# select (sum(sys_database_size(datname))/1024/1024) || 'MB' MB from sys_database;
           MB
------------------------
 234.1652364730834961MB
(1 行记录)
```

<br>

<br>

## 查看表和索引的大小

```shell
# 统计表的空间占用
xjnxdb_test=# select sys_relation_size('xjnxdb_test.pa_user')/1024 || 'KB' KB;
  KB
------
 48KB
(1 行记录)

xjnxdb_test=# select sys_size_pretty(sys_relation_size('xjnxdb_test.pa_user'));
 sys_size_pretty
-----------------
 48 kB
(1 行记录)

# 统计表和与表关联的索引占用空间总量
xjnxdb_test=# select sys_total_relation_size('xjnxdb_test.pa_user')/1024 || 'KB' KB;
  KB
------
 88KB
(1 行记录)

xjnxdb_test=# select sys_size_pretty(sys_total_relation_size('xjnxdb_test.pa_user'));
 sys_size_pretty
-----------------
 88 kB
(1 行记录)

# 统计表的记录数
xjnxdb_test=# select count(*) || ' rows' "rows" from xjnxdb_test.pa_user;
   rows
----------
 145 rows
(1 行记录)
```

<br>

<br>

## 查看时区和时间

```shell
# 查看最近一次加载参数文件的时间
xjnxdb_test=# select sys_conf_load_time();
      sys_conf_load_time
-------------------------------
 2022-09-25 08:04:55.822231+08
(1 行记录)

# 查看时区
xjnxdb_test=# show timezone;
   TimeZone
---------------
 Asia/Shanghai
(1 行记录)

# 查看当前日期或时间
xjnxdb_test=# select now();
              now
-------------------------------
 2022-09-28 05:43:22.416126+08
(1 行记录)

xjnxdb_test=# select current_timestamp;
       current_timestamp
-------------------------------
 2022-09-28 05:43:45.124372+08
(1 行记录)

xjnxdb_test=# select sysdate;
       sysdate
---------------------
 2022-09-28 05:44:16
(1 行记录)

xjnxdb_test=# select current_date;
 current_date
--------------
 2022-09-28
(1 行记录)

```

<br>

<br>

## 查看当前登录数据库的名称

```shell
xjnxdb_test=# select current_catalog;
 current_catalog
-----------------
 xjnxdb_test
(1 行记录)

xjnxdb_test=# select current_database();
 current_database
------------------
 xjnxdb_test
(1 行记录)
```

<br>

<br>

## 查看当前会话信息

```shell
# 查看当前会话的客户端 IP 和端口。
test=# select inet_client_addr(),inet_client_port();
 inet_client_addr | inet_client_port
------------------+------------------
 10.114.200.15    |            52665
(1 行记录)

# 查看服务器的 IP 和端口。
test=# select inet_server_addr(),inet_server_port();
 inet_server_addr | inet_server_port
------------------+------------------
 10.114.12.66     |            54321
(1 行记录)

# 查看当前会话的后台进程 ID。
test=# select sys_backend_pid();
 sys_backend_pid
-----------------
          341129
(1 行记录)
```

<br>

<br>

## 查看数据库中的连接信息

```shell
test=# select datname,usename,client_addr,client_port  from sys_stat_activity;
   datname    |   usename    |  client_addr   | client_port
--------------+--------------+----------------+-------------
              |              |                |
              | system       |                |
              | system       |                |
 xjnxdb_2_pt  | xjnxdb_2_pt  | 10.114.12.67   |       45560
 xjnxdb_2_pt  | xjnxdb_2_pt  | 10.114.12.67   |       45584
 xjnxdb_test2 | xjnxdb_test2 | 10.114.12.67   |       45624
 xjnxdb_2_pt  | xjnxdb_2_pt  | 10.114.12.67   |       45630
```

<br>

<br>

## 查看会话执行的 SQL 信息

```shell
# （1）设置参数 track_activities 为 on。
test=# show track_activities;
 track_activities
------------------
 on
(1 行记录)

# 查看所有会话执行的 SQL 信息
test=# select datname,usename,client_addr,client_port  from sys_stat_activity;
   datname    |   usename    |  client_addr   | client_port
--------------+--------------+----------------+-------------
              |              |                |
              | system       |                |
              | system       |                |
 xjnxdb_2_pt  | xjnxdb_2_pt  | 10.114.12.67   |       45654
 xjnxdb_2_pt  | xjnxdb_2_pt  | 10.114.12.67   |       45694
 tfsdb        | tfsdb        | 10.43.1.113    |       55971
 xjnxdb_2_pt  | xjnxdb_2_pt  | 10.114.12.67   |       45630
 xjnxdb_2_pt  | xjnxdb_2_pt  | 10.114.12.67   |       45684

# 只看正在运行的 SQL 信息
test=# select datname,usename,client_addr,client_port  from sys_stat_activity where state not like 'idle%';
   datname   |   usename   |  client_addr   | client_port
-------------+-------------+----------------+-------------
 xjnxdb_test | xjnxdb_test | 10.114.12.67   |       45706
 xjnxdb      | xjnxdb      | 10.114.200.108 |       54113
 test        | system      | 10.114.200.15  |       56839
(3 行记录)
```

<br>

<br>

## 查看耗时较长的 SQL

```shell
tfsdb=# select current_timestamp - query_start as runtime ,datname,usename,pid,query from  sys_stat_activity where state != 'idle' order by 1 desc;
-[ RECORD 1 ]----------------------------------------------------------------------------------------------------------------------------------------
runtime | 00:00:00.000000
datname | tfsdb
usename | system
pid     | 341563
query   | select current_timestamp - query_start as runtime ,datname,usename,pid,query from  sys_stat_activity where state != 'idle' order by 1 desc;
```



<br>

<br>

## 事务阻塞会话的简单处理

```shell
# 会话 1—关闭自动提交后删除记录
```



<br>

<br>

<br>

# 单表查询







<br>

<br>

<br>

# 学习备注

```shell
熟悉语义 /KingbaseES/V8/Server/bin/kingbase -D /data >log1 2>&1 &

systemctl , sys_ctl service各个命令的隔离性？


几种关闭模式需要详细了解和实验记录，需要自己动手做实验

sql 的元命令 需要详细了解一下呢

copy 和 \copy

事务阻塞会话的简单处理 这一块还需要了解一下
```









