---
title: 编程小技巧（3）：查看优化后的SQL
id: tips-of-coding-3
date: 2019-06-19 22:51:25
updated: 2019-06-19 22:51:25
categories:
  - 这就很有灵性了
tags:
  - mysql
---
众所周知，当我们用MySQL执行SQL语句时，执行顺序不一定会像我们所写的那样运行。MySQL中存在优化器，会重写我们的SQL，那如何查看优化后的SQL呢？格式如下：

```
EXPLAIN <你的SQL>;
SHOW WARNINGS;
```
例如：

```
EXPLAIN SELECT name FROM t_product WHERE code IN (SELECT product_code FROM t_order);
SHOW WARNINGS;
```
其中第一句EXPLAIN是查看SQL执行计划的语句，通常用来查看MySQL如何使用索引来处理SELECT语句以及连接表

第二句SHOW WARNINGS表示显示上一句的警告，所以一定要和上一句一起运行，我们要查看的优化后语句就在SHOW WARNINGS运行后显示的result的message字段里

```
/* select#1 */ 
select `mytest`.`t_product`.`name` AS `name` 
from `mytest`.`t_product` 
semi join (`mytest`.`t_order`) 
where (`mytest`.`t_product`.`code` = convert(`mytest`.`t_order`.`product_code` using utf8mb4))
```
明显看出，最终执行时使用的是semi join半连接操作（MySQL 8.0.15）。
