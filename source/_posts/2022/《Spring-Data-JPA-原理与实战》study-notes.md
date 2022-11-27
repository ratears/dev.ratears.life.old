---
title: 《Spring Data JPA 原理与实战》study notes
auth: ratears
date: 2022-09-06 04:08:46
update: 2022-09-06 04:08:46
categories:
	- [spring,jpa]
tags:
	- jpa
	- spring
---



# Spring Data JPA基础知识

## Spring Data JPA优势

- 某些大公司的大部分项目都在用 Spring Data JPA，究其原因，主要缘于它具有以下 4 点优势

### 第一，大势所趋，大厂必备技能

> 近两年由于 Spring Cloud、Spring Boot 逐渐统一 Java 的框架江湖，而与 Spring Boot 天然集成的 Spring Data JPA 也逐渐走进了 Java 开发者的视野，大量尝鲜者享受到了这门技术带来的便利与功能。JPA 可以使团队在框架约定下进行开发，几乎很难出现有性能瓶颈的 SQL。因此你会发现很多大厂，如阿里、腾讯、抖音等公司，近几年在招聘的时候写明了要熟悉 JPA，这些大厂以及业内很多开源的新项目都在使用 JPA

<br>

### 第二，提升开发效率

> 现在有很多人知道什么是 Spring Data JPA，但是却觉得 JPA 很难用，使用中发现 Bug 不知道原因，本来用 JPA 是为了提升开发效率的，不会使用反倒踩了很多坑，所以我们需要体系化地学习。当你遇到复杂问题，比如平时你可能要花几个小时去想方法名、SQL 逻辑，如果你可以熟练使用 JPA，那么半小时甚至几分钟就可以写好查询方法了；再配合测试用例，你的开发质量也会明显提高很多，系统地学习可以让你少走很多弯路

<br>

### 第三，提高技术水平

> Spring Data 对数据操作进行了大统一，统一了抽象关系型数据库和非关系型数据的接口、公共的部分，你会发现当掌握了 Spring Data JPA 框架后，你的开发水平几乎可以达到——轻易实现 Redis、MongoDB 等 NoSQL 的操作，因为它们都有统一的 Spring Data Common。如下图所示，从中你可以看到 Spring Data 和 JPA 的全景位置关系，这样一来，你可以清楚地知道 JPA 的重要作用，方便你了解 JPA 的脉络，从而更好地学习



<img src="https://git.poker/ratears/image-hosting/blob/main/blog-img-bed/image.55f368bvsmc0.webp?raw=true" width="65%" />

<br>

### 第四，求职加分项

> 如果简历中突出 Spring Data JPA 框架的使用，面试官会眼前一亮。因为掌握了 JPA，就意味着掌握了很多原理，比如 Session 原理、事务原理、PersistenceContext 原理等，而掌握了底层原理对于技术人员来说可以在开发中解决很多问题。因此，公司可以由此更好地过滤和筛选人才，也能从侧面看出求职者是否对技术足够感兴趣、有追求













# 备注

> 1. 第一篇需要动手跟着学习构建