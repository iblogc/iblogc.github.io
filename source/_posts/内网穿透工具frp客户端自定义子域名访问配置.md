---
title: 内网穿透工具frp客户端自定义子域名访问配置
date: 2017-08-16 16:36:56
tags: [教程, 内网穿透]
categories: [程序猿]
---
<br />
<!--more-->
## 前提
A: 公网电脑
B: 内网电脑

## 下载
从[releases]([Releases · fatedier/frp · GitHub](https://github.com/fatedier/frp/releases))下载系统对应的压缩包，Mac可使用`darwin amd64`的包，在公网电脑和本地电脑各放一份。

## 配置
公网电脑上`frps.ini`
```
[common]
# 用于接收 frpc 连接的端口
bind_port = 7000
# 通过此端口访问http服务
vhost_http_port = 8080
# 日志文件输出位置
log_file = ./frps.log
# 日志等级
log_level = info
# 域名
subdomain_host = example.com
# frp管理后台端口
dashboard_port = 7500
# frp管理后台用户名
dashboard_user = admin
# frp管理后台密码
dashboard_pwd = admin
```

本地电脑上`frpc.ini`
```
[common]
# 公网电脑IP
server_addr = 111.111.111.111
# frp连接的端口
server_port = 7000

[web]
type = http
# 本地http服务端口
local_port = 8080
# 子域名前缀, 子域名前缀里不要使用下划线"_"，不然可能会出现莫名其妙的400错误可以用"-"代替。
subdomain = iblogc
```

配置域名`example.com`的A记录的泛解析
`*.example.com`指向公网电脑IP`111.111.111.111`

## 运行
1. 在内网电脑B上`8080`端口运行`http`服务
2. 在公网电脑上运行（Windows电脑上运行请去掉`./`）
```
./frps -c ./frps.ini
```
3. 在本地电脑上运行（Windows电脑上运行请去掉`./`）
```
./frpc -c ./frpc.ini
```

## 成功
在任何一台能联网的机器上访问 `http://iblogc.example.com:8080` 即可访问内网电脑B上的http服务。
在任务一台能联网的机器上访问`111.111.111.111:7500`即可访问frp的管理后台。

## frps服务端与nginx可共用80端口

```
server {
       listen 80;
       server_name *.example.com;
       location / {
           proxy_pass http://127.0.0.1:8080;
           proxy_redirect http://$host/ http://$http_host/;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header Host $host;
       }
}
```