title: ubuntu下运行shadowsocks客户端
date: 2015-03-28 16:54:44
categories: [优化辅助]
tags: [Ubuntu, Shadowsocks]
---
Shadowsocks 是一个安全的socks5代理,用于保护网络流量,是一个开源项目,[项目地址](https://github.com/shadowsocks/shadowsocks)。<!--more-->

## 下载
```
sudo apt-get install python-pip python-m2crypto
sudo pip install shadowsocks
```
## 配置
`sudo gedit /etc/shadowsocks/config.json`
```
{
    "server":"remote-shadowsocks-server-ip-addr",
    "server_port":8883,
    "local_address":"127.0.0.1",
    "local_port":1080,
    "password":"abcdef",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open":false,
    "workers":1
}
```
> 请根据实际情况配置

## 启动客户端
`sslocal -c /etc/shadowsocks.json`

## 浏览器扩展
Firefox可使用FoxyProxy Standard
Chrome可使用Proxy SwitchOmega
配置请自行Google/百度