---
title: Windows下安装MySQL
id: install-mysql-in-windows
date: 2019-07-23 19:30:48
updated: 2019-07-23 19:30:48
categories:
  - 人类的本质是复读机
tags:
  - mysql
---
今天给电脑安装个MySQL 5.7.27，以前一直下一步的安装包不见了，只找到了bin包安装的那种，看着有点懵...虽然搜索一下都能找到解决方法，还是记录一下。本次安装为windows 10 + MySQL 5.7.27-win64

<!-- more -->

##### 下载安装包

[下载地址](https://dev.mysql.com/downloads/mysql/5.7.html#downloads)

##### 解压到想要放置的位置

比如我就放到了 C:\Program Files (x86)\MySQL 下

##### 添加my.ini文件
```
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8 
[mysqld]
#设置3306端口
port = 3306 
# 设置mysql的安装目录
basedir=C:\Program Files (x86)\MySQL\mysql-5.7.27-winx64
# 设置mysql数据库的数据的存放目录
datadir=C:\Program Files (x86)\MySQL\mysql-5.7.27-winx64\data
# 允许最大连接数
max_connections=10
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
```
##### 添加环境变量

系统环境里加个名字叫MYSQL的变量，变量值是C:\Program Files (x86)\MySQL\mysql-5.7.27-winx64，然后在Path里加上%MYSQL%\bin

##### 安装MySQL（管理员身份）

进入C:\Windows\System32文件夹下，以管理员身份运行cmd.exe，输入命令
```
mysqld install
```
##### 初始化data目录（管理员身份）
```
mysqld --initialize
```
##### 启动服务（管理员身份）
```
net start mysql
```
##### 登录
```
mysql -u root
```
此时cmd显示Enter password，直接敲Enter键。

如果你到这里幸运的进去了，恭喜你，因为我就没进去！提示
> ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)

如果你跟我一样的话，接着执行以下步骤。

##### 停止服务（管理员身份）
```
net stop mysql
```
##### 安全模式打开（管理员身份）
```
mysqld --skip-grant-tables
```
这个时候，光标会一直闪，让它闪，不要动

##### 再次登录

打开另一个终端窗口，输入命令，回车，这次应该进去了
```
mysql -u root -p
```
##### 修改密码
```
use mysql;
// 5.7.11 版本之前
update user set password=password("123456") where user="root";
// 5.7.11 版本之后
update user set authentication_string=password("123456") where user="root";
```
修改完密码，两个终端窗口就都可以关掉了。重新以管理员身份打开一个新终端窗口，再次启动服务，再再次登录。

反正我到这里是完事了，你呢？


