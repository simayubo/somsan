---
title: nginx反向代理websocket
tags:
  - socket
categories:
  - 服务器
abbrlink: efc4f59c
date: 2018-08-01 09:21:11
---

使用国外服务器，发现国内访问十分卡，想到使用阿里云香港vps做跳板，没想到效果很好，原本400多的延迟下降到了200多，但是会出现一个问题，就是源站websocket会报错，其实处理方法很简单，代码如下！

```conf
# websocket配置
upstream wsbackend {
  server 127.0.0.1:8888;
}

server
{
    listen 8888; # 监听端口
    server_name xxx.com;  # 实际访问域名
    location / 
    {
        # http代理
    	proxy_pass      http://111.111.111.111;  # 你的目标域名或ip
        proxy_set_header Host   $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        # websocket配置
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
   
}
```

