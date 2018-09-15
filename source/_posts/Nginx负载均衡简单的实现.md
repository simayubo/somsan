---
title: Nginx负载均衡简单的实现
date: 2018-09-10 22:29:13
tags: [Linux, Nginx]
categories: [Linux]
abbrlink: efc4f59b
---

## 准备

VirtualBox创建三个虚拟主机如下：
> 负载主机：192.168.137.101
WEB主机: 192.168.137.102
WEB主机: 192.168.137.103

负载主机只需要安装nginx，WEB主机安装lnmp环境

## 配置

### 负载主机配置nginx
```smartyconfig
user  www www;

worker_processes auto;

error_log  /home/wwwlogs/nginx_error.log  crit;

pid        /usr/local/nginx/logs/nginx.pid;

#Specifies the value for maximum file descriptors that can be opened by this process.
worker_rlimit_nofile 51200;

events {
   worker_connections  1024;
}
http {
   include     mime.types;
   default_type application/octet-stream;
   sendfile        on;
   keepalive_timeout  65;
   log_format  main  '$remote_addr - $remote_user [$time_local]"$request" '
             '$status $body_bytes_sent"$http_referer" '
             '"$http_user_agent" "$http_x_forwarded_for"';
   upstream server_pools
   {
       #ip_hash;
       server 192.168.137.102:81;
       server 192.168.137.103:81;
   }
   server {
       listen  80;
       server_name 192.168.137.101;
       location / {
           proxy_pass http://server_pools;
           proxy_set_header Host $host;
           proxy_set_header X-Forwarded-For $remote_addr;
           proxy_set_header X-Real-IP $remote_addr;
       }
   }
}
```

<!--more-->

### WEB主机配置

WEB主机配置环境参考: https://lnmp.org/install.html
创建对应虚拟主机，监听81端口，域名绑定WEB主机的IP

```smartyconfig
server
{
    listen 81;
    #listen [::]:80;
    server_name 192.168.137.102 ;
    index index.html index.htm index.php default.html default.htm default.php;
    root  /home/wwwroot/192.168.137.102;

    include rewrite/none.conf;
    #error_page   404   /404.html;

    # Deny access to PHP files in specific directory
    #location ~ /(wp-content|uploads|wp-includes|images)/.*\.php$ { deny all; }

    include enable-php.conf;

    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
    {
        expires      30d;
    }

    location ~ .*\.(js|css)?$
    {
        expires      12h;
    }

    location ~ /.well-known {
        allow all;
    }

    location ~ /\.
    {
        deny all;
    }

    access_log  /home/wwwlogs/192.168.137.102.log;
}
```

WEB主机根目录上传php测试文件，文件内容：
```php
<?php

echo "主机：".$_SERVER['SERVER_ADDR'];
```

访问：`http://192.168.137.101`，
刷新访问，会发现随机打印 `主机：192.168.137.102` 和 `主机：192.168.137.103`
