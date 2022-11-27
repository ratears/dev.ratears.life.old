---
title: Java - Collection Framework
author: ratears
date: 2022-06-22 00:36:47
updated: 2022-06-22 00:36:47
categories:
  - [java,collection]
tags:
  - collection
---



# Collection

- 容器主要包括 Collection 和 Map 两种，Collection 存储着对象的集合，而 Map 存储着键值对(两个对象)的映射表

- Collection 接口常用方法

```java
boolean	add(E e)
boolean	addAll(Collection<? extends E> c)

void	clear()
boolean	remove(Object o)
    
Iterator<E>	iterator()
int	size()
    
boolean	contains(Object o)
boolean	equals(Object o) 
boolean	isEmpty()
```

- 集合只能存放引用数据类型，不能存放基本数据类型