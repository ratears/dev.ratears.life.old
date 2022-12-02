---
title: jvm
author: ratears
date: 2022-12-01 17:12:30
updated: 2022-12-01 17:12:30
categories:
	- [jvm]
tags:
	- jvm
---





# JVM与Java体系结构

## 前言

### **作为java工程师可能经常会遇到的问题**

- 运行着的线上系统突然卡死，系统无法访问，甚至直接OOM
- 想解决线上JVM GC问题，但却无从下手
- 新项目上线，对各种JVM参数设置一脸茫然，直接默认吧然后就JJ了
- 每次面试之前都要重新背一遍JVM的一些原理概念性的东西，然而面试官却经常问你在实际项目中如何调优VM参数，如何解决GC、OOM等问题，一脸懵逼

> 大部分Java开发人员，除会在项目中使用到与Java平台相关的各种高精尖技术，对于Java技术的核心Java虚拟机了解甚少。

<br>

### **开发人员如何看待上层框架**

- Java虚拟机的知识就好比公式的推导过程

> 计算机系统体系对我们来说越来越远，在不了解底层实现方式的前提下，通过高级语言很容易编写程序代码。但事实上计算机并不认识高级语言

<br>

### **我们为什么要学习JVM？**

- 面试的需要（BATJ、TMD，PKQ等面试都爱问）
- 中高级程序员必备技能
  - 项目管理、调优的需求
- 追求极客的精神
  - 比如：垃圾回收算法、JIT、底层原理

<br>

### **Java vs C++**

- 垃圾收集机制为我们打理了很多繁琐的工作，大大提高了开发的效率，但是，垃圾收集也不是万能的，懂得JVM内部的内存结构、工作机制，是设计高扩展性应用和诊断运行时问题的基础，也是Java工程师进阶的必备能力。

<br>

### **面向人群及参考书目**

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.39wp8r5uj6m0.webp" width="70%">

<br>

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.1urwzyssik68.webp" width="70%">

<br>

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.20k9lclpjjts.webp	" width="70%">

<br>

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.bwj574rfetk.webp" width="70%">

<br>

<br>

## Java及JVM简介

> 世界上没有最好的编程语言，只有最适用于具体应用场景的编程语言

<br>

### **JVM：跨语言的平台**

- Java是目前应用最为广泛的软件开发平台之一。随着Java以及Java社区的不断壮大Java 也早已不再是简简单单的一门计算机语言了，它更是一个平台、一种文化、一个社区。



- 作为一个平台，Java虚拟机扮演着举足轻重的作用
  - Groovy、Scala、JRuby、Kotlin等都是Java平台的一部分
- 作为一种文化，Java几乎成为了“开源”的代名词。
  - 第三方开源软件和框架。如Tomcat、Struts，MyBatis，Spring等。
  - 就连JDK和JVM自身也有不少开源的实现，如openJDK、Harmony。
- 作为一个社区，Java拥有全世界最多的技术拥护者和开源社区支持，有数不清的论坛和资料。从桌面应用软件、嵌入式开发到企业级应用、后台服务器、中间件，都可以看到Java的身影。其应用形式之复杂、参与人数之众多也令人咋舌。

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.5iwwquuo9ms0.webp" width="70%">

<br>

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.y3kf2h91gww.webp" width="70%">

<br>

> - 随着Java7的正式发布，Java虚拟机的设计者们通过JSR-292规范基本实现在Java虚拟机平台上运行非Java语言编写的程序。
> - Java虚拟机根本不关心运行在其内部的程序到底是使用何种编程语言编写的，它只关心“字节码”文件。也就是说Java虚拟机拥有语言无关性，并不会单纯地与Java语言“终身绑定”，只要其他编程语言的编译结果满足并包含Java虚拟机的内部指令集、符号表以及其他的辅助信息，它就是一个有效的字节码文件，就能够被虚拟机所识别并装载运行。

<br>

### **字节码**

- 我们平时说的java字节码，指的是用java语言编译成的字节码。准确的说任何能在jvm平台上执行的字节码格式都是一样的。所以应该统称为：jvm字节码。
- 不同的编译器，可以编译出相同的字节码文件，字节码文件也可以在不同的JVM上运行。
- Java虚拟机与Java语言并没有必然的联系，它只与特定的二进制文件格式—Class文件格式所关联，Class文件中包含了Java虚拟机指令集（或者称为字节码、Bytecodes）和符号表，还有一些其他辅助信息。

<br>

### **多语言混合编程**

- Java平台上的多语言混合编程正成为主流，通过特定领域的语言去解决特定领域的问题是当前软件开发应对日趋复杂的项目需求的一个方向。
- 试想一下，在一个项目之中，并行处理用Clojure语言编写，展示层使用JRuby/Rails，中间层则是Java，每个应用层都将使用不同的编程语言来完成，而且，接口对每一层的开发者都是透明的，各种语言之间的交互不存在任何困难，就像使用自己语言的原生API一样方便，因为它们最终都运行在一个虚拟机之上。
- 对这些运行于Java虚拟机之上、Java之外的语言，来自系统级的、底层的支持正在迅速增强，以JSR-292为核心的一系列项目和功能改进（如Da Vinci Machine项目、Nashorn引擎、InvokeDynamic指令、java.lang.invoke包等），推动Java虚拟机从“Java语言的虚拟机”向 “多语言虚拟机”的方向发展。

<br>

### **如何真正搞懂JVM？**

- Java虚拟机非常复杂，要想真正理解它的工作原理，最好的方式就是自己动手编写一个！

<br>

<br>

## Java发展的重大事件

- 1990年，在Sun计算机公司中，由Patrick Naughton、MikeSheridan及James Gosling领导的小组Green Team，开发出的新的程序语言，命名为oak，后期命名为Java
- 1995年，Sun正式发布Java和HotJava产品，Java首次公开亮相。
- 1996年1月23日，Sun Microsystems发布了JDK 1.0。
- 1998年，JDK1.2版本发布。同时，sun发布了JSP/Servlet、EJB规范，以及将Java分成了J2EE、J2SE和J2ME。这表明了Java开始向企业、桌面应用和移动设备应用3大领域挺进。
- 2000年，JDK1.3发布，Java HotSpot Virtual Machine正式发布，成为Java的默认虚拟机。
- 



<br>

<br>

<br>



























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