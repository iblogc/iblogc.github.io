---
title: Apache Rewrite
categories: [程序猿]
tags: [Apache, 重定向]
date: 2015-05-18 21:46:34
---
本文是对今天使用Apache的Rewrite技术做一个简单的记录。
> Apache的rewrite模块，提供了一个基于规则的重写(rewrite,也许译为重构更为合适)引擎，来实时重写发送到Apache的请求URL。因功能极其强大，被称为URL重写的“瑞士军刀”。

这个模块使用一个基于正则表达式解析器开发的重写引擎，根据web管理员定义的规则来实时(on the fly)重写请求URL。它支持任意数目的重写规则，以及附加到一条规则上的任意数目的规则条件，从而提供了一套非常灵活和功能强大的URL处理机制。 URL处理操作的实施与否，依赖于各种各样的条件检查，如检查服务器变量、环境变量、HTTP头字段、时间戳的值，甚至外部数据库的检索结果。这个模块可 以在服务器范围内(http.conf)、目录范围内(.htaccess)或请求串(query-string)的一部分处理有关的URL。重写的结果 URL，可以指向一个站内的处理程序、指向站外的重定向或者一个站内的代理。与灵活和功能强大相随的是设置的复杂。
<!--more-->

## 更新历史
2015年05月18日 - 初稿

## 开启模块
在`http.conf`中找到
```
# LoadModule rewrite_module modules/mod_rewrite.so
```
取消注释

## 定义规则
在`http.conf`中加入下列代码（如果启用了`httpd-vhosts.conf`，请在`httpd-vhosts.conf`里做配置）
```
<IfModule rewrite_module>
    RewriteEngine on
    RewriteCond %{HTTP_HOST} ^www.a.com [NC]
    RewriteRule ^/(.*) http://www.b.com/$1 [R=301,l]
<IfModule>
```

`RewriteCond`义重写发生的条件，在一条RewriteRule指令前面可能会有一条或多条RewriteCond指令，只有当自身的模板(pattern)匹配成功且这些条件也满足时规则才被应用于当前URL处理，上面代码的
`NC`：不区分大小写
`RewriteRule`满足`^/(.*)`此规则的所有URL都重定向到`http://www.b.com/$1`，`$1`使用前面`(.*)`匹配后的字符填充

所以前面的规则就是的最终效果是访问`www.a.com`的所以页面都会被重定向到`www.b.com`相应路径下的页面

## 参考
> [http://blog.chinaunix.net/uid-20639775-id-154471.html](http://blog.chinaunix.net/uid-20639775-id-154471.html)
> [http://man.lupaworld.com/content/manage/Apache2.2_chinese_manual/mod/mod_rewrite.html](http://man.lupaworld.com/content/manage/Apache2.2_chinese_manual/mod/mod_rewrite.html)
> [http://httpd.apache.org/docs/current/mod/mod_rewrite.html](http://httpd.apache.org/docs/current/mod/mod_rewrite.html)