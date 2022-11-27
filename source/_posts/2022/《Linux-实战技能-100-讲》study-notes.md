---
title: 《Linux 实战技能 100 讲》study notes
auth: ratears
date: 2022-08-21 22:37:16
update: 2022-08-21 22:37:16
categories:
	- [linux]
tags:
	- linux
---



# linux 基础介绍

- linux有两种含义
  - 一种是linus编写的开源操作系统内核
  - 另一种是广义的操作系统

- linux内核版本（分为三个部分）
  - 主版本号、次版本号、末版本号
  - 次版本号是奇数为开发版本，偶数为稳定版

<br>

<br>

## linux 常见目录介绍

- / 根目录
- /root root用户家目录
- /home/username 普通用户的家目录
- /etc 配置文件目录
- /bin 命令目录
- /sbin 管理命令目录
- /usr/bin /usr/sbin 系统预装的其它命令

<br>

<br>

## linux 关机/重启 命令

```bash
# 关机
init 0

# 延时关机 19:30关机
shutdown -h 19:30

# 延时30分钟关机
shutdown -h 30 

# 停止延时关闭
shutdown -c

# 重启
reboot
```

<br>

<br>

<br>

# 系统操作

## 帮助命令

### man

```shell
# man是manual的缩写
# 演示
man ls
```



### help 

- shell 自带的命令称为内部命令，其它的是外部命令

```shell
# 内部命令使用 help 帮助
help cd

# 外部命令使用 help 帮助
ls --help

# 区分内部、外部命令 演示
type cd
```



### info

- info 帮助比 help 更详细，作为 help 的补充

```shell
info ls
```

<br>

<br>

## 文件/目录 - 增删改查

- linux 操作系统中，一切皆文件

```shell
# 创建非空目录 
mkdir [parameter]

# -p 递归创建目录
mkdir -p [parameter]
```

```shell
# 删除非空目录
rm [parameter]

# 递归删除目录（包括目录下的所有文件）
rm -r [parameter]

# 不提示，无需确认，递归删除目录
rm -rf [parameter]
```

```shell
# 仅仅复制文件
cp [] []

# 复制文件 or 目录
cp -r [] []

# -v 查看复制提示、过程  -p 复制的文件保留原文件的时间 -a 保留权限、保留属主
cp -vpa [] []

cd [parameter]
cd -
cd ~
```

```shell
# 移动文件
mv [参数] [源文件] [目标文件/目录]

# 重命名
mv [] []
```

```shell
# 查看当前目录下的文件
ls [选项...]...

# -l 长格式显示文件
# -a 显示隐藏文件
# -r 逆序显示
# -t 按照时间顺序显示
# -R 递归显示

ll

pwd
```

<br>

<br>

## 通配符

- 定义：shell 内建的符号
- 用途：操作多个相似（有规律）的文件
- 常用通配符
  - \* 匹配任意字符串
  - ? 匹配一个字符串
  - [xyz] 匹配xyz任意一个字符
  - [a-z] 匹配一个范围
  - [!xyz] 或 [\^xyz] 不匹配

<br>

<br>

 ## 文本

```shell
touch [file_name]
```

```shell
# 文本内容显示到终端
cat [text_file_name]

# 查看文件开头 -number  表示查看文件开头的number行 。不加默认查看10行
head -5 []

# 查看文件结尾 -number 表示查看文件结尾哦的number行 。不加默认查看10行  -f文件内容更新后，显示同步更新
tail -20 -f []

# 统计文件内容信息
wc []

# 查看文件行数
wc -l []

more [filename]
less [filename]
```

<br>

<br>

## 打包 / 压缩

- 打包/压缩

```shell
tar cf /tmp/etc.tar /etc

tar czf /tmp/etc.tar.gz /etc
tar cjf /tmp/etc.tar.bz2 /etc
```

- 解包/解压缩

```shell
tar xf /tmp/etc.tar -C /bak

tar zxf /tmp/etc.tar.gz -C /bak
tar jxf /tmp/etc.tar.bz2 -C /bak
```

<br>

<br>

##  文本编辑器/vi/vim

### 四种模式

- 正常模式
- 插入模式
- 命令模式
- 可视模式

```shell
# 进入插入模式 i I o O a A
i 进入插入模式，光标不移动
I 进入插入模式，光标移动到当前行的首字符
a 进入插入模式，光标移动到下一个字符
A 进入插入模式，光标移动到当前行的末尾
o 进入插入模式，光标移动到当前行的下一行（新建一行）
O 进入插入模式，光标移动到当前行的上一行（新建一行）

# 进入可视 模式 v

# 移动光标 h j k l 

yy 复制当前行
p 粘贴

3yy 复制3行
y$ 复制光标位置到当前行文本结尾

dd 剪切当前行
d$ 剪切光标位置到当前行文本结尾

u 撤销
ctrl + r 对撤销的内容重做

x 单个字符删除
r + [新字符] 替换

:set nu 显示行号
:set nonu 不显示行号
# 设置每次打开vim都显示行号
vim /etc/vimrc
# 在最后一行添加
set nu
# 保存退出后便生效


11+shift+g 11+G    光标移动到11行
gg 移动到第一行
G 移动到最后一行

shift+^ 移动到行首
shift+$ 移动到行尾

:w 保存
:q 退出
:wq 保存并退出

:q! 不保存退出

:! ll /etc/ 临时执行命令

/x 查找x，按n匹配下一个，shift+n 匹配上一个字符

:%s/x/X 全局替换查找到的第一个字符 （单次替换）
:s/x/X 光标所在行替换，替换查找到的第一个字符 （单次替换）

:%s/x/X/g 全局替换查找到的所有字符 
:s/x/X/g 光标所在行替换，替换查找到的当前行的所有字符 

:3,5s/x/X/g 第三行到第五行之间替换

# v 字符可视化
# V 行可视
# ctrl+v 块可视
```

<br>

<br>

## 用户和用户组管理

```shell
# 会创建同名的用户组
useradd [user_name]

# 不加 -r 会保留用户的家目录，加 -r 会在删除用户的同时删除其家目录
userdel -r [user_name]



passwd

#修改用户属性
usermod

# 修改用户家目录
usermod -d /wilson wilson

# 修改用户组
usermod -g group1 wilson

#修改用户属性（生命周期，密码修改周期等）
chage

# 用户信息会被记录到 /etc/passwd 文件中 /etc/shadow 这个是密码相关的文件

groupadd

# 新建用户并直接指定组
useradd -g group1 lily

groupdel

# 切换用户并切换home目录
su - wilson

# 以其它用户身份执行
# visudo 设置需要使用sudo的用户组
sudo 


visudo
# 赋予Wilson 执行如下命令的权限
wilson ALL=/sbin/shutdown -c

```

<br>

<br>

## 文件与目录权限

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.3v78rhmj4va0.webp" width="65%"/>

<br>

- 权限的前三个字符，表示所属用户对该文件的权限
- 中间三个字符，表示所属用户组对该文件的权限
- 最后三个字符表示其他人对该文件有什么权限

<br>

<br>

### 文件类型

• - 普通⽂文件
• d ⽬目录⽂文件
• b 块特殊⽂文件
• c 字符特殊⽂文件
• l 符号链接
• f 命名管道
• s 套接字⽂文件

<br>

<br>

<br>

# 系统管理

















# 学习备注

- linux下的打包 压缩还需深入理解 和操作
- vim



