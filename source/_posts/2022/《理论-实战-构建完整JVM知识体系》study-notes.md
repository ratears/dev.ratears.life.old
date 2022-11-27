---
title: 《理论+实战 构建完整JVM知识体系》study notes
author: sonzonzy
date: 2022-07-02 05:54:04
updated: 2022-07-02 05:54:04
categories:
  - [java,jvm]
tags:
  - jvm
---



# 认识JVM规范

## JVM概述

- JVM：Java Virtual Machine
- 所谓虚拟机是指：通过软件模拟的，具有完整硬件系统功能的、运行在一个完全隔离环境中的计算机

- JVM是通过软件来模拟Java字节码的指令集，是Java程序的运行环境

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.3luw1u10fja0.webp" width="90%" />

<br/>

### JVM主要功能

- 通过ClassLoader寻找和加载class文件
- 解释字节码成机器指令并执行，提供class文件运行环境
- 进行运行期间的内存分配和垃圾回收
- 提供与硬件交互的平台

<br/>

### 虚拟机是java平台无关的保障

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.rlwtwx7dsuo.webp" width="70%" />

<br/>

## JVM规范

### JVM规范的作用

- Java虚拟机规范为不通的硬件平台提供了一种编译Java技术代码的规范
- 该规范使Java软件独立于平台，因为编译是针对作为虚拟机的 ”一般机器“ 而做
- 这个 ”一般机器“ ，可用软件模拟，并运行于各种现存的计算机系统，也可用硬件来实现

<br/>

### JVM规范主要内容

- cpu(字节码指令集）
- class文件格式
- 数据类型和值
- 运行时数据区
- 栈帧
- 特殊方法
- 类库
- 异常
- 虚拟机的启动、加载、连接和初始化

<br/>

## Class字节码





