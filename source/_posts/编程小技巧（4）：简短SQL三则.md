---
title: 编程小技巧（4）：简短SQL三则
id: tips-of-coding-4
date: 2019-07-22 09:29:58
updated: 2019-07-22 09:29:58
categories:
  - 这就很有灵性了
tags:
  - mysql
---
周末看了看LeetCode上的数据库题目，有些平常基本没用到，但是很基础的SQL，确实有些生疏，在此记录一下。

##### 1. 添加行号

[原题链接](https://leetcode-cn.com/problems/rank-scores/)

```sql
SELECT
    a.score,
    @rowNum := @rowNum + 1 AS rank
FROM scores a,
(SELECT @rowNum := 0) b
```

##### 2. 查询连续出现n次的数据

[原题链接](https://leetcode-cn.com/problems/consecutive-numbers/comments/)

```sql
SELECT 
    DISTINCT a.num 
FROM ( 
    SELECT 
        t.num,
        @count := IF (@pre = t.num, @count + 1, 1)  AS count,
        @pre := t.num AS pre
    FROM logs t, 
    (SELECT @pre := NULL, @count := 0) b 
) a 
WHERE a.count >= 3;
```

##### 3. 删除重复数据，只保留Id最小的那条

[原题链接](https://leetcode-cn.com/problems/delete-duplicate-emails/)

```sql
DELETE FROM person WHERE id NOT IN ( 
    SELECT t.id FROM (
        SELECT MIN(id) AS id FROM person GROUP BY email 
    ) t 
);
```

