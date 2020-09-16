---
title: 使用ngrok配置微信公众号本地开发环境（ADSL屏蔽80端口问题）
date: 2015-02-04 23:35:40
categories: [程序猿]
tags: [微信公众号, 端口, ngrok, 内网穿透]
---
## 更新记录
2016-03-04 更新教程
2015-	09-16 添加旧版软件下载
2015-02-04 初稿

鉴于国内大部分ADSL屏蔽80端口，而微信公众号开发只支持80端口，所以在本地开发测试微信公众号就是一个问题了，这里我们可以使用软件ngrok来解决这个问题。<!--more-->

## 配置步骤

### 1. [注册ngrok账号](https://dashboard.ngrok.com/user/signup)
注册成功后拿到授权码`auth token`，使用ngrok时并不强制用户注册，但注册后会附加更多功能(如自定义二级域名)；

### 2. [下载ngrok](https://ngrok.com/download)，解压；

### 3. 启动
##### 方式一:
让本地的‘http://127.0.0.1:80’ 可以让外网访问
```yml
ngrok http 80
```
ngrok会随机分配一个二级域名，可直接通过外网可通过`http://xxxx.tunnel.mobi`来访问本机的`http://127.0.0.1:80`网站

##### 方式二：使用配置文件启动:
在`ngrok.exe`目录下执行命令（不带尖括号），生成配置文件（配置文件会在`C:\Users\用户名/.ngrok2/ngrok.yml`下「windows」）
```
ngrok authtoken <you authtoken>
```
修改配置文件，可配置多个tunnel（注意，配置文件是yaml格式，冒号后面如果还有内容需要加空格）
```
authtoken:<you authtoken>
tunnels:
  # 自定义隧道名 
  iblogc:
    #本地服务端口 
    addr: 4000
    # 用于http/https里的身份认证
    #auth: "username:password"
    proto: http
    # 二级域名，如果运行提示重复，换一个就行
    subdomain: iblogc
  django:
    addr: 8000
    auth: "abc:123456"
    proto: http
    subdomain: django
  weixin:
    addr: 80
    proto: http
    subdomain: weixin
```

现在执行
```
ngrok start iblogc
```

试试，如果你设置的的二级域名没有被占用的话，那么就会启动成功，否则请更换一个二级域重试。
`http://iblogc.ngrok.io` `https://iblogc.ngrok.io` 协议均可以访问。
ngrok-start-iblogc.png)

你也可以同时启动两个tunnel
```
ngrok start iblogc django weixin
```
ngrok-start-iblogc-django-weixin.png)

因为我的django tunnel配置文件里添加了`auth`配置所以访问`http://django.ngrok.io`需要输入用户名密码。
ngrok-auth.png)

假设`weixin`就是我本地跑在80端口的微信项目，现在就可以在微信公众平台「开发者中心」可以使用`weixin.ngrok.com`进行配置了，所有发向此域名的请求都会转发到你的本地`127.0.0.1:80`上。

### 4. 查看详细信息如果想查看详细的请求信息可以在浏览器里打开`http://127.0.0.1:4040`查看详细信息
nrok-web-interface.png)

### 5. 参考文档
官方文档：https://ngrok.com/docs

