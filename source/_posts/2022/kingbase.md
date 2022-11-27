---
title: kingbase
author: sonzonzy
date: 2022-05-17 02:51:05
updated: 2022-05-17 02:51:05
categories:
  - [database,kingbase]
tags:
  - database
  - kingbase
---

# Kingbase 安装与启停

## 安装前准备工作

- 服务器安装jdk1.8+版本并配置环境变量
- 创建kingbase用户组与用户。创建目录，并设置目录属组、属组、权限
- 上传kingbase安装包和kingbase的license.dat 到服务器（安装包和license可以到[官网](https://www.kingbase.com.cn/rjcxxz/index.htm)下载） 

  <br/>

```shell
# 为了利于数据库的日常运维、持续使用、存储扩容等，我们在安装前需做好选项、存储目录规划

# 使用root用户登录进服务器
# /install 安装软件上传目录  
# /kingbase/V8  数据库安装目录  
# /backup 备份目录  
# /data 数据存储目录  
# /archive 归档目录
[root@bu2-vm-svr-66 ~]# mkdir -p /install /KingbaseES/V8 /backup /data /archive

# 上传 安装包、license.dat 到 /install 目录
# 上传 执行脚本 optimize_system_conf_kcp.sh 和 optimize_database_conf.s 到 /install 目录
# optimize_system_conf_kcp.sh 优化操作系统的脚本   optimize_database_conf.sh 优化数据库的脚本

[root@bu2-vm-svr-66 ~]# ll /install
总用量 852348
-rw-r--r--. 1 root root 872781824 5月  11 03:27 KingbaseES_V008R006C005B0023_Lin                                      64_single_install.iso
-rw-r--r--. 1 root root      3351 5月  11 03:27 license_12350_0.dat
-rw-r--r--. 1 root root      6504 5月  11 03:27 optimize_database_conf.sh
-rw-r--r--. 1 root root      8023 5月  11 03:27 optimize_system_conf_kcp.sh


# 执行 optimize_system_conf_kcp.sh（优化操作系统的脚本）。主要帮我们 创建了kingbase 用户组和用户（用户密码：kingbase）。具体详情可以查看脚本内容
bash /install/optimize_system_conf_kcp.sh

[root@node1 install]# id kingbase
uid=1001(kingbase) gid=1001(kingbase) 组=1001(kingbase)

# 修改目录属组、属主、权限
chown -R kingbase:kingbase /install /KingbaseES /backup /archive /data
chmod -R 775 /install /KingbaseES /backup /archive
chmod -R 700 /data
```

 <br/>

## Kingbase 安装

```shell
# 我们使用的是 KingbaseES_V008R006C005B0023_Lin64_single_install.iso 文件，所以先使用root用户登录并挂载
[root@node1 install]# mount -o loop /install/KingbaseES_V008R006C005B0023_Lin64_single_install.iso  /mnt/

[root@node1 install]# ll /mnt/
总用量 6
dr-xr-xr-x. 2 root root 2048 11月  5 2021 setup
-r-xr-xr-x. 1 root root 3820 11月  5 2021 setup.sh
```

```bash
# 
# 使用kingbase 用户登录服务器 ，进入/mnt 下执行 setup.sh 则开始安装 （
[kingbase@node1 mnt]$ cd /mnt
[kingbase@node1 mnt]$ bash setup.sh

# 也可以使用命令行安装
./setup.sh -i console
```

```bash
# 安装完成后 使用root用户登录进服务器，把数据库服务注册成系统服务。并启动数据库
[root@node1 ~]# /KingbaseES/V8/install/script/root.sh


# 把kingbase注册成系统服务后（root用户执行 /KingbaseES/V8/Scripts/root.sh 后）。kingbase已经启动了，但此时 为什么不可以使用 systemctl  这种方式启停 kingbase？
# 必须 使用sys_ctl 数据库先停止. 然后再使用systemctl 启动数据. 才能成功启动, 因为systemctl 需要获取进程状态信息.
```

- 运行 数据库优化文件

```bash
bash /install/optimize_database_conf.sh
```

<br/>

## Kingbase 相关环境变量配置

```bash
# 使用kingbase用户登录。配置ksql环境变量

[root@sonronzy ~]# su - kingbase
[kingbase@sonronzy ~]$ cd ~
[kingbase@sonronzy ~]$ vi .bashrc

export PATH=/KingbaseES/V8/Server/bin:$PATH

[kingbase@sonronzy ~]$ source .bashrc
```

```sql
# 使用kingbase用户登录 ，使用sys_ctl 专用命令管理金仓数据库
# 配置sys_ctl 环境变量
[kingbase@sonronzy ~]$ cd ~
[kingbase@sonronzy ~]$ vim .bashrc

export PATH
export KINGBASE_DATA=/data
export PATH=/KingbaseES/V8/Server/bin:$PATH

[kingbase@sonronzy ~]$ source .bashrc
```

<br/>

## Kingbase 启停

```bash
# kingbase 是进程，kingbase8d 是服务
# 注意没有修改linux参数的时候 systemctl 和 service 方式启停数据库 不要混用

service kingbase8d start/stop/restart/status
service kingbase8 start/stop/restart/status

systemctl start kingbase8d

/etc/init.d/kingbase8d start

# kingbase用户 ，使用sys_ctl 专用命令管理金仓数据库
# 首先要配置 环境变量

sys_ctl start/stop/restart/status
```

```bash
# 有任何用户连接到数据库里来，都无法关闭数据库，必须等所有用户提交完数据断开连接后 才可关闭数据库。这个可能会关闭很长时间
sys_ctl stop -m smart

# 默认方式 关闭数据库。最好选用这个。 已经提交的用户踢开连接，未提交的用户 回滚，然后关闭数据库（一致状态，安全关闭方式）
sys_ctl stop -m fast | sys_ctl stop

# 断电式关闭数据库 (不推荐，可能会导致数据不一致，数据库无法启动)
sys_ctl -m immediate
```

<br/>

## Kingbase 卸载

```bash
# 使用root用户登录，进入到数据库的安装目录下的 Scripts 目录下，执行 rootuninstall.sh 卸载kingbase数据库
cd /KingbaseES/V8/Scripts

/KingbaseES/V8/Scripts/rootuninstall.sh

# 最后确认已删除的kingbase8d 服务
```

<br/>

## 实践环境中常见的问题

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/gitee-blog/0097e80f862e58ac1ee42674eb5f03e.5mzf9cefwgc0.webp"  width="80%"/>

<br/>

## * 注意事项

### Kingbase数据库大小写敏感说明及转换

[<font color="red">Kingbase数据库大小写敏感说明及转换</font>](https://bbs.kingbase.com.cn/wenda/question/137.html)

<br/>

# Kingbase 客户端

## Kingbase 对象管理器

- 我们安装kingbase的时候，如果选择完全安装，则会帮我们安装上 数据库对象管理工具
- 使用Kingbase 对象管理器连接数据库 操作如下图

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/gitee-blog/image.6gb216h4y0g0.webp"  width="90%"/>

### 模式

- 业务软件所使用的对象的集合。包括：表、视图、序列、索引、函数、存储过程等......
- 非模式对象：表空间

<br/>

## 其它客户端

- 当我们不想在本机安装kingbase数据库时，可以选择第三方数据库客户端连接kingbase数据库。可以使用的相关客户端有：DBeaver、DataGrip 2020.1 x64、Dbvisualizer

<br/>

### DBeaver

- [官网](https://dbeaver.io/download/) 下载 DBeaver
- 使用DBeaver 连接Kingbase 数据库 操作如下图

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/gitee-blog/image.5o8iqfctzac0.webp"  />

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/gitee-blog/image.4z57m966ojc0.webp" />

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/gitee-blog/image.7essaooqm480.webp" /> 

<br/>

# ksql 命令行工具

- Ksql是Kingbase的交互式终端
- 支持上下键翻页，tab键补全

<br/>

## ksql登录 、执行sql语句、sql脚本

```bash
ksql -Usystem -h localhost -p54321 -W TEST

ksql -Usystem -W xjnxdb -l

ksql -Usystem -W xjnxdb -c  "select * from xjnxdb.pa_user"

ksql -Usystem -W test -f /install/1.sql
```

<br/>

## ksql终端常用快捷键

```bash
# 查看所有快捷键
\?

# help
\h create

# 查看当前有哪些数据库
\l

# 查看表结构
\d xjxndb.pa_user

\c 切换数据库
23:17:59 (system@[local]:54321)TEST=# \c template1
口令：
您现在已经连接到数据库 "template1",用户 "system".
23:18:09 (system@[local]:54321)template1=#


# 退出ksql终端
\q
```

<br/>

## 自定义sql提示符

```bash
# 定制sql提示符，便于我们了解 目前在哪台终端、哪个用户、哪个数据库下操作

# 使用kingbase用户登录

[kingbase@sonronzy ~]$ vim ~/.ksqlrc
[kingbase@sonronzy ~]$
[kingbase@sonronzy ~]$ cat ~/.ksqlrc
\set PROMPT1 '%`date +%H:%M:%S` (%n@%M:%>)%/%R%#%x '
\set PROMPT2 '%M %n@%/%R%# '
[kingbase@sonronzy ~]$
[kingbase@sonronzy ~]$ source ~/.ksqlrc
[kingbase@sonronzy ~]$
[kingbase@sonronzy ~]$ ksql -Usystem -W TEST
口令：
ksql (V8.0)
输入 "help" 来获取帮助信息.

23:13:03 (system@[local]:54321)TEST=#
```

```bash
\set PROMPT1 '%`date +%H:%M:%S` (%n@%M:%>)%/%R%#%x '
\set PROMPT2 '%M %n@%/%R%# '
```

解析： 
◆ %M  	指数据库服务器的主机名 - 如果连接是通过 Unix 域套接字，则为“[local]” 
◆ %m  	也表示数据库主机名，会截断第一个 . 后的内容 
◆ %>		数据库端口号 
◆ %n		是指会话用户名 
◆ %/		当前数据库名 
◆ %#		如果是超级用户显示为 #，否则显示为 > 
◆ %R		是指您处于单行模式（^）还是断开连接（！ ） ，但通常为= 
◆ %x		指的是事务状态 - 通常为空白，除非在事务块（*）

<br/>

## 常用sql

```sql
show database_mode;

show shared_buffers;

select get_license_validdays();
```





## copy 与 \copy

- copy 命令属于 SQL 命令， \copy 命令属于元命令
- copy 命令进行数据导出、导入时，需要具有 superuser 的权限；导出至 stdout 时，仅需模式、
  对象的相关权限即可；\copy 命令进行数据导出、导入时，无需 superuser 权限
- copy 命令只能在源数据库服务器上进行数据导出、导入；\copy 命令还可以通过远程服务器连
  接至源数据库服务器，将数据导出至远程服务器、或将远程服务器的数据导入源数据库中
- 大数据量的数据进行导出、导入时，copy 比\copy 的性能高
- 如果进行小数据量导出、导入，建议通过\copy 操作便利；大数据量操作时，建议在源数据库中
  使用 copy 效率更高

<br/>

# Kingbase 数据迁移

## 第一步：基础数据结构及数据迁移

- 准备工作
  - 根据需要创建用户、表空间、模式等对象
- 使用【数据库迁移工具】完成基础数据迁移工作

<br/>

## 第二步：应用接口及框架迁移

### springboot 数据源配置

```yaml
#环境业务自身配置开始
#默认数据源default，不能修改
spring.datasource.dynamic.primary = default

#默认数据源，名称 default
#spring.datasource.dynamic.datasource.default.driver-class-name=com.mysql.jdbc.Driver
#spring.datasource.dynamic.datasource.default.url=jdbc:mysql://localhost:3306/xjnxdb?useUnicode=true&characterEncoding=utf8&serverTimezone=UTC&useSSL=false
#spring.datasource.dynamic.datasource.default.username=root
#spring.datasource.dynamic.datasource.default.password=root

#kingbase 数据源配置
spring.datasource.dynamic.datasource.default.driver-class-name=com.kingbase8.Driver
spring.datasource.dynamic.datasource.default.url=jdbc:kingbase8://10.114.12.66:54321/xjnxdb
spring.datasource.dynamic.datasource.default.username=xjnxdb
spring.datasource.dynamic.datasource.default.password=xjnxdb
```

<br/>

### maven 配置Kingbase 驱动

在maven repository中查找kingbase的驱动依赖配置，加入到我们的pom文件

```xml
<!-- https://mvnrepository.com/artifact/kingbase/kingbase8 -->
<dependency>
    <groupId>kingbase</groupId>
    <artifactId>kingbase8</artifactId>
    <version>8</version>
</dependency>
```

- <font color="red">**注意：**</font>我们会发现Kingbase8驱动依赖根本下载不下来。此时：**我们可以把驱动下载到本地，再使用maven命令install到maven本地仓库即可**

[kingbase8-8.jar](https://maven.jeecg.org/nexus/content/repositories/jeecg/kingbase/kingbase8/8/kingbase8-8.jar)

```xml
mvn install:install-file -DgroupId=kingbase -DartifactId=kingbase8 -Dversion=8 -Dfile=D:\bak\kingbase8-8.jar -Dpackaging=jar -DgeneratePom=true

mvn install:install-file -DgroupId=kingbase -DartifactId=kingbase8 -Dversion=8.6.0 -Dfile=H:\bank\kingbase8-8.6.0.jar -Dpackaging=jar -DgeneratePom=true
```

> 该语句中参数：
>
> DgroupId ：组id 【对应pom中的groupId】
> DartifactId：项目id 【对应pom中的artifactId】
> Dversion：版本号 【对应pom中的version】
> Dfile：jar包的绝对路径
> Dpackaging：是什么包
> DgeneratePom：是否生成pom

- 最后在pom中直接写入 dependency 就可以了，刷新即可使用

```xml
<!-- https://mvnrepository.com/artifact/kingbase/kingbase8 -->
<dependency>
    <groupId>kingbase</groupId>
    <artifactId>kingbase8</artifactId>
    <version>8.6.0</version>
</dependency>
```

<br/>

## 第三步：应用功能测试（SQL兼容情况）

### date_format 函数支持

```java
@Mapper
@TableInfo(name = "wf_sequence", primaryKeys = {"seqNo"})
public interface WfSequenceMapper {
	@Update("update wf_sequence set seqval = seqval+1 where seqno=#{seqNo}")
	int incBySeqNo(@Param("seqNo") String seqNo);

	@Select("select * from wf_sequence where seqno=#{seqNo}")
	WfSequenceDO getBySeqNo(@Param("seqNo") String seqNo);

	@Insert("insert into wf_sequence (seqno,seqval,seqdesc)values(#{seqNo},#{seqVal},#{seqDesc})")
	int insert(WfSequenceDO seq);

	@Select("SELECT concat(DATE_FORMAT(sysdate(),'%Y%m%d'),right(lpad(seqval,15,0),8)) as seqDesc FROM wf_sequence where seqno=#{seqNo}")
	WfSequenceDO getTxnBySeqNo(@Param("seqNo") String seqNo);

}
```

```sql
00:28:12 (system@[local]:54321)TEST=# select date_format('2022-05-15','yyyy-mm-dd');
错误:  函数 date_format(unknown, unknown) 不存在
第1行select date_format('2022-05-15','yyyy-mm-dd');
            ^
提示:  没有匹配指定名称和参数类型的函数. 您也许需要增加明确的类型转换.
00:28:15 (system@[local]:54321)TEST=#
```

- 参考 《[应用开发及迁移][参考手册]KingbaseES扩展插件参考手册.pdf》

> - kdb_date_function
>
> kdb_date_function 是一个兼容 mysql 数据库 date 相关函数的扩展。使用时需要 `create extension kdb_date_function`，不需要时 `drop extension kdb_date_function` 即可。

```sql
00:41:28 (system@[local]:54321)TEST=# select date_format('2022-05-15','yyyy-mm-dd');
错误:  函数 date_format(unknown, unknown) 不存在
第1行select date_format('2022-05-15','yyyy-mm-dd');
            ^
提示:  没有匹配指定名称和参数类型的函数. 您也许需要增加明确的类型转换.
00:41:29 (system@[local]:54321)TEST=#
00:41:30 (system@[local]:54321)TEST=#
00:41:30 (system@[local]:54321)TEST=#
00:41:30 (system@[local]:54321)TEST=# create extension kdb_date_function;
CREATE EXTENSION
00:44:42 (system@[local]:54321)TEST=# select date_format('2022-05-15','yyyy-mm-dd');
 date_format
-------------
 2022-05-15
(1 行记录)

00:44:46 (system@[local]:54321)TEST=#
```

<br/>

# 数据库迁移评估系统

- 具体详情参考官方文档
