---
title: virtualbox下最小化安装centos7后上网设置
tags:
  - virtualbox
categories:
  - Linux
abbrlink: 92bf9786
date: 2018-09-01 21:31:10
---

## 问题
在虚拟机中以最小化方式安装centos7，后无法上网。
## 原因
因为centos7默认网卡未激活。
## 解决办法
编辑`ifcfg-enp0s3`
```bash
vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
```
将最后一行 `ONBOOT=no` 改为 `ONBOOT=yes`
保存并退出
```
esc -> !wq
```
重启网卡
```bash
service network restart
```
问题解决


<!--more-->


![QQ截图20170116195750.png][1]

## 另外
因为是最小化安装，此时ifconfig命令不能用，可用

命令：`ip addr`  查看分配网卡情况。

联网后可运行命令：
```bash
yum install net-tools
```
来安装ifconfig功能


  [1]: https://simayubocc.oss-cn-hangzhou.aliyuncs.com/img/2017/01/2379295140.png