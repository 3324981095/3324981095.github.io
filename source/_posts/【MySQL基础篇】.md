---
title: mysql 基础篇
data: 2025-5-13
description: 本期带来mysql基础
mathjax: true
sticky: 3
swiper_index: 2
cover: 'https://cdn.szyd.fun/2025/04/26/680cd3227de71.jpg'
tags:
  - mysql
  - 数据库
categories:
  - mysql
abbrlink: df2d8b43
---

# 前言
### 数据库是什么

> 数据库是一个有组织的、可以存储、管理、操作大量数据的系统。它允许你方便地存储、查询、更新和删除数据，确保数据的安全性、完整性和高效访问。换句话说，数据库就是一个“数据仓库”，它能让我们高效地存储和提取需要的信息。
> **关系型数据库**（如 MySQL、SQL Server、Oracle 等）是最常见的一种，它通过表格（类似于 Excel 的行和列）来组织数据。每一行代表一条记录，每一列代表数据的某个属性。通过特定的查询语言（SQL），你可以对数据进行检索和操作。
> 

> 简单来讲，数据库时在磁盘或者内存中存储的特定结构组织的数据--将来在磁盘上存储的一套数据库方案。
> 数据库服务--mysqld

### 初学数据库掌握的核心概念

> **1、数据库管理系统（DBMS）**
>  * 数据库本身并不是一个独立的软件，而是由数据库管理系统（DBMS）来管理。DBMS负责创建、管理和操作数据库。常见的DBMS包括 MySQL、Oracle、PostgreSQL 和 SQL Server 等。
> 
> **2、表、行和列**
> * 在关系型数据库中，数据通常以表格的形式存储。每个表由若干行（记录）和列（字段）组成。每一行代表一条数据记录，而每一列则代表该记录的某个属性。
> 
> **3、SQL语言**
> * SQL（结构化查询语言）是与数据库交互的标准语言。通过 SQL，你可以执行各种操作，如插入数据、查询数据、更新数据、删除数据等。掌握 SQL 是学习数据库的关键一步。
> 
> **4、主键与外键**
> * 主键（Primary Key）：是表中用来唯一标识一条记录的字段。每个表只能有一个主键，且主键的值不能重复。
> * 外键（Foreign Key）：是用来在表之间建立关联的字段。外键字段的值在另一个表的主键或唯一键中必须存在。
> 
> **5、关系与查询**
> * 数据库中的表之间通常有关系，关系可以通过外键来建立。通过 SQL 查询语言，你可以连接不同的表，查询出你需要的综合数据。
### 为什么要用数据库
> 一般的文件确实提供了数据的存储功能，但是文件没有提供良好的数据管理能力（用户角度）

### SQL分类
| 分类 | 全称 |说明|  
|--|--|--|
|DDL  |Data Definition  Language          |数据定义语言，用来定义数据库对象(数据库，表，字段)|
| DML | Data Manipulation Language |数据操作语言，用来对数据库表中的数据进行增删改|  
|DQL|Data Query Language|数据查询语言，用来查询数据库中表的记录|
|DCL  | Data Control Language |数据控制语言，用来创建数据库用户、控制数据库的访问权限|


# 一、认识MySQL
## 1.1MySQL启动
首先按win+r键启动命令窗口，输入cmd，打开后输入以下指令

```cpp
mysql -u root -p
```
接下来，输入自己密码即可（图中*为密码）。如果出现以下情况，说明启动成功了。
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/c4933b48e30c48768fe0d145eebfd883.png)
## 1.2数据模型
关系型数据库：
# 二、数据库操作
## 2.1使用案例
* 创建数据库

> create database text;

text是数据库名，也可以自己定义。
* 使用数据库

> use text;

* 创建数据库表
```
create table student(
	id int,
	name varchar(32),
	gender varchar(2)
);
```
* 查询表中的数据

> select *from student;

* 显示数据库
```
show database text;
```

## 3.1创建表
* 查询当前所有表

> show tables;

* 查看指定表结构
```
desc 表名；
```
* 查询指定表的建表语句
```
show create table 表名 ;
```

* 创建表结构
```cpp
CREATE TABLE 表名(
 字段1 字段1类型 [ COMMENT 字段1注释 ],
 字段2 字段2类型 [COMMENT 字段2注释 ],
 字段3 字段3类型 [COMMENT 字段3注释 ],
 ......
 字段n 字段n类型 [COMMENT 字段n注释 ]
) [ COMMENT 表注释 ] ;
```
例如：我们创建如下的表
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/892bfb0e700943a199caee3796b78711.png)
语句为：

```cpp
create table tb_user(
 id int comment '编号',
 name varchar(50) comment '姓名',
 age int comment '年龄',
 gender varchar(1) comment '性别'
) comment '用户表';
```

## 4.1表操作
### 添加字段
案例：
* 为emp表添加一个新的字段”昵称”为nickname，类型为varchar(20)

>  ALTER TABLE emp add nickname varchar(20) comment '昵称'，

### 修改
* 将emp表的nickname字段修改为username，类型为varchar(30)
```
ALTER TABLE emp CHANGE nickname varchar(30)  comment'昵称' ；
```

### 删除字段
 案例：
 * 将emp表的字段username删除
```
ALTER TABLE emp DROP username  ;
```

### 修改表名
> alter table 表明 rename to  新表明;

案例：
将emp 表的表名修改为 employ
```
ALTER TABLE emp RENAME TO employ
```

## 5.1表操作--删除
### 删除表
```
DROP TABLE [if exists] 表明;
```
案例：
如果tmp表存在，则删除tmp表
```
drop table if exists tmp;
```
删除指定表，并重新创建表
```
truncate table 表名;
```

注意：在删除表的时候，表中的全部数据也都会被销毁。
## 6.1 添加数据


```
insert into 表名(字段1，字段2，)  values(值1，值2，...);
```
案例：
```
insert into employee(id,workno,name,gender,age,idcard,entrydate)
values(1,'1','Itcast','男',10,'123456789012345678','2000-01-01');
```
查询数据库中的数据
```
select* from employee;
```
将全部字段添加数据
```
insert into employee values(2,'2','张无忌','男',18,'123456789012345670','2005-01-01');
```

批量添加数据
```
insert into employee values(3,'3','韦一笑','男',38,'123456789012345670','2005-01-01'),(4,'4','赵敏','女',18,'123456789012345670','2005-01-01');
```

注意事项：
> 插入数据时，指定的字段顺序需要与值的顺序一一对应
> • 字符串和日期型数据应该包含在引号中。
   • 插入的数据大小，应该在字段的规定范围内

## 修改数据
语法：

```
update 表名 set 字段1=值1，，....  [where 条件];

```
案例：
A：修改id为1的数据，将name修改为timi
```
update employee set name='timi'where id=1;
```
B. 修改id为1的数据, 将name修改为小美, gender修改为 女
```
update employee set name='小美',gender='女'where id=1;
```

注意：
>修改语句的条件可以有，也可以没有，如果没有条件，则会修改整张表的所有数据

### 删除数据
语法：
```
delete from 表名 [where 条件];
```
案例：
A：删除所有员工
```
delete from employee;
```

注意：
>• DELETE 语句的条件可以有，也可以没有，如果没有条件，则会删除整张表的所有数
据。
  • DELETE 语句不能删除某一个字段的值(可以使用UPDATE，将该字段值置为NULL即
可)。
  • 当进行删除全部数据操作时，datagrip会提示我们，询问是否确认删除，我们直接点击
Execute即可。

*** 