---
title: 编程小技巧（2）：警惕MySQL中NOT IN的NULL值问题
id: tips-of-coding-2
date: 2019-04-12 17:26:52
categories:
  - 这就很有灵性了
tags:
  - mysql
---
本文使用 MySQL 版本：5.6.16

以下四条 SQL 语句中，哪些能查出 1 来？

```
SELECT 1 FROM dual WHERE 1 IN (1, null);
SELECT 1 FROM dual WHERE 1 IN (2, null);
SELECT 1 FROM dual WHERE 1 NOT IN (1, null);
SELECT 1 FROM dual WHERE 1 NOT IN (2, null);
```

答案是只有第 1 条能查出来。MySQL 的 NOT IN 中，如果子查询中返回的任意一条记录含有空值，则查询将不返回任何记录。（现象暂列于此，原因待查，若查到将补充于此）

解决方案是什么呢？使用 NOT EXISTS。

存在表 table\_a，列 id 中值为1，2， 3。

存在表 table\_b，列 id 中值为1，2，null。

```
SELECT a.id FROM table_a a WHERE NOT EXISTS (
    SELECT 1 FROM table_b b WHERE b.id = a.id
);
```
查询结果为 3
