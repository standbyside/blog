---
title: 编程小技巧（1）：随机获取boolean值
id: tips-of-coding-1
date: 2019-03-20 16:27:45
updated: 2019-03-20 16:27:45
categories:
  - 这就很有灵性了
tags:
  - java
  - 随机
---

今天看同事写的预览合同模版代码。业务是在合同模版配置好后，需要预览展示一下合同生成效果，这时比如甲方乙方姓名等值，需要给一些默认值。有一些boolean属性，他写的按照随机给值。

一般来说最先想到什么呢？获取随机数判断是否满足某个条件。

```
boolean b = new Random().nextInt(1) == 0;
```
既然看到了 Random，自然就会想有没有直接的随机获取boolean值方式呢？有的。

```
boolean b = new Random().nextBoolean();
```
但是"年会现场review抽奖代码"事件告诉我们，这些都不是真随机。

那这位同事是怎么写的呢？

```
boolean b = (System.currentTimeMillis() & 1) != 1
```
1的二进制是00000001，根据&运算的定义，两个位全部为1结果才为1，也就是无论 System.currentTimeMillis() 是多少，运算结果只能是0或1。

当 System.currentTimeMillis() 为偶数时，最右位是0，& 运算结果是0，b为true。

当 System.currentTimeMillis() 为奇数时，最右位是1，& 运算结果是1，b为false。

本质上也是一个奇偶判断，我感觉挺随机的。
