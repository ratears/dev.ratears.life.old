---
title: Spring
date: 2022-07-16 21:42:14
tags:
	- spring
---



# Spring 概览

- Spring IoC

|       内容        |               说明               | 重要程度 |
| :---------------: | :------------------------------: | :------: |
|  Spring 框架介绍  | Spring IoC、DI 和 AOP 等核心概念 |  ※※※※※   |
|  Spring IoC 容器  |     Spring 实例化与管理对象      |  ※※※※※   |
|   集合对象注入    |    注入List、Set、Map集合对象    |  ※※※※※   |
|     底层原理      |      Spring Bean 的生命周期      |  ※※※※※   |
| 注解与Java Config |  Spring 注解分类和常用注解应用   |  ※※※※※   |

<br/>

Spring AOP

|      内容       |                说明                 | 重要程度 |
| :-------------: | :---------------------------------: | :------: |
| 理解 AOP 及名词 |      Spring AOP 开发与配置流程      |   ※※※※   |
|  五种通知类型   |    Spring 五种通知类型与应用场景    |   ※※※    |
|   切点表达式    | PointCut 切点表达式的语法规则及应用 |    ※※    |
|    代理模式     | JDK 动态代理和 CGLib代理的执行过程  |   ※※※※   |

<br/>

- Spring JDBC 与声明式事物

|        内容        |              说明               | 重要程度 |
| :----------------: | :-----------------------------: | :------: |
|    Spring JDBC     |     Spring JDBC 的环境配置      |   ※※※※   |
|    RestTemplate    | 基于 RestTemplate 实现 SQL 处理 |   ※※※    |
|   配置声明式事物   |      声明式事物的配置过程       |  ※※※※※   |
|  事物传播行为介绍  |   讲解常用事物传播行为的用途    |   ※※※    |
| 声明式事物注解形式 |     基于注解使用声明式事物      |  ※※※※※   |

<br/>

## Spring 的含义

- Spring 可以从狭义与广义两个角度看待
- 狭义的 Spring 是指 Spring 框架（Spring Framework）
- 广义的 Spring 是指 Spring 生态体系



### 狭义的 Spring 框架

- Spring 框架是企业开发复杂性的一站式解决方案
- Spring 框架的核心是 IoC 容器与 AOP 面向切面编程
- Spring IoC 负责创建与管理系统对象，并在此基础上扩展功能



### 广义的 Spring 生态体系

<img src="https://git.poker/sonzonzy/image-hosting/blob/main/blog-img-bed/image.ueqlwks019s.webp?raw=true"  width="80%"/>

<br/>

## IoC 控制反转

- IoC 控制反转，全称 Inverse of Control ，是一种设计理念
- 由代理人来创建与管理对象，消费者通过代理人来获取对象
- IoC 的目的是降低对象之间直接耦合，加入 IoC 容器将对象统一管理，让对象关联变为弱耦合



## DI

- IoC 是设计理念，是现代程序设计遵循的标准，是宏伟目标
- DI （Dependency Injection）是具体技术实现，是微观实现
- DI 在 Java 中利用反射技术实现对象注入（Injection）



## Spring框架组成模块

<img src="https://git.poker/sonzonzy/image-hosting/blob/main/blog-img-bed/image.55c4tw9371o0.webp?raw=true" width="70%" />

<br/>

# Spring IoC

## 传统开发方式

- 对象直接饮用，导致对象硬性关联，程序难以扩展维护

<img src="https://git.poker/sonzonzy/image-hosting/blob/main/blog-img-bed/image.58jxehmqgy80.webp?raw=true"  width="80%"/>

<br/>

## Spring IoC 容器

- IoC 容器是 Spring 生态的地基，用于统一创建与管理对象依赖

<img src="https://git.poker/sonzonzy/image-hosting/blob/main/blog-img-bed/image.4lxmp6457ng0.webp?raw=true"  width="80%"/>

<br/>

## Spring IoC 容器职责

- 对象的控制权交由第三方统一管理（IoC 控制反转）
- 利用 Java 反射技术实现运行时对象创建与关联（DI 依赖注入）
- 基于配置提高应用程序的可维护性与可扩展性



## 配置 bean 的三种方式

- 基于XML配置bean
- 基于注解配置bean
- 基于Java代码配置bean



### XML实例化Bean的配置方式

#### （1）基于构造方法实例化对象

```xml
    <!--通过默认构造方法创建对象-->
    <bean id="lily" class="com.study.spring.entity.Person" >
    </bean>
    
    <!--通过有参构造方法创建对象-->
    <bean id="andy" class="com.study.spring.entity.Person" >
        <!-- 没有constructor-arg 则代表调用默认构造方法实例化        -->
        <constructor-arg name="nickName" value="andy" ></constructor-arg>
        <constructor-arg name="age" value="18"></constructor-arg>
    </bean>
    
    <!--通过有参构造方法创建对象-->
    <bean id="nacos" class="com.study.spring.entity.Person" >
        <!-- 利用构造方法参数位置实现对象实例化        -->
        <constructor-arg index="0" value="nacos" ></constructor-arg>
        <constructor-arg index="1" value="18"></constructor-arg>
    </bean>
```



#### （2）基于静态工厂实例化对象

```java
/**
 * 使用静态工厂方法创建对象，隐藏创建对象的细节
 */
public class PersonStaticFactory {
    public static Person createPerson() {
        Person person = new Person();
        person.setAge(19);
        person.setNickName("张三");
        System.out.println("使用静态工厂方法创建对象，隐藏创建对象的细节");
        return person;
    }
}
```

```xml
<!--利用静态工厂实例化对象    -->
    <bean id="personStaticFactory" class="com.study.spring.factory.PersonStaticFactory"
          factory-method="createPerson" ></bean>
```



#### （3）基于工厂实例方法实例化对象

```java
/**
 * 工厂实例方法创建对象是指IoC容器对工厂类进行实例化并调用对应的实例方法创建对象的过程
 */
public class PersonFactoryInstance {
    public  Person createPerson() {
        Person person = new Person();
        person.setAge(19);
        person.setNickName("张三");
        System.out.println("使用工厂实例方法创建对象，隐藏创建对象的细节");
        return person;
    }
}
```

```xml
 	<!--利用工厂实例方法实例化bean -->
    <bean id="personFactoryInstance" class="com.study.spring.factory.PersonFactoryInstance" ></bean>
    <bean id="wangwu" factory-bean="personFactoryInstance" factory-method="createPerson" ></bean>

```



## 从Spring IoC 容器获取bean

```java
        // 创建IoC容器并根据配置文件创建对象（初始化IoC容器并实例化对象）
        ApplicationContext ac = new ClassPathXmlApplicationContext("classpath:applicationContext.xml");

        Person person = ac.getBean("andy", Person.class);

        Person person1 = (Person) ac.getBean("nacos");
```



###  xml 方式中 id 与 name属性相同点

- bean id 与 name 都是设置对象在 ioc 容器中唯一标识
- 两者在同一配置文件中都不允许出现重复
- 两者允许在多个配置文件中出现重复，新对象 覆盖旧对象



### xml 方式中 id 与 name 属性区别

- id 要求更为严格，一次只能定义一个对象标识（推荐）
- name 更为宽松，一次允许定义多个对象标识
- tips：id 与 name 的命名要求有意义。按驼峰命名书写



```xml
    <!-- 没有 id 与 name，默认使用类全名 作为 bean 标识-->
    <bean class="com.study.spring.entity.Person">
        <constructor-arg name="nickName" value="andy2" ></constructor-arg>
        <constructor-arg name="age" value="118"></constructor-arg>
    </bean>
```

```java
ApplicationContext ac = new ClassPathXmlApplicationContext("classpath:applicationContext.xml");

Person p = (Person) ac.getBean("com.study.spring.entity.Person");
```



### Spring 配置文件，配置路径表达式

<img src="https://git.poker/sonzonzy/image-hosting/blob/main/blog-img-bed/image.1i297kdcvls0.webp?raw=true" width="70%"/>

### 加载Spring 的配置文件

```java
 // 加载单个配置文件
ApplicationContext ac = new ClassPathXmlApplicationContext("classpath:applicationContext.xml");
```

```java
// 加载多个配置文件
String [] config = new String[]{"classpath:applicationContext.xml","classpath:applicationContext-2.xml"};
ApplicationContext applicationContext = new ClassPathXmlApplicationContext(config);
```



## 对象依赖注入

- 依赖注入是指运行时将容器内对象利用反射赋给其它对象的操作



### 基于setter方法注入对象

- 利用setter对象实现静态数值注入

```xml
<!--ioc 容器自动利用反射机制运行时调用 setXXX方法为属性赋值-->
    <bean id="guog" class="com.study.spring.entity.Person">
        <property name="nickName" value="guod" ></property>
        <property name="age" value="55" ></property>
    </bean>
```

- 利用setter对象实现对象注入







### 基于构造方法注入对象

- 





