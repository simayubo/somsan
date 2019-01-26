---
title: Docker容器开启ssh
date: 2019-01-15 19:38:13
tags: 
   - Linux
   - docker
categories: [docker]
---
## 问题
默认docker镜像是没有ssh的，需要我们去手动安装开启

## 解决
#### 安装openssl,openssh-server

```bash
$ yum -y update
$ yum install passwd openssl openssh-server -y
```

#### 启动sshd
```bash
$ /usr/sbin/sshd -D &
```
启动需要pid文件存在，可以创建/var/run/ssh
```bash
$ /usr/sbin/sshd
```
执行上面命令会报错:
```bash
Could not load host key: /etc/ssh/ssh_host_rsa_key
Could not load host key: /etc/ssh/ssh_host_ecdsa_key
Could not load host key: /etc/ssh/ssh_host_ed25519_key
```
解决办法
```bash
 $ ssh-keygen -q -t rsa -b 2048 -f /etc/ssh/ssh_host_rsa_key -N ''
 $ ssh-keygen -q -t ecdsa -f /etc/ssh/ssh_host_ecdsa_key -N ''
 $ ssh-keygen -t dsa -f /etc/ssh/ssh_host_ed25519_key -N ''
```
#### 修改 /etc/ssh/sshd_config 配置信息

去掉注释 `Port`，`ListenAddress`，`ListenAddress`
`UsePAM yes` 改为 `UsePAM no`
`sePrivilegeSeparation sandbox` 改为 `UsePrivilegeSeparation no`

执行：
```bash
$ sed -i "s/#UsePrivilegeSeparation.*/UsePrivilegeSeparation no/g" /etc/ssh/sshd_config
$ sed -i "s/UsePAM.*/UsePAM no/g" /etc/ssh/sshd_config
```

修改完后，重新启动sshd
```bash
$ /usr/sbin/sshd -D &
```