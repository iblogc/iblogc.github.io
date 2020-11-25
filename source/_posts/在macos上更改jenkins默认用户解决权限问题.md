---
title: 在macOS上更改Jenkins的默认用户，解决权限问题
comments: true
date: 2017-08-24 14:29:14
tags: [Jenkins,macOS,权限,用户]
categories: [程序猿]
---
<br />
<!--more-->
在MacOS上使用`dmg`安装包安装完Jenkins之后，发了Jenkins自动在系统里新建了一个名为`jenkins`的用户。默认的，Jenkins程序里的自动化构建操作都是以这个用户身份来进行的，所以有时会出现一些权限问题，解决方法就是修改Jenkins配置文件，把Jenkins运行的默认账户改成平时用的账户。

```shell
#停止Jenkins
sudo launchctl unload /Library/LaunchDaemons/org.jenkins-ci.plist

# 修改Group和User
# <用户名>填写你的MacOS用户名，不知道的可以在命令行使用whoami查看，不需要尖括号
$ sudo vim +1 +/daemon +’s/daemon/staff/’ +/daemon +’s/daemon/<用户名> +wq org.jenkins-ci.plist

# 可能相应文件夹的权限
sudo chown -R <用户名>:staff /Users/Shared/Jenkins/
sudo chown -R <用户名>:staff /var/log/jenkins/

# 启动Jenkins
sudo launchctl load /Library/LaunchDaemons/org.jenkins-ci.plist
```