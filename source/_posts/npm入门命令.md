title: npm入门命令
comments: true
date: 2016-07-06 23:46:37
categories: 程序猿
tags: [npm, Nodejs]
sticky:
---
<br/>
<!--more-->

## 更新记录
2016-07-06 初稿

## 基础命令

显示npm版号
```
npm -v
# 或
npm version
```

安装模块
```
# 带-g为全局安装
# 本地安装：package会被下载到当前所在目录，也只能在当前目录下使用。
# 全局安装：package会被下载到到特定的系统目录下，安装的package能够在所有目录下使用。
npm install <package> -g
# 简写
npm i <package> -g
```

升级全局安装的指定模块
```
npm update <package> -g
```

升级当前目录下的指定模块
```
npm update <package>
```

升级当前目录下全部模块
```
npm update
```

升级node自身
```
# 安装一个叫n的模块
npm install -g n
# 升级到最新稳定版
n stable
# 升级到指定版本
n v0.10.26
```

卸载移除指定模块
```
npm uninstall <package>
# 别名：remove, rm, r, un, unlink
```

显示已安装模块
```
npm list
```

显示模块详细信息
```
npm show <package>
```

查看全局包安装路径
```
npm root -g
```

查看当前包安装路径
```
npm root
```

查看npm配置
```
npm config list
```
查看帮助
```
npm help
```

查看相关命令的帮助文档
```
npm help <command>
```
