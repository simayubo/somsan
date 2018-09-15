---
title: 解决ssh连接很慢的问题
date: 2018-09-09 19:38:13
tags: [Linux]
categories: [Linux]
---
## 问题
SSH长时间才能连上，会长时间卡在以下代码:
```bash
Connecting to 192.168.137.101:22...
Connection established.
To escape to local shell, press 'Ctrl+Alt+]'.
```

## 原因
> 文档解释: UseDNS Specifies whether sshd should look up the remote host name and check that the resolved host name for the remote IP address maps back to the very same IP address. The default is “yes”.

UseDNS默认值为yes，这个选项打开的状态下，当客户端试图登录SSH服务器时，服务器端先根据客户端的IP地址进行DNS反向查询出客户端的主机名，然后根据查询出的客户端主机名进行DNS正向记录查询，验证与其原始IP地址是否一致，所以在登陆的时候会出现卡顿。

## 解决
设置SSH配置项UseDNS no

```bash
vi /etc/ssh/sshd_config
# 将#UseDNS yes改为
UseDNS no
```
然后重启sshd服务，问题解决。
```bash
service sshd restart
# CentOS 7 命令: /bin/systemctl restart sshd.service
```
