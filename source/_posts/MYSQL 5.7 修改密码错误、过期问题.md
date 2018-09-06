---
title: MYSQL 5.7 修改密码错误、过期问题
tags:
  - mysql
categories:
  - 服务器
abbrlink: b6e52540
date: 2018-07-27 18:12:11
---

## 报错：
```
ERROR 1862 (HY000): Your password has expired. To log in you must change it using a client that supports expired passwords.
```
错误1862(HY000):你的密码已经过期。登录必须改变它使用一个客户端,支持过期的密码。

## 解决办法：
- 首先关闭Mysql进程；
- cmd进入mysql的bin目录下，运行
```
mysqld --skip-grant-tables
```
- 再在bin目录下打开cmd，运行如下进入数据库
```
mysql
```
- 选择数据库
```
> use mysql;
```
- 查看数据
```
> select * from mysql.user where user='root' \G
```
- 将密码过期修改为 密码不过期
```
> UPDATE user SET `password_expired`='N' where user='root';
```
- 修改密码
```
> UPDATE user SET `authentication_string` = PASSWORD('root');
```
- 退出
```
> exit;
```
- 重启数据库
