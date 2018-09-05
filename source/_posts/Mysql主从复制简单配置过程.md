---
title: Mysql主从复制简单配置过程
date: 2018-09-04 09:31:11
tags: [mysql, 主从复制]
categories: [服务器]
---

Mysql主从复制在数据库优化方面是一个很常用的方法，数据库的读写分离、分布式都需要多个服务器之间数据库数据同步，比如读写分离，主数据库负责写数据，其它从服务器负责读数据，这样很大避免了高并发写数据库锁库、数据库负载过高等问题。
## 【准备】
### 主数据库：
```
IP：192.168.199.100
Mysql版本：5.5
操作系统：CentOS 7.1
```
### 从数据库：
```
IP：192.168.199.227
Mysql版本：5.5
操作系统：CentOS 7.1
```
### 其它
- 保持mysql版本一致性
- 如果你的系统开启了防火墙，请将数据库端口（3306）添加白名单，不然无法通信。
<!--more-->

## 【配置主数据库】
### 创建一个新用户: 
```shell
create user master
```
### 授权该用户访问权限:
```shell
GRANT REPLICATION SLAVE ON *.* TO 'master'@'192.168.%.%' IDENTIFIED BY 'mysql';
```
> 用户必须具有REPLICATION SLAVE权限，除此之外没有必要添加不必要的权限，密码为mysql。说明一下192.168.%.%，这个配置是指明master用户所在服务器，这里%是通配符，表示192.168.0.0-192.168.255.255的Server都可以以master用户登陆主服务器。当然你也可以指定固定Ip。

### 找到MySQL安装文件夹修改my.cnf文件,在[mysqld]下面增加下面几行代码
```bash
server-id=1   #给数据库服务的唯一标识，必须唯一
log-bin=master-bin
log-bin-index=master-bin.index
```
### 执行 show master status;
```bash
mysql> show master status;
+-------------------+----------+--------------+------------------+
| File              | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+-------------------+----------+--------------+------------------+
| master-bin.000005 |      796 |              |                  |
+-------------------+----------+--------------+------------------+
1 row in set (0.00 sec)
```
> 这里一定要记住File和Position，在从数据库中要用到，我这里分别是 master-bin.000005 和 796
### 重启MySQL服务

## 【配置从数据库】

### 打开my.cnf配置文件，在[mysqld]下加入
```ini
server-id=2 #必须唯一
relay-log-index=slave-relay-bin.index
relay-log=slave-relay-bin
```
### 重启MySQL服务
### 执行
```shell
change master to master_host='192.168.199.100', #ip为主数据库的ip地址
master_port=3306, #主数据库端口
master_user='master',  #上面添加的主数据库用户
master_password='mysql',  #主数据库密码
master_log_file='master-bin.000005', #主服务器产生的日志
master_log_pos=796;
```
### 启动slave
```shell
mysql> stop slave;
Query OK, 0 rows affected (0.01 sec)
```

## 【到了这个步骤基本就已经完成了】
可以运行下面命令来验证状态
```shell
mysql> show slave status\G 
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.199.100
                  Master_User: master
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: master-bin.000005
          Read_Master_Log_Pos: 492
               Relay_Log_File: localhost-relay-bin.000002
                Relay_Log_Pos: 254
        Relay_Master_Log_File: master-bin.000005
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 492
              Relay_Log_Space: 414
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 1
1 row in set (0.00 sec)
```
关键是
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
这两个状态

Slave_IO_Running: No 一般是因为防火墙没有添加mysql端口白名单，其它错误请查看Last_IO_Error
Slave_SQL_Running: No 可能是因为两个数据库数据不一致，可以重新设定从数据库对应主数据库的 Position （show master status;）

这时候你可以在主数据库新建数据库，添加数据表等操作都会同时同步到从数据库
![QQ截图20171022233817.jpg][1]


  [1]: https://img.somsan.cc/image/VJg