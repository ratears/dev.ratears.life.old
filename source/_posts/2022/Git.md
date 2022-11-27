---
title: Git
author: sonzonzy
date: 2022-06-22 00:36:47
updated: 2022-06-22 00:36:47
categories:
  - git
tags:
  - git
---

# 学习备注

> 1、要清楚执行每个git命令后 提示信息表达的意思，不会的单词要记忆，写在这个文档里面

# 版本控制系统

## 什么是版本控制系统

- 版本控制系统是一种记录一个或若干文件内容变化，以便将来查阅特定版本修订情况的系统

## 为什么要使用版本控制

- 软件开发中采用版本控制系统是个明智的选择
- 有了它你就可以将某个文件回溯到之前的状态,甚至将整个项目都回退到过去某个时间点的状态
- 就算你乱来一气把整个项目中的文件改的改删的删，你也照样可以轻松恢复到原先的样子
- 你可以比较文件的变化细节,查出最后是谁修改了哪个地方,从而找出导致怪异问题出现的原因，又是谁在何时报告了某个功能缺陷等等
- 但额外增加的工作量却微乎其微

## 版本管理的演变

- VCS 出现前
  - 用目录拷贝区别不不同版本
  - 公共文件容易易被覆盖
  - 成员沟通成本很高，代码集成效率低下
- 集中式 VCS
  - 有集中的版本管理服务器
  - 具备文件版本管理和分支管理能力
  - 集成效率有明显地提高
  - 客户端必须时刻和服务器相连
- 分布式 VCS
  - 服务端和客户端都有完整的版本库
  - 脱离服务端，客户端照样可以管理理版本
  - 查看历史和版本比较等多数操作，都不不需要访问服务器器，比集中式 VCS 更更能提⾼高版本管理理效率

## 版本控制系统的分类

### 集中化的版本控制系统

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/gitee-blog/image.4l3axedwasg0.webp" width="60%" >

> 集中化的版本控制系统诸如CVS, SVN 以及Perforce 等，都有一个单一的集中管理的服务器,保存所有文件的修订版本，而协同工作的人们都通过客户端连到这台服务器,取出最新的文件或者提交更新。多年以来,这已成为版本控制系统的标准做法,这种做法带来了许多好处,现在,每个人都可以在一定程度上看到项目中的其他人正在做些什么。而管理员也可以轻松掌控每个开发者的权限，并且管理一个集中化的版本控制系统;要远比在各个客户端上维护本地数据库来得轻松容易。
>
> 事分两面，有好有坏。这么做最显而易见的缺点是中央服务器的**单点故障**。如果服务器宕机一小时，那么在这一小时内， 谁都无法提交更新，也就无法协同工作



### 分布式的版本控制系统

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/gitee-blog/image.4l3axedwasg0.webp" width="60%" >

- 由于上面集中化版本控制系统的那些缺点，于是分布式版本控制系统面世了
- 在这类系统中，像Git, BitKeeper 等,**客户端并不只提取最新版本的文件快照，而是把代码仓库完整地镜像下来**
- 许多这类系统都可以指定和**若干不同的远端代码仓库进行交互**。这样，你就可以在同一个项目中分别和不同工作小组的人相互协作

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/gitee-blog/image.2ip2z3escfc0.webp" width="65%" >

- 分布式的版本控制系统在管理项目时存放的不是项目版本与版本之间的差异.它存的是索引(所需磁盘空间很少所以每个客户端都可以放下整个
  项目的历史记录)

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/gitee-blog/image.55w0uw51yhs0.webp" width="65%" >

## 版本控制系统的存储方式

- 世界上的版本控制总共有两种存储方式，一种是存储差异，另一种是存储快照
- 存储差异：存储base文件，以后每次存储base文件的更改，SVN就是这种方式
- 存储快照：每次更改都存储一个新文件，Git是这种方式

# Git 基础

## 概念

- Git是一个<font color=#008000>免费的</font>、<font color=#008000>开源的</font><font color=#008000>**分布式**</font><font color=red>版本控制系统</font>，可以快速高效地管理从小型到大型的项目

## Git 简史

- Linux 内核开源项目有着为数众多的参与者。 绝大多数的 Linux 内核维护工作都花在了提交补丁和保存归档的繁琐事务上（1991－2002年间）。 到 2002 年，整个项目组开始启用一个专有的分布式版本控制系统 BitKeeper 来管理和维护代码。

- 到了 2005 年，开发 BitKeeper 的商业公司同 Linux 内核开源社区的合作关系结束，他们收回了 Linux 内核社区免费使用 BitKeeper 的权力。 这就迫使 Linux 开源社区（特别是 Linux 的缔造者 Linus Torvalds）基于使用 BitKeeper 时的经验教训，开发出自己的版本系统。 他们对新的系统制订了若干目标

## Git 的使命 / 目标

Git在设计之初就是为了搞定linux内核这种巨无霸而设计的，所以制定了自己的使命

- 速度
- 简单的设计
- 对非线性开发模式的强力支持（允许成千上万个并行开发的分支）
- 完全分布式
- 有能力高效管理类似 Linux 内核一样的超大规模项目（速度和数据量）

## Git 的优点 / 特点

- 快、非凡的性能
- 本地仓库
- 轻量级分支
- 分布式
- 各种工作流
- 最优的存储能力
- 开源的
- 很容易易做备份
- 支持离线操作
- 很容易易定制工作流程

## Git 的结构

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/gitee-blog/image.1756qy15ac5c.webp" width="60%" >

## Git 的交互方式

### 代码托管中心是干嘛的

- 我们已经有了本地库，本地库可以帮我们进行版本控制，为什么还需要代码托管中心呢？
  它的任务是帮我们维护远程库

### 本地库和远程库的交互方式

#### 团队内部协作

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/gitee-blog/image.6ia1ri43p640.webp" >

#### 跨团队协作

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/gitee-blog/image.n89r9sjz3gw.webp" >

### 托管中心种类

- 局域网环境下：可以搭建 GitLab服务器作为代码托管中心，GitLab可以自己去搭建
- 外网环境下：可以由GitHub或者Gitee作为代码托管中心，GitHub或者Gitee是现成的托管中心，不用自己去搭建 

## Git 的下载安装 / 基本设置

- 下载安装略（官网下载，一般情况傻瓜式安装即可）
- 查看git安装版本（是否安装成功）

```bash
git --version
```

### 基本设置

- 配置`user.name`和`user.email`

```bash
git config --global  user.name 'sonzonzy'
git config --global  user.email 'sonzonzy@gmail.com'
```

### config 的三个作用域

- 缺省等同于 local
- 优先级：local > global > system

```bash
# local只对当前仓库有效
git config --local

# global 对当前登录用户所有仓库有效
git config --global

# system 对系统的所有用户有效
git config --system
```

### 显示 config 的配置

```bash
git config --list --local
git config --list --global
git config --list --system
```

### 信息设置与清除

- 设置	缺省等同于 local

```bash
git config --local user.name "ratears"
git config --global user.name "ratears"
git config --system user.name "ratears"
```

- 清除	--unset

```bash
git config --unset --local user.name
git config --unset --global user.name
git config --unset --system user.name
```

# Git 常用命令 & 操作

## init / 初始化本地仓库

```bash
# 在git终端进入到本地的文件夹 （例如 $ cd D:\dev\git_ws\git_demo） 执行如下命令
#初始化本地仓库，只能初始化本地仓库
git init
```

## add / 添加到暂存区

```bash
git add .
```

## commit / 提交到本地仓库

```bash
# 把暂存区的 文件提交到本地仓库。-m"message" 后的双引号 填写该次提交的说明信息
git commit -m"add test1.txt"
```

- 注意事项

  - 不放在本地仓库中的文件，git是不进行管理

  - 即使放在本地仓库的文件，git也不管理，必须通过add,commit命令操作才可以将内容提交到本地库

## status / 查看工作区和暂存区的状态

```bash
git status
```

## mv / 重命名暂存区的文件

- 方式一

```bash
mv readme readme.md
git rm readme
git add readme.md
git commit -m"rename file"
```

- 方式二

```bash
git mv readme readme.md
git commit -m'file name'
```

## log / 查看提交日志

```bash
# 可以让我们查看提交的，显示从最近到最远的日志
git log -n[number] --graph --online --all/[branch_name]
# -n 指定查看条数
# --graph 图形化查看
# --online 简单显示
# --all 显示所有分支，不加则显示当前分支
# branch_name 指定分支

git log --pretty=oneline

git reflog
# 多了信息：HEAD@{数字} 这个数字的含义：指针回到当前这个历史版本需要走多少步

```

```bash
# 在浏览器打开git log 的帮助文档
git help --web log
```

## gitk / git 的gui界面

```bash
# 打开git的gui界面
gitk
```

## rest / 前进或者后退历史版本

### hard 参数

```bash
# 本地库的指针移动的同时，重置暂存区，重置工作区
git reset --hard [索引]
```

### mixed参数

```bash
# 本地库的指针移动的同时，重置暂存区，但是工作区不动
git reset --mixed [索引]
```

### soft参数

```bash
# 本地库的指针移动的时候，暂存区，工作区都不动
git reset --soft [索引]
```

## diff 

```bash
# 将工作区中的文件和暂存区中文件进行比较 
git diff [文件名]  

# 比较工作区中和暂存区中 所有文件的差异
git diff

# 比较暂存区和工作区中内容
git diff [历史版本] [文件名] 
```

# 分支

