---
title: Maven核心知识梳理
date: 2022-09-17 20:45:22
tags:
	- maven
categories:
	- maven
---



# 第一章 Maven概述

## Why？为什么要学习 Maven？

### Maven 作为依赖管理工具

#### ①jar 包的规模

- 随着我们使用越来越多的框架，或者框架封装程度越来越高，项目中使用的jar包也越来越多。项目中，一个模块里面用到上百个jar包是非常正常的
- 比如下面的例子，我们只用到 SpringBoot、SpringCloud 框架中的三个功能：
  - Nacos 服务注册发现
  - Web 框架环境
  - 图模板技术 Thymeleaf

> 最终却导入了 106 个 jar 包



而如果使用 Maven 来引入这些 jar 包只需要配置三个『**依赖**』：

```xml
	<!-- Nacos 服务注册发现启动器 -->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>

    <!-- web启动器依赖 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- 视图模板技术 thymeleaf -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
```



#### ②jar 包的来源

- 这个jar包所属技术的官网。官网通常是英文界面，网站的结构又不尽相同，甚至找到下载链接还发现需要通过特殊的工具下载
- 第三方网站提供下载。问题是不规范，在使用过程中会出现各种问题:
  - jar包的名称
  - jar包的版本
  - jar包内的具体细节
- 而使用 Maven 后，依赖对应的 jar 包能够**自动下载**，方便、快捷又规范



#### ③jar 包之间的依赖关系

- 框架中使用的 jar 包，不仅数量庞大，而且彼此之间存在错综复杂的依赖关系。依赖关系的复杂程度，已经上升到了完全不能靠人力手动解决的程度。另外，jar 包之间有可能产生冲突。进一步增加了我们在 jar 包使用过程中的难度

- 下面是前面例子中 jar 包之间的依赖关系：

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.mqcmj2wu734.webp" width="65%" />

<br>

- 而实际上 jar 包之间的依赖关系是普遍存在的，如果要由程序员手动梳理无疑会增加极高的学习成本，而这些工作又对实现业务功能毫无帮助
- **使用 Maven 则几乎不需要管理这些关系，极个别的地方调整一下即可，极大的减轻了我们的工作量**

<br>

<br>

### Maven 作为构建管理工具

#### ①你没有注意过的构建

- 可以不使用 Maven，但是构建必须要做。当我们使用 IDEA 进行开发时，构建是 IDEA 替我们做的



#### ②脱离 IDE 环境仍需构建

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.1ivp2tn48qw0.webp" width="60%"/>

<br>

<br>

### 结论

- **管理规模庞大的 jar 包，需要专门工具。**
- **脱离 IDE 环境执行构建操作，需要专门工具。**

<br>

<br>

<br>

## What？什么是 Maven？

- Maven 是 Apache 软件基金会组织维护的一款专门为 Java 项目提供**构建**和**依赖**管理支持的工具

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.3ecian9do5e0.webp" width="30%"/>

<br>

### 构建

> Java 项目开发过程中，构建指的是使用**『原材料生产产品』**的过程

- 原材料
  - Java 源代码
  - 基于 HTML 的 Thymeleaf 文件
  - 图片
  - 配置文件
  - ......
- 产品
  - 一个可以在服务器上运行的项目



- **构建过程包含的主要的环节：**

> - 清理：删除上一次构建的结果，为下一次构建做好准备
> - 编译：Java 源程序编译成 *.class 字节码文件
> - 测试：运行提前准备好的测试程序
> - 报告：针对刚才测试的结果生成一个全面的信息
> - 打包
>   - Java工程：jar包
>   - Web工程：war包
> - 安装：把一个 Maven 工程经过打包操作生成的 jar 包或 war 包存入 Maven 仓库
> - 部署
>   - 部署 jar 包：把一个 jar 包部署到 Nexus 私服服务器上
>   - 部署 war 包：借助相关 Maven 插件（例如 cargo），将 war 包部署到 Tomcat 服务器上



### 依赖

- 如果 A 工程里面用到了 B 工程的类、接口、配置文件等等这样的资源，那么我们就可以说 A 依赖 B。例如：
  - junit-4.12 依赖 hamcrest-core-1.3
  - thymeleaf-3.0.12.RELEASE 依赖 ognl-3.1.26
    - ognl-3.1.26 依赖 javassist-3.20.0-GA



- 依赖管理中要解决的具体问题：
  - jar 包的下载：使用 Maven 之后，jar 包会从规范的远程仓库下载到本地
  - jar 包之间的依赖：通过依赖的传递性自动完成
  - jar 包之间的冲突：通过对依赖的配置进行调整，让某些jar包不会被导入



### Maven 的工作机制

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.6wuv1htwzko0.webp" width="60%"/>

<br>

<br>

<br>

# 第二章 Maven 核心程序解压和配置

## Maven核心程序解压与配置

- Maven 官网地址：[Maven – Welcome to Apache Maven](https://maven.apache.org/)



- 解压Maven核心程序
  - 核心程序压缩包：apache-maven-3.8.4-bin.zip，解压到**非中文、没有空格**的目录
  - 在解压目录中，我们需要着重关注 Maven 的核心配置文件：**conf/settings.xml**



- 指定本地仓库
  - 建议将 Maven 的本地仓库放在其他盘符下。配置方式如下：

```xml
<!-- localRepository
| The path to the local repository maven will use to store artifacts.
|
| Default: ${user.home}/.m2/repository
<localRepository>/path/to/local/repo</localRepository>
-->
<localRepository>D:\maven-repository</localRepository>
```



- 配置阿里云提供的镜像仓库
  - Maven 下载 jar 包默认访问境外的中央仓库，而国外网站速度很慢。改成阿里云提供的镜像仓库，**访问国内网站**，可以让 Maven 下载 jar 包的时候速度更快。配置的方式是：

```xml
<!--将原有的例子配置注释掉 -->
<!-- <mirror>
  <id>maven-default-http-blocker</id>
  <mirrorOf>external:http:*</mirrorOf>
  <name>Pseudo repository to mirror external repositories initially using HTTP.</name>
  <url>http://0.0.0.0/</url>
  <blocked>true</blocked>
</mirror> -->
```

```xml
	<!--加入我们的配置 -->
	<mirror>
		<id>nexus-aliyun</id>
		<mirrorOf>central</mirrorOf>
		<name>Nexus aliyun</name>
		<url>http://maven.aliyun.com/nexus/content/groups/public</url>
	</mirror>
```



- 配置 Maven 工程的基础 JDK 版本

```xml
	<profile>
	  <id>jdk-1.8</id>
	  <activation>
		<activeByDefault>true</activeByDefault>
		<jdk>1.8</jdk>
	  </activation>
	  <properties>
		<maven.compiler.source>1.8</maven.compiler.source>
		<maven.compiler.target>1.8</maven.compiler.target>
		<maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
	  </properties>
	</profile>
```

<br>

<br>

## 配置环境变量

- 检查 JAVA_HOME 配置是否正确
  - Maven 是一个用 Java 语言开发的程序，它必须基于 JDK 来运行，需要通过 JAVA_HOME 来找到 JDK 的安装位置
- 配置 MAVEN_HOME
- 配置PATH
- 验证

```bash
C:\Users\Administrator>mvn -v
Apache Maven 3.8.4 (9b656c72d54e5bacbed989b64718c159fe39b537)
Maven home: D:\software\apache-maven-3.8.4
Java version: 1.8.0_141, vendor: Oracle Corporation, runtime: D:\software\Java\jre
Default locale: zh_CN, platform encoding: GBK
OS name: "windows 10", version: "10.0", arch: "amd64", family: "windows"
```

<br>

<br>

<br>

# 第三章 使用 Maven：命令行环境

## 实验一：根据坐标创建 Maven 工程

### Maven 核心概念：坐标

#### ①数学中的坐标

- 使用 x、y、z 三个**『向量』**作为空间的坐标系，可以在**『空间』**中唯一的定位到一个**『点』**



#### ②Maven中的坐标

[1]向量说明

- 使用三个**『向量』**在**『Maven的仓库』**中**唯一**的定位到一个**『jar』**包。
  - **groupId**：公司或组织的 id
  - **artifactId**：一个项目或者是项目中的一个模块的 id
  - **version**：版本号



[2]三个向量的取值方式

- groupId：公司或组织域名的倒序，通常也会加上项目名称
  - 例如：com.atguigu.maven
- artifactId：模块的名称，将来作为 Maven 工程的工程名
- version：模块的版本号，根据自己的需要设定
  - 例如：SNAPSHOT 表示快照版本，正在迭代过程中，不稳定的版本
  - 例如：RELEASE 表示正式版本



> 举例：

- groupId：com.atguigu.maven
- artifactId：pro01-atguigu-maven
- version：1.0-SNAPSHOT



#### ③坐标和仓库中 jar 包的存储路径之间的对应关系

```xml
  <!-- 坐标： -->
  <groupId>javax.servlet</groupId>
  <artifactId>servlet-api</artifactId>
  <version>2.5</version>
```

```text
上面坐标对应的 jar 包在 Maven 本地仓库中的位置：
Maven本地仓库根目录\javax\servlet\servlet-api\2.5\servlet-api-2.5.jar
```



### 实验操作：使用命令生成Maven工程

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.52zgdx8nqpc0.webp" width="40%"/>

<br>

- 运行 **mvn archetype:generate** 命令

> TIP
>
> Choose a number or apply filter (format: [groupId:]artifactId, case sensitive contains): 7:【直接回车，使用默认值】
>
> Define value for property 'groupId': com.atguigu.maven
>
> Define value for property 'artifactId': pro01-maven-java
>
> Define value for property 'version' 1.0-SNAPSHOT: :【直接回车，使用默认值】
>
> Define value for property 'package' com.atguigu.maven: :【直接回车，使用默认值】
>
> Confirm properties configuration: groupId: com.atguigu.maven artifactId: pro01-maven-java version: 1.0-SNAPSHOT package: com.atguigu.maven Y: :【直接回车，表示确认。如果前面有输入错误，想要重新输入，则输入 N 再回车。】



- 调整

```xml
<!-- Maven 默认生成的工程，对 junit 依赖的是较低的 3.8.1 版本，我们可以改成较适合的 4.12 版本。

自动生成的 App.java 和 AppTest.java 可以删除  -->

<!-- 依赖信息配置 -->
<!-- dependencies复数标签：里面包含dependency单数标签 -->
<dependencies>
	<!-- dependency单数标签：配置一个具体的依赖 -->
	<dependency>
		<!-- 通过坐标来依赖其他jar包 -->
		<groupId>junit</groupId>
		<artifactId>junit</artifactId>
		<version>4.12</version>
		
		<!-- 依赖的范围 -->
		<scope>test</scope>
	</dependency>
</dependencies>
```



- 自动生成的 pom.xml 解读

```xml
  <!-- 当前Maven工程的坐标 -->
  <groupId>com.atguigu.maven</groupId>
  <artifactId>pro01-maven-java</artifactId>
  <version>1.0-SNAPSHOT</version>
  
  <!-- 当前Maven工程的打包方式，可选值有下面三种： -->
  <!-- jar：表示这个工程是一个Java工程  -->
  <!-- war：表示这个工程是一个Web工程 -->
  <!-- pom：表示这个工程是“管理其他工程”的工程 -->
  <packaging>jar</packaging>

  <name>pro01-maven-java</name>
  <url>http://maven.apache.org</url>

  <properties>
	<!-- 工程构建过程中读取源码时使用的字符集 -->
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>

  <!-- 当前工程所依赖的jar包 -->
  <dependencies>
	<!-- 使用dependency配置一个具体的依赖 -->
    <dependency>
	
	  <!-- 在dependency标签内使用具体的坐标依赖我们需要的一个jar包 -->
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.12</version>
	  
	  <!-- scope标签配置依赖的范围 -->
      <scope>test</scope>
    </dependency>
  </dependencies>
```

<br>

### Maven核心概念：POM

#### 含义

- POM：**P**roject **O**bject **M**odel，项目对象模型。和 POM 类似的是：DOM（Document Object Model），文档对象模型。它们都是模型化思想的具体体现



#### 模型化思想

- POM 表示将工程抽象为一个模型，再用程序中的对象来描述这个模型。这样我们就可以用程序来管理项目了。我们在开发过程中，最基本的做法就是将现实生活中的事物抽象为模型，然后封装模型相关的数据作为一个对象，这样就可以在程序中计算与现实事物相关的数据



#### 对应的配置文件

- POM 理念集中体现在 Maven 工程根目录下 **pom.xml** 这个配置文件中。所以这个 pom.xml 配置文件就是 Maven 工程的核心配置文件。其实学习 Maven 就是学这个文件怎么配置，各个配置有什么用。



### Maven核心概念：约定的目录结构

#### ①各个目录的作用

<img src="https://cdn.staticaly.com/gh/ratears/image-hosting@main/blog-img-bed/image.178pqbq36pd.webp" width="60%"/>

<br>

- 另外还有一个 target 目录专门存放构建操作输出的结果



#### ②约定目录结构的意义

- Maven 为了让构建过程能够尽可能自动化完成，所以必须约定目录结构的作用。例如：Maven 执行编译操作，必须先去 Java 源程序目录读取 Java 源代码，然后执行编译，最后把编译结果存放在 target 目录。



#### ③约定大于配置

- Maven 对于目录结构这个问题，没有采用配置的方式，而是基于约定。这样会让我们在开发过程中非常方便。如果每次创建 Maven 工程后，还需要针对各个目录的位置进行详细的配置，那肯定非常麻烦。

- 目前开发领域的技术发展趋势就是：约定大于配置，配置大于编码。





# 学习备注

> 1. maven的工作机制还需要深入熟悉

<br>

<img src="" width="60%"/>

<br>

