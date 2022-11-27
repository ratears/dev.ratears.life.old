---
title: linux实战技能
author: sonzonzy
date: 2022-06-01 09:16:27
updated: 2022-06-01 09:16:27
categories:
tags:
  - linux
---

# 学习备注

- linux下的打包 压缩还需深入理解 和操作



# linux 基础介绍

- linux有两种含义
  - 一种是linus编写的开源操作系统内核
  - 另一种是广义的操作系统

- 内核版本（分为三个部分）
  - 主版本号、次版本号、末版本号
  - 次版本号是奇数为开发版本，偶数为稳定版

## linux 常见目录介绍

- / 根目录
- /root root用户家目录
- /home/username 普通用户的家目录
- /etc 配置文件目录
- /bin 命令目录
- /sbin 管理命令目录
- /usr/bin /usr/sbin 系统预装的其它命令

## linux 关机 / 重启 命令

```bash
# 关机
init 0

# 延时关机 19:30关机
shutdown -h 19:30

# 延时30分钟关机
shutdown -h +30 

# 重启
reboot
```

# 系统操作

## 帮助命令

### man

```linux
# man是manual的缩写
# 演示
man ls
```

### help 

- shell 自带的命令称为内部命令，其它的是外部命令

```linux
# 内部命令使用 help 帮助
help cd

# 外部命令使用 help 帮助
ls --help
```

### info

- info 帮助比 help 更详细，作为 help 的补充

```linux
info ls
```

## 文件 - 增删改查

- linux 操作系统中，一切皆文件

```linux
# 创建非空目录 
mkdir [参数]

# -p 递归创建目录
mkdir -p [参数]
```

```linux
# 删除非空目录
rm [参数]

# 递归删除目录（包括目录下的所有文件）
rm -r [参数]

# 不提示，无需确认，递归删除目录
rm -rf [参数]
```

```linux
# 仅仅复制文件
cp [] []

# 复制文件 or 目录
cp -r [] []

# -v 查看复制提示、过程  -p 复制的文件保留原文件的时间 -a 保留权限、保留属主
cp -vpa [] []
```

```linux
# 移动文件
mv [参数] [源文件] [目标文件/目录]

# 重命名
mv [] []
```

```linux
# 查看当前目录下的文件
ls [选项...] [参数...]

# -l 长格式显示文件
# -a 显示隐藏文件
# -r 逆序显示
# -t 按照时间顺序显示
# -R 递归显示

ll
```

## 通配符

- 定义：shell 内建的符号
- 用途：操作多个相似（有规律）的文件
- 常用通配符
  - \* 匹配任意字符串
  - ? 匹配一个字符串
  - [xyz] 匹配xyz任意一个字符
  - [a-z] 匹配一个范围
  - [!xyz] 或 [\^xyz] 不匹配

 ## 文本

```linux
touch []
```

```linux
# 文本内容显示到终端
cat []

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

## 打包 / 压缩

