---
title: Windows下安装Redis
id: install-redis-in-windows
date: 2019-09-16 16:19:06
updated: 2019-09-16 16:19:06
categories:
  - 人类的本质是复读机
tags:
  - redis
  - windows
---
继上次Windows下安装MySQL后，今天安装Redis发现也复杂许多，github上有几个仓库似乎提供了windows一键安装版，但是都到2016年就不更新了......所以Macbook真的好使 o(﹏)o

由于Redis本来就是不支持Windows的，为了安装Redis我们首先需要安装另一个工具：cygwin。这个工具能帮助我们在Windows上运行类UNIX模拟环境。

本文安装环境：Windows10 + redis-5.0.5 + cygwin-x86_64

<!-- more -->

### 下载cygwin

下载地址：https://cygwin.com/setup-x86.exe

### 安装cygwin

双击下载的.exe文件一路next下去就行了。

如果在选择镜像时没有加载出镜像列表，可以输入http://mirrors.aliyun.com/cygwin/，然后点击旁边的Add。

![image](http://cdn.standbyside.com/shortcut/cygwin-3.png)

到 Select Packages 时注意一下，这里我们要额外添加 make，gcc-core，gcc-g++，这三个默认是不安装的

![image](http://cdn.standbyside.com/shortcut/cygwin-6.png)

安装后，桌面上会出现一个快捷方式，双击图标就可以用。如果双击完提示"Windows正在查找mintty"，首先查看安装目录（比如我的就是C:\cygwin64）下的bin包里有没有mintty.exe文件，如果没有就重新安装。如果有查看下快捷键的目标里是不是"C:\cygwin64\bin\mintty.exe -i /Cygwin-Terminal.ico -"，尤其检查有没有.exe结尾。

![image](http://cdn.standbyside.com/shortcut/cygwin-4.png)

如果后面安装时发现有漏掉的命令没安装也没关系，重新双击.exe文件重新安装一遍就行，上一遍已经安装上的不会丢失，只需要找到你这次想补上的就可以。

后面步骤的所有命令行都在cygwin下运行。

### 下载Redis

下载地址：https://redis.io/download

### 编译Redis

首先将下载的redis进行解压，然后将里面deps文件夹下的hiredis删掉，从git上重新拉取一个新的（压缩包里的有问题，编译时报错）
```
git clone https://github.com/redis/hiredis.git
```
然后把deps目录下几个文件夹编译一下，在deps下执行

```
make hiredis jemalloc linenoise lua
```
然后回到 redis 根目录执行
```
make && make install
```
### 运行redis

到 src 目录下执行
```
redis-server
```
完美运行！

![image](http://cdn.standbyside.com/shortcut/cygwin-5.png)

