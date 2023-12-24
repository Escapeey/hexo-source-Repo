---
title: 数据库系统3-SQL语言
abbrlink: d733c799
date: 2023-11-19 17:00:46
tags:
---
<a url="https://www.cnblogs.com/rqy0526/p/11015943.html#:~:text=%E6%95%B0%E6%8D%AE%E5%BA%93%E6%93%8D%E4%BD%9C%E7%9A%84%E5%9F%BA%E6%9C%AC%E8%AF%AD%E6%B3%95%E5%A4%A7%E5%85%A8%201%201.%20C%20%28Create%29%3A%E5%88%9B%E5%BB%BA%201.%20%E8%AF%AD%E6%B3%95%EF%BC%9A%20create,%E5%88%97%E5%90%8D%20%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%3B%204.%20...%204%204.%20D%20%28Delete%29%3A%E5%88%A0%E9%99%A4">数据库操作的基本语法大全</a>

# 1 SQL概述
SQL(Structured Query Language):结构化查询语言，是关系数据库的标准语言
## 1.1 SQL的特点
- 综合统一
    - 集DDL、DML、DCL于一体
- 高度非过程化
- 面向集合的操作方式
- 以同一种语法结构提供两种使用方式
    - 可交互式和嵌入式使用
- 以简捷的自然语言作为操作语言

## 1.2 SQL语言所使用的动词
|SQL功能| 动词|
|:-----:|:-----:|
|数据查询|SELECT|
|数据定义|CREATE、DROP、ALTER|
|数据操纵|INSERT、UPDATE、DELETE|
|数据控制|GRANT、REVOKE|
## 1.3 SQL支持数据库三级模式体系结构
<img src="..\..\img\DB\SQL-支持数据库三级模式体系结构.png" width="70%" height="70%" align="middle">

# 2 数据定义
- SQL提供了专门的语言用来定义数据库、表、索引等数据库对象，这些语言被称作**数据库定义语言DDL**
- SQL的数据库定义语句
<img src="..\..\img\DB\SQL-数据库定义语句.png" width="70%" height="70%" align="middle">




## 2.1 创建基本表
<img src="..\..\img\DB\SQL-创建基本表.png" width="70%" height="70%" align="middle">

### 2.1.1 常用完整性约束
- 主码约束：**PRIMARY KEY**
- 唯一性约束：**UNIQUE**
- 非空值约束：**NOT NULL**
- 参照完整性约束：
    - **FOREIGN KEY<列名>REFERENCES<表名>(<列名>)**
- 用户定义完整性约束：
    - **Check(<约束条件表达式>)**
- 缺省值约束：Default
    - **Default<缺省值>**

### 2.1.2 SQL中数据类型
<img src="..\..\img\DB\SQL-数据类型.png" width="70%" height="70%" align="middle">


## 2.2 索引



# 查询
# 实例分析
# 数据更新
# 视图
# 嵌入式SQL

