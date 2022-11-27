---
title: 《MySQL 必知必会》study notes
author: ratears
date: 2022-06-04 14:56:48
updated: 2022-06-04 14:56:48
categories:
  - [database,MySQL]
tags:
  - MySQL
  - database
---

# 前言

- 近期学习了极客时间的专栏《MySQL 必知必会》，对专栏的核心知识做了学习笔记，便于参考及查漏补缺

<br>

<br>

<br>

# 课前准备 (2讲)

## 开篇词-在实战中学习，是解锁MySQL技能的最佳方法

- 熟练使用MySQL，对技术人来说变得越来越重要，是我们拿到心仪Offer的敲门砖
- <font color="red">**最重要的绝对不是你的知识储备量，而是你解决实际问题的能力**</font>
- **正确的学习方法，远比你投入的时间更重要**。而实战，就是最高效的方法
- **项目的实际需求-->解决问题所需的知识点-->用好这些知识的实战经验**

<br>

<br>

<br>

# 实践篇 (13讲)

## 01 | 存储：一个完整的数据存储过程是怎样的？

- 一个完整的**数据存储过程总共有4步，分别是创建数据库、确认字段、创建数据表、插入数据**

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.4qktlsqna5y0.webp" width="65%"/>



- MySQL数据库系统从大到小依次是：数据库服务器、数据库、数据表、数据表的行与列
- **数据库是MySQL里面最大的存储单元** 

<br>

<br>

## ① 创建数据库

```mysql
# 创建数据库
CREATE DATABASE `demo`  DEFAULT CHARACTER SET utf8mb4;

# 查看数据库
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| demo               |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)
```

- “information_schema”是MySQL系统自带的数据库，主要保存MySQL数据库服务器的系统信息，比如数据库的名称、数据表的名称、字段名称、存取权限、数据文件所在的文件夹和系统使用的文件夹，等等。
- “performance_schema”是MySQL系统自带的数据库，可以用来监控MySQL的各类性能指标
- “sys”数据库是MySQL系统自带的数据库，主要作用是，以一种更容易被理解的方式展示MySQL数据库服务器的各类性能指标，帮助系统管理员和开发人员监控MySQL的技术性能
- “mysql”数据库保存了MySQL数据库服务器运行时需要的系统信息，比如数据文件夹、当前使用的字符集、约束检查信息，等等

- 想深入了解MySQL数据库系统的相关信息，可以看下[官方文档](https://dev.mysql.com/doc/refman/8.0/en/system-schema.html)

<br>

<br>

## ② 确认字段

- MySQL会让我们确认新表中有哪些列，以及它们的数据类型。这些列就是MySQL数据表的字段

<br>

<br>

## ③ 创建数据表

- **创建表的时候，最好指明数据库**

```mysql
CREATE TABLE demo.test
( 
  barcode text,
  goodsname text,
  price int
); 
```

<br>

### 查看表的结构

```mysql
# 查看表的结构
mysql> describe demo.test;
+-----------+------+------+-----+---------+-------+
| Field     | Type | Null | Key | Default | Extra |
+-----------+------+------+-----+---------+-------+
| barcode   | text | YES  |     | NULL    |       |
| goodsname | text | YES  |     | NULL    |       |
| price     | int  | YES  |     | NULL    |       |
+-----------+------+------+-----+---------+-------+
3 rows in set (0.01 sec)
```

- Field：表示字段名称
- Type：表示字段类型
- Null：表示这个字段是否允许是空值（NULL）。注意，**在MySQL里面，空值不等于空字符串。一个空字符串的长度是0，而一个空值的长度是空。而且，在MySQL里面，空值是占用空间的。**
- Key：我们暂时把它叫做键。
- Default：表示默认值。
- Extra：表示附加信息。

<br>

### 查看数据库中的表

```mysql
USE demo;
SHOW TABLES;
```

<br>

### 设置主键

- MySQL中数据表的主键，是表中的一个字段或者几个字段的组合。它主要有3个特征：
  - 必须唯一，不能重复；
  - 不能是空；
  - 必须可以唯一标识数据表中的记录

> 一个MySQL数据表中只能有一个主键。虽然MySQL也允许创建没有主键的表，但是，**建议一定要给表定义主键，并且养成习惯。因为主键可以帮助减少错误数据，并且提高查询的速度**

```mysql
# 如果数据表中所有的字段都有重复的可能。我们可以自己添加一个不会重复的字段来做主键
ALTER TABLE demo.test ADD COLUMN itemnumber int PRIMARY KEY AUTO_INCREMENT;
```

<br>

<br>

## ④ 插入数据

```mysql
INSERT INTO demo.test VALUES ('003','橡皮',3);
```

- 要插入数据的字段名也可以不写，但是建议不要怕麻烦，**一定要每次都写**。这样做的好处是可读性好，不易出错，而且容易修改。否则，如果记不住表的字段，就只能去查表的结构，才能知道值所对应的字段了。
- 由于字段itemnumber定义了AUTO_INCREMENT，所以我们插入一条记录的时候，不给它赋值，系统也会自动给它赋值。而且，每次赋值，都会在上次的赋值基础上，自动增加1。也可以在插入一条记录的时候给itemnumber 赋值，由于它是主键，新的值必须与已有记录的itemnumber值不同，否则系统会提示错误。

<br>

<br>

## 小结（sql汇总）

```mysql
-- 创建数据库
CREATE DATABASE demo；
-- 删除数据库
DROP DATABASE demo；
-- 查看数据库
SHOW DATABASES;
-- 创建数据表：
CREATE TABLE demo.test
(  
  barcode text,
  goodsname text,
  price int
); 
-- 查看表结构
DESCRIBE demo.test;
-- 查看所有表
DESCRIBE TABLES;
-- 添加主键
ALTER TABLE demo.test
ADD COLUMN itemnumber int PRIMARY KEY AUTO_INCREMENT;
-- 向表中添加数据
INSERT INTO demo.test
(barcode,goodsname,price)
VALUES ('0001','本',3);
```

<br>

<br>

## 02 | 字段：这么多字段类型，该怎么定义？

> MySQL中有很多字段类型，比如整数、文本、浮点数，等等。如果类型定义合理，就能节省存储空间，提升数据查询和处理的速度，相反，如果数据类型定义不合理，就有可能会导致数据超出取值范围，引发系统报错，甚至可能会出现计算错误的情况，进而影响到整个系统。

<br>

<br>

### 整数类型

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.5qvu0awjo5s0.webp" width="75%"/>



- 在评估用哪种整数类型的时候，**需要考虑存储空间和可靠性的平衡问题**：一方面，用占用字节数少的整数类型可以节省存储空间；另一方面，要是为了节省存储空间，使用的整数类型取值范围太小，一旦遇到超出取值范围的情况，就可能引起系统错误，影响可靠性。

- 最佳实践
  - 在评估用哪种整数类型的时候，**需要考虑存储空间和可靠性的平衡问题**（在实际工作中，系统故障产生的成本远远超过增加几个字段存储空间所产生的成本）
  - **确保数据不会超过取值范围**，再去考虑如何节省存储空间

<br>

<br>

### 浮点数类型和定点数类型

> 可以把整数看成小数的一个特例

- FLOAT表示单精度浮点数
- DOUBLE表示双精度浮点数
- REAL默认就是DOUBLE。如果你把SQL模式设定为启用“REAL_AS_FLOAT”，那么，MySQL就认为REAL是FLOAT

```mysql
# 如果要启用“REAL_AS_FLOAT”，就可以通过以下SQL语句实现：
SET sql_mode = “REAL_AS_FLOAT”;
```

- FLOAT占用字节数少，取值范围小；DOUBLE占用字节数多，取值范围也大



<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.3rxrxdypui80.webp" width="70%" />



- 为什么浮点数类型的无符号数取值范围，只相当于有符号数取值范围的一半，也就是只相当于有符号数取值范围大于等于零的部分呢？

> 原因是：MySQL是按照这个格式存储浮点数的：符号（S）、尾数（M）和阶码（E）。因此，无论有没有符号，MySQL的浮点数都会存储表示符号的部分。因此，所谓的无符号数取值范围，其实就是有符号数取值范围大于等于零的部分。

<br>

#### 浮点数类型有缺陷：不精准

- 不精准问题原因在于：**MySQL对浮点类型数据的存储方式上**

> MySQL用4个字节存储FLOAT类型数据，用8个字节来存储DOUBLE类型数据。无论哪个，都是采用二进制的方式来进行存储的。比如9.625，用二进制来表达，就是1001.101，或者表达成1.001101×2^3。看到了吗？如果尾数不是0或5（比如9.624），你就无法用一个二进制数来精确表达。怎么办呢？就只好在取值允许的范围内进行近似（四舍五入）。
>

- 为什么数据类型是DOUBLE的时候，计算结果误差比FLOAT小一点？

> 原因：DOUBLE有8位字节，精度更高

<br>

#### **定点数类型：DECIMAL**

- DECIMAL的存储方式决定了它一定是精准的

> 浮点数类型是把十进制数转换成二进制数存储，DECIMAL则不同，它是把十进制数的整数部分和小数部分拆开，分别转换成十六进制数，进行存储。这样，所有的数值，就都可以精准表达了，不会存在因为无法表达而损失精度的问题



#### 小结：浮点数和定点数的特点/适用场景/最佳实践

> 浮点类型取值范围大，但是不精准，适用于需要取值范围大，又可以容忍微小误差的科学计算场景（比如计算化学、分子建模、流体动力学等）；定点数类型取值范围相对小，但是精准，没有误差，适合于对精度要求极高的场景（比如涉及金额计算的场景）

<br>

### 文本类型

- CHAR(M)：固定长度字符串。CHAR(M)类型必须预先定义字符串长度。如果太短，数据可能会超出范围；如果太长，又浪费存储空间。
- VARCHAR(M)： 可变长度字符串。VARCHAR(M)也需要预先知道字符串的最大长度，不过只要不超过这个最大长度，具体存储的时候，是按照实际字符串长度存储的。
- TEXT：字符串。系统自动按照实际长度存储，不需要预先定义长度。
- ENUM： 枚举类型，取值必须是预先设定的一组字符串值范围之内的一个，必须要知道字符串所有可能的取值。
- SET：是一个字符串对象，取值必须是在预先设定的字符串值范围之内的0个或多个，也必须知道字符串所有可能的取值。



- TEXT类型也有4种，它们的区别就是最大长度不同。

  - TINYTEXT：占用255字符。

  - TEXT： 占用65535字符。

  - MEDIUMTEXT：占用16777215字符。

  - LONGTEXT： 占用4294967295字符（相当于4GB）

- TEXT类型存在的问题：**由于实际存储的长度不确定，MySQL不允许TEXT类型的字段做主键。遇到这种情况，你只能采用CHAR(M)，或者VARCHAR(M)**

- 最佳实践
  - 项目中，只要不是主键字段，就可以按照数据可能的最大长度，选择这几种TEXT类型中的的一种，作为存储字符串的数据类型

<br>

<br>

### 日期与时间类型

<img src="https://cdn.jsdelivr.net/gh/sonzonzy/image-hosting@main/blog-img-bed/image.4fo8vblt22w0.webp" width="70%" />

- 最佳实践

> **在实际项目中，尽量用DATETIME类型**。因为这个数据类型包括了完整的日期和时间信息，使用起来比较方便
>
> 为了确保数据的完整性和系统的稳定性，优先考虑使用DATETIME类型。因为虽然DATETIME类型占用的存储空间最多，但是它表达的时间最为完整，取值范围也最大



- 为什么时间类型TIME的取值范围不是-23:59:59～23:59:59呢

> 原因是MySQL设计的TIME类型，不光表示一天之内的时间，而且可以用来表示一个时间间隔，这个时间间隔可以超过24小时。



#### 小结

```mysql
-- 修改字段类型语句
ALTER TABLE demo.goodsmaster
MODIFY COLUMN price DOUBLE;
-- 计算字段合计函数：
SELECT SUM(price)
FROM demo.goodsmaster;
```



> 在定义数据类型时，如果确定是整数，就用INT；如果是小数，一定用定点数类型DECIMAL；如果是字符串，只要不是主键，就用TEXT；如果是日期与时间，就用DATETIME。（首先确保你的系统不会因为数据类型定义出错。）

<br>

<br>

## 03 | 表：怎么创建和修改数据表？

### 创建数据表

```mysql
CREATE TABLE <表名>
{
字段名1 数据类型 [字段级别约束] [默认值]，
字段名2 数据类型 [字段级别约束] [默认值]，
......
[表级别约束]
};
```

- **“约束”限定了表中数据应该满足的条件**
- MySQL会根据这些限定条件，对表的操作进行监控，阻止破坏约束条件的操作执行，并提示错误，从而确保表中数据的唯一性、合法性和完整性

<br>

<br>

### 都有哪些约束

**1.非空约束**

- 非空约束表示字段值不能为空

**2.唯一性约束**

- 唯一性约束表示这个字段的值不能重复。**满足主键约束的字段，自动满足非空约束，但是满足唯一性约束的字段，则可以是空值**

**3.自增约束**

- 在数据表中，只有整数类型的字段（包括TINYINT、SMALLINT、MEDIUMINT、INT和BIGINT），才可以定义自增约束。自增约束的字段，每增加一条数据，值自动增加1。
- 给自增约束的字段赋值，这个时候，MySQL会重置自增约束字段的自增基数，下次添加数据的时候，自动以自增约束字段的最大值加1为新的字段值。

<br>

<br>

### 如何修改表

```mysql
CREATE demo.importheadhist
LIKE demo.importhead;

ALTER TABLE demo.importheadhist ADD confirmer INT;

ALTER TABLE demo.importheadhist ADD confirmdate DATETIME;
```

<br>

<br>

### 修改字段

- change 可以更改列名 和 列类型 (每次都要把新列名和旧列名写上, 即使两个列名没有更改,只是改了类型)
- modify 只能更改列属性 只需要写一次列名, 比change 省事点

```mysql
-- modify 能修改字段类型、类型长度、默认值、注释
ALTER  TABLE 表名 MODIFY [COLUMN] 字段名 新数据类型 新类型长度  新默认值  新注释;

ALTER  TABLE 表名 CHANGE [column] 旧字段名 新字段名 新数据类型;

-- 指定添加字段在表中位置
ALTER TABLE demo.importheadhist ADD suppliername TEXT AFTER supplierid;
```

<br>

<br>

### 小结

```mysql
CREATE TABLE
(
字段名 字段类型 PRIMARY KEY
);
CREATE TABLE
(
字段名 字段类型 NOT NULL
);
CREATE TABLE
(
字段名 字段类型 UNIQUE
);
CREATE TABLE
(
字段名 字段类型 DEFAULT 值
);
-- 这里要注意自增类型的条件，字段类型必须是整数类型。
CREATE TABLE
(
字段名 字段类型 AUTO_INCREMENT
);
-- 在一个已经存在的表基础上，创建一个新表
CREATE demo.importheadhist LIKE demo.importhead;
-- 修改表的相关语句
ALTER TABLE 表名 CHANGE 旧字段名 新字段名 数据类型;
ALTER TABLE 表名 ADD COLUMN 字段名 字段类型 FIRST|AFTER 字段名;
ALTER TABLE 表名 MODIFY 字段名 字段类型 FIRST|AFTER 字段名;
```

<br>

<br>

## 04 | 增删改查：如何操作表中的数据？

### 添加数据

```mysql
INSERT INTO 表名 [(字段名 [,字段名] ...)] VALUES (值的列表);
```

<br>

<br>

### 插入数据记录

- **部分插入一条数据记录是可以的，但前提是，没有赋值的字段，一定要让MySQL知道如何处理，比如可以为空、有默认值，或者是自增约束字段，等等，否则，MySQL会提示错误的**

<br>

<br>

### 插入查询结果

```mysql
INSERT INTO 表名 （字段名）
SELECT 字段名或值
FROM 表名
WHERE 条件
```

<br>

<br>

### 删除数据

```mysql
DELETE FROM 表名 WHERE 条件
```

<br>

<br>

### 修改数据

```mysql
UPDATE 表名 SET 字段名=值 WHERE 条件
```

- **不要修改主键字段的值**，如果你必须要修改主键的值，那有可能就是主键设置得不合理

<br>

<br>

### 查询数据

```mysql
SELECT *|字段列表
FROM 数据源
WHERE 条件
GROUP BY 字段
HAVING 条件
ORDER BY 字段
LIMIT 起始点，行数

-- GROUP BY：作用是告诉MySQL，查询结果要如何分组，经常与MySQL的聚合函数一起使用
-- HAVING：用于筛选查询结果，跟WHERE类似
```

<br>

<br>

### 小结

```mysql
INSERT INTO 表名 [(字段名 [,字段名] ...)] VALUES (值的列表);
 
INSERT INTO 表名 （字段名）
SELECT 字段名或值
FROM 表名
WHERE 条件
 
DELETE FROM 表名
WHERE 条件
 
UPDATE 表名
SET 字段名=值
WHERE 条件

SELECT *|字段列表
FROM 数据源
WHERE 条件
GROUP BY 字段
HAVING 条件
ORDER BY 字段
LIMIT 起始点，行数
```

- 如果我们把查询的结果插入到表中时，导致主键约束或者唯一性约束被破坏了，就可以用“ON DUPLICATE”关键字进行处理。这个关键字的作用是，告诉MySQL，如果遇到重复的数据，该如何处理。

```mysql
INSERT INTO demo.goodsmaster 
SELECT *
FROM demo.goodsmaster1 as a
ON DUPLICATE KEY UPDATE barcode = a.barcode,goodsname=a.goodsname;
```

<br>

<br>

# 05 | 主键：如何正确设置主键？

### 业务字段做主键

- **尽量不要用业务字段，也就是跟业务有关的字段做主键**。毕竟，作为项目设计的技术人员，我们谁也无法预测在项目的整个生命周期中，哪个业务字段会因为项目的业务需求而有重复，或者重用之类的情况出现

<br>

### 使用自增字段做主键

```mysql
UPDATE demo.trans AS a,demo.membermaster AS b
SET a.memberid=b.id
WHERE a.transactionno > 0  
AND a.cardno = b.cardno; 
-- 这样操作可以不用删除trans的内容，在实际工作中更适合
```

- 如果是一个小项目，只有一个MySQL数据库服务器，用添加自增字段作为主键的办法是可以的。不过，这并不意味着，在任何情况下你都可以这么做
- **自增字段做主键，对于单机系统来说是没问题的。但是，如果有多台服务器，各自都可以录入数据，那就不一定适用了。因为如果每台机器各自产生的数据需要合并，就可能会出现主键重复的问题**

<br>

### 手动赋值字段做主键

- **我们可以采用手动赋值的办法，通过一定的逻辑，确保字段值在全系统的唯一性，这样就可以规避主键重复的问题了**

> 取消字段“id”的自增属性，改成信息系统在添加会员的时候对“id”进行赋值。
>
> 门店在添加会员的时候，先到总部MySQL数据库中获取这个最大值，在这个基础上加1，然后用这个值作为新会员的“id”，同时，更新总部MySQL数据库管理信息表中的当前会员编号的最大值
>
> 各个门店添加会员的时候，都对同一个总部MySQL数据库中的数据表字段进行操作，就解决了各门店添加会员时会员编号冲突的问题，同时也避免了使用业务字段导致数据错误的问题

<br>

### 最佳实践

- 刚开始使用MySQL时，很多人都很容易犯的错误是喜欢用业务字段做主键，想当然地认为了解业务需求，但实际情况往往出乎意料，而更改主键设置的成本非常高。所以，如果你的系统比较复杂，尽量给表加一个字段做主键，采用手动赋值的办法，虽然系统开发的时候麻烦一点，却可以避免后面出大问题。

<br>

<br>

# 06 | 外键和连接：如何做关联查询？

- **把分散在多个不同的表里的数据查询出来的操作，就是多表查询**

<br>

### 创建外键

- **外键就是从表中用来引用主表中数据的那个公共字段**

> 在MySQL中，外键是通过外键约束来定义的。外键约束就是约束的一种，它必须在从表中定义，包括指明哪个是外键字段，以及外键字段所引用的主表中的主键字段是什么。MySQL系统会根据外键约束的定义，监控对主表中数据的删除操作。如果发现要删除的主表记录，正在被从表中某条记录的外键字段所引用，MySQL就会提示错误，从而确保了关联数据不会缺失。

```mysql

-- 外键约束定义的语法结构
[CONSTRAINT <外键约束名称>] FOREIGN KEY 字段名
REFERENCES <主表名> 字段名

-- 外键约束可以在创建表的时候定义
CREATE TABLE 从表名
(
  字段名 类型,
  ...
-- 定义外键约束，指出外键字段和参照的主表字段
CONSTRAINT 外键约束名
FOREIGN KEY (字段名) REFERENCES 主表名 (字段名)
)

-- 可以通过修改表来定义。
ALTER TABLE 从表名 ADD CONSTRAINT 约束名 FOREIGN KEY 字段名 REFERENCES 主表名 （字段名）;
```

<br>

### 连接

- MySQL中，有2种类型的连接，分别是内连接（INNER JOIN）和外连接（OUTER JOIN
  - 内连接表示查询结果只返回符合连接条件的记录
  - 外连接则不同，表示查询结果返回某一个表中的所有记录，以及另一个表中满足连接条件的记录
    - 左连接，一般简写成LEFT JOIN，返回左边表中的所有记录，以及右表中符合连接条件的记录。
    - 右连接，一般简写成RIGHT JOIN，返回右边表中的所有记录，以及左表中符合连接条件的记录

- 在MySQL里面，关键字JOIN、INNER JOIN、CROSS JOIN的含义是一样的，都表示内连接



> 大型网站的中央数据库，可能会因为外键约束的系统开销而变得非常慢。所以，MySQL允许你不使用系统自带的外键约束，在应用层面完成检查数据一致性的逻辑。也就是说，即使你不用外键约束，也要想办法通过应用层面的附加逻辑，来实现外键约束的功能，确保数据的一致性。

<br>

### 小结

```mysql
-- 定义外键约束：
CREATE TABLE 从表名
(
字段 字段类型
....
CONSTRAINT 外键约束名称
FOREIGN KEY (字段名) REFERENCES 主表名 (字段名称)
);
ALTER TABLE 从表名 ADD CONSTRAINT 约束名 FOREIGN KEY 字段名 REFERENCES 主表名 （字段名）;

-- 连接查询
SELECT 字段名
FROM 表名 AS a
JOIN 表名 AS b
ON (a.字段名称=b.字段名称);
 
SELECT 字段名
FROM 表名 AS a
LEFT JOIN 表名 AS b
ON (a.字段名称=b.字段名称);
 
SELECT 字段名
FROM 表名 AS a
RIGHT JOIN 表名 AS b
ON (a.字段名称=b.字段名称);
```

- **无法承担外键约束的成本，也可以不定义外键约束，但是一定要在应用层面实现外键约束的逻辑功能，这样才能确保系统的正确可靠**

<br>

<br>

# 07 | 条件语句：WHERE 与 HAVING有什么不同?









<br>

<br>

<br>

# 08 | 聚合函数：怎么高效地进行分组统计？





<br>

<br>

<br>

# 09 | 时间函数：时间类数据，MySQL是怎么处理的？









<br/>

# SQL汇总

```mysql
-- 创建数据库
CREATE DATABASE demo；
-- 删除数据库
DROP DATABASE demo；
-- 查看数据库
SHOW DATABASES;
-- 创建数据表：
CREATE TABLE demo.test
(  
  barcode text,
  goodsname text,
  price int
); 
-- 查看表结构
DESCRIBE demo.test;
-- 查看所有表
-- DESCRIBE TABLES;
-- 添加主键
ALTER TABLE demo.test
ADD COLUMN itemnumber int PRIMARY KEY AUTO_INCREMENT;
-- 向表中添加数据
INSERT INTO demo.test
(barcode,goodsname,price)
VALUES ('0001','本',3);

-- 修改字段类型语句
ALTER TABLE demo.goodsmaster
MODIFY COLUMN price DOUBLE;
-- 计算字段合计函数：
SELECT SUM(price)
FROM demo.goodsmaster;
```

<br>

<br>

<br>

# 参考延伸

- [SQL样式指南](https://www.sqlstyle.guide/zh/)
- 



# 学习备注

> 1. work_bench 的熟悉使用
> 2. 浮点数存储数据的方式，需要深入理解一下
> 3. 需要了解，修改表名，不同位置插入等

<br>

<br>

<br>

<img src="" width="70"/>

<br>

