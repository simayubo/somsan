---
title: 搭建自己的ngrok服务器实现内网穿透
tags:
  - ngrok
categories:
  - 服务器
abbrlink: e031ff1c
date: 2018-09-04 09:32:11
---

## 需求
本地开发调试时难免需要外网访问，比如支付，微信开发等等，以前用的是花生壳，但是免费版的又卡又不稳定，还好有个ngrok，但是官方的也是很卡，刚好这东西是开源的，你可以在自己的vps上搭建！
## 环境
Vps环境：CentOs 6.8 64位（我是在这个环境下搭建的）
## 解析域名
安装前请先把您的域名解析到服务器，例如：
ngrok.simayubo.cc =》 47.0.0.0
*.ngrok.simayubo.cc =》 47.0.0.0
## 安装git
这里安装2.6版本，防止会出现另一个错误


<!--more-->


安装git依赖
```base
yum -y install zlib-devel openssl-devel perl hg cpio expat-devel gettext-devel curl curl-devel perl-ExtUtils-MakeMaker hg wget gcc gcc-c++
```
下载git
```php
wget https://www.kernel.org/pub/software/scm/git/git-2.6.0.tar.gz
```
解压git
```base
tar zxvf git-2.6.0.tar.gz
```
编译git
```base
cd git-2.6.0
./configure --prefix=/usr/local/git
make
make install
```
创建git软连接
```base
ln -s /usr/local/git/bin/* /usr/bin/
```
## 安装go环境
准备go环境，32位的请选择386包，64位的请选择amd64 ([https://www.golangtc.com/static/go/][1])
这里选择的是1.9版本的64位：[https://www.golangtc.com/static/go/1.9/go1.9.linux-amd64.tar.gz][2]
下载安装包
```base
wget https://www.golangtc.com/static/go/1.9/go1.9.linux-amd64.tar.gz
```
解压并移动
```base
tar -zxvf go1.9.linux-amd64.tar.gz
mv go /usr/local/
```
创建软连接
```base
ln -s /usr/local/go/bin/* /usr/bin/
```
## 编译ngrok
```base
cd /usr/local/
git clone https://github.com/inconshreveable/ngrok.git
export GOPATH=/usr/local/ngrok/
export NGROK_DOMAIN="ngrok.simayubo.cc" #你的域名
cd ngrok
```
## 为域名生成证书
```base
openssl genrsa -out rootCA.key 2048
openssl req -x509 -new -nodes -key rootCA.key -subj "/CN=$NGROK_DOMAIN" -days 5000 -out rootCA.pem
openssl genrsa -out server.key 2048
openssl req -new -key server.key -subj "/CN=$NGROK_DOMAIN" -out server.csr
openssl x509 -req -in server.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out server.crt -days 5000
```
## 拷贝证书到指定位置
```base
cp rootCA.pem assets/client/tls/ngrokroot.crt
cp server.crt assets/server/tls/snakeoil.crt
cp server.key assets/server/tls/snakeoil.key
```
如果是国内服务器，需要修改下载源
```base
vi /usr/local/ngrok/src/ngrok/log/logger.go
log "github.com/keepeye/log4go"
```

## 编译ngrok服务端
```base
cd /usr/local/go/src
GOOS=linux GOARCH=amd64 ./make.bash  #这里请注意32位位386，64位位amd64
cd /usr/local/ngrok/
GOOS=linux GOARCH=386 make release-server
```
## 编译客户端
MAC OS
```base
cd /usr/local/go/src
GOOS=darwin GOARCH=amd64 ./make.bash #这里请注意32位位386，64位位amd64
cd /usr/local/ngrok/
GOOS=darwin GOARCH=amd64 make release-client
```
Windows
```base
cd /usr/local/go/src
GOOS=windows GOARCH=amd64 ./make.bash
cd /usr/local/ngrok/
GOOS=windows GOARCH=amd64 make release-client
```
## 服务端启动
```base
/usr/local/ngrok/bin/ngrokd -domain="$NGROK_DOMAIN" -httpAddr=":80"
```
后台启动：`setsid /usr/local/ngrok/bin/ngrokd -domain="$NGROK_DOMAIN" -httpAddr=":80"`

## 客户端使用
这里简单说明Windows
从服务器下载编译好的客户端，路径`/usr/local/ngrok/bin`
在客户端同目录新建文件`ngrok.cfg`，并粘贴下面内容：
```base
server_addr: "ngrok.simayubo.cc:4443"
trust_host_root_certs: false
```
按住Shift键，单击右键'在此处打开命令框',键入以下命令
```base
ngrok -config=./ngrok.cfg -subdomain=test 80
```
如果不出意外将会出现以下界面
![QQ截图20171005161952.jpg][3]

## 错误调试
#### Failed to read message: remote error: bad certificate
请重新生成证书，重新编译服务端和客户端，并且注意域名一定都要一样！
#### Failed to read valid http request: malformed HTTP request
检查 `ngrok.cfg` 文件里的端口号是否与 Listening for control and proxy connections on [::]:4443 后的端口号一致。


  [1]: https://www.golangtc.com/static/go/
  [2]: https://www.golangtc.com/static/go/1.9/go1.9.linux-amd64.tar.gz
  [3]: https://img.somsan.cc/images/2018/09/05/P5B.jpg