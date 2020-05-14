---
title: IDEA常见问题解决方法
id: faq-solution-with-idea
date: 2020-05-14 14:06:53
updated: 2020-05-14 14:06:53
categories:
  - 那些古怪又令人忧心的问题
tags:
  - idea
---

### 1. plugins 搜索不出来

#### 现象描述：

进入 plugin 的 Marketplace 无法所搜出任何插件

![image](http://cdn.standbyside.com/shortcut/idea-plugin-1.jpg)

#### 问题原因：

可能就是网络不好吧...

#### 解决方案：

进入`Appearance & Behavior -> System Settings -> Http Proxy` 勾选上 `Auto-detect proxy settings` 和 `Automatic proxy configuration URL`即可（有人说后面还要写上 `http://127.0.0.1:1080` 或者 `https://plugins.jetbrains.com`，我自己试是什么都没写就ok了）

![image](http://cdn.standbyside.com/shortcut/idea-plugin-2.jpg)

然后，重启试试 :)


