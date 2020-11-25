---
title: GitHub Pages/GitCafe Pages绑定自定义域名
date: 2014-11-09 18:32:11
categories: [网站]
tags: [域名,GitHub,GitCafe]
---
## 更新记录
2015-01-01 更新 GitCafe-Page IP地址
2014-11-09 初稿

虽然GitHub Pages和GitCafe Pages默认为每个用户分配了一个二级域名（GitHub为`username.github.io`或`username.github.com`,GitCafe为`username.gitcafe.com`），但如果你对这个二级域名不满意也可以申请一个自己的域名进行绑定。下面就说说GitHub和GitCafe的绑定过程。
<!--more-->
## 准备工作
+ 域名（例：iblogc.com）
+ 一个GitHub Pages/GitCafe Pages

---
## GitHub
+ 在repo目录下创建一个名为`CNAME`的文件（无后缀）
+ 打开CNAME，在里面写入你要绑定的域名
 + ~~1)如果你绑定的是二级域名，请在域名管理里添加一条CNAME记录，指向username.github.io或username.github.com~~
 + ~~2)如果你绑定的是顶级域名，请在域名管理里添加一条A记录，指向103.245.222.133~~
 + 请在域名管理里添加一条CNAME记录，指向username.github.io
+ 等待生效 

---
## GitCafe
+ 打开你自己的gitcafe pages项目，
+ 进入 项目管理>>自定义域名，在这里添加你要绑定的域名就可以，比如我配置了顶级域名iblogc.com（当然也可以设置二级域名）
+ QQ截图20141109181543.png)
+ 在域名管理里添加一条CNAME记录，记录值为gitcafe.io，如果您的域名注册商不提供CNAME记录选项，请将A记录值修改为 207.226.141.135(IP地址截止2015-01-01有效，如失效，请以[官方说明](https://gitcafe.com/GitCafe/Help/wiki/Pages-%E7%9B%B8%E5%85%B3%E5%B8%AE%E5%8A%A9#wiki)为准)。
+ 等待生效


