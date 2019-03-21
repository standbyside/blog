---
title: 探索 RabbitMQ（1）：简介与安装
id: explore-rabbitmq-1
date: 2019-01-17 15:32:44
updated: 2019-01-17 15:32:44
categories:
  - 人类的本质是复读机
tags:
  - rabbitmq
---

## 写在前面

本系列作为本人学习RabbitMQ的学习笔记，边学边写，主要记录了一个探索的过程，所以可能文章结构并不科学或者系统，想要系统地了解RabbitMQ相关内容还是建议去看书或者其他资料之类的。反正我就写个自己开心。如有错误或者疑惑，欢迎讨论指教。

本系列演示代码均基于 rabbitmq-3.7.1 + java 8 + springboot 2.4.0

演示代码GitHub地址：https://github.com/standbyside/quick-spring-boot

资料参考：
- [《RabbitMQ实战指南》](https://book.douban.com/subject/27591386/)
- 一篇博客，地址忘了，找不到了...

<!-- more -->

## RabbitMQ简介

自己搜索一下

## Mac下安装RabbitMQ

### 进入[官网](http://www.rabbitmq.com/install-standalone-mac.html)下载并解压

### 设置环境变量
```
export RABBIT_HOME=/Users/zn/software/rabbitmq_server-3.7.10
# 注意这里是sbin，不是bin
export PATH=${PATH}:${RABBIT_HOME}/sbin
```

### 启动server（默认端口5672）

```
// 以窗口进程进行启动
rabbitmq-server
// 以后台进程进行启动
rabbitmq-server -detached
```
### 启动界面管理

进入${RABBIT_HOME}/sbin路径，输入
```
sudo ./rabbitmq-plugins enable rabbitmq_management
```
执行一次以后不用再次执行

执行后在浏览器里输入localhost:15672可进入管理界面

用户名：guest，密码：guest

### 查看状态

```
rabbitmqctl status
```

### 关闭server

```
// 前台进程状态下
ctrl + c
// 后台进程状态下
rabbitmqctl stop
```

