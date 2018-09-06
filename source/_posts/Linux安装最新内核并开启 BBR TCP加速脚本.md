---
title: Linux安装最新内核并开启 BBR TCP加速脚本
date: 2018-08-10 09:31:11
tags: 
    - BBR
    - TCP加速
categories:
    - Linux
---

最近，Google 开源了其 TCP BBR 拥塞控制算法，并提交到了 Linux 内核，最新的 4.11版内核已经用上了该算法。根据以往的传统，Google 总是先在自家的生产环境上线运用后，才会将代码开源，此次也不例外。
根据实地测试，在部署了最新版内核并开启了 TCP BBR 的机器上，网速甚至可以提升好几个数量级。
这个脚本是网友提供的一个一键脚本，挺好用的。
<!-- more -->

> 本脚本适用环境
系统支持：CentOS 6+，Debian 7+，Ubuntu 12+
虚拟技术：OpenVZ 以外的，比如 KVM、Xen、VMware 等
内存要求：≥128M
日期　　：2017 年 05 月 15 日

## 关于本脚本
1. 本脚本已在 Vultr 上的 VPS 全部测试通过。（博主是在virmach上的128M的KVM，Ubuntu 14.04 x64上安装的）
2. 当脚本检测到 VPS 的虚拟方式为 OpenVZ 时，会提示错误，并自动退出安装。
3. 脚本运行完重启发现开不了机的，打开 VPS 后台控制面板的 VNC, 开机卡在 grub 引导, 手动选择内核即可。
4. 由于是使用最新版系统内核，最好请勿在生产环境安装，以免产生不可预测之后果。

## 使用方法
使用root用户登录，运行以下命令：

```bash
wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh
chmod +x bbr.sh
./bbr.sh

```
安装完成后，脚本会提示需要重启 VPS，输入 y 并回车后重启。
重启完成后，进入 VPS，验证一下是否成功安装最新内核并开启 TCP BBR，输入以下命令：

```bash
uname -r
```

查看内核版本，含有 4.11 就表示 OK 了

```bash
sysctl net.ipv4.tcp_available_congestion_control
```

返回值一般为：
net.ipv4.tcp_available_congestion_control = bbr cubic reno

```bash
sysctl net.ipv4.tcp_congestion_control
```

返回值一般为：
net.ipv4.tcp_congestion_control = bbr

```bash
sysctl net.core.default_qdisc
```

返回值一般为：
net.core.default_qdisc = fq

```bash
lsmod | grep bbr
```
返回值有 tcp_bbr 模块即说明bbr已启动。

加速后效果（1080p都毫无压力）

----------
## OVZ已有解决方案(已打上魔改bbr)：

```bash
wget https://raw.githubusercontent.com/kuoruan/shell-scripts/master/ovz-bbr/ovz-bbr-installer.sh
chmod +x ovz-bbr-installer.sh
./ovz-bbr-installer.sh
```

已测试通过的系统： Ubuntu 14.04 x64、Ubuntu 16.04 x64、CentOS 6 x64、CentOS 7 x64 只支持 64 位系统，要求 glibc 版本 2.14 以上。

需要配置的有如下几个选项：
1. 需要加速的端口，即的 SS 端口。加速开启之后，流量会先经过 BBR 处理，之后再发送给后端的 SS。
2. 可能需要配置 “公网接口名称”，即你服务器上具有公网 IP 的接口名称。搬瓦工 OpenVZ 上默认都是 venet0，但是有朋友可能需要安装在其他服务器上，所以我加入了此选项。

需要注意的是，在有 firewalld 的服务器上安装的时候，firewalld 会干扰 iptables 的规则，造成网络不通（现在具体原因未知，谁有解决方案可以提示一下）。所以在装有 firewalld 的服务器上需要先退出 firewalld：

```
systemctl disable firewalld
systemctl stop firewalld
```

如需卸载，请使用：
```
./ovz-bbr-installer.sh uninstall
```
