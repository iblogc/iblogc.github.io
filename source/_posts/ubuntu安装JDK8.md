---
title: ubuntu安装JDK8
date: 2015-03-28 17:29:42
categories: [程序猿]
tags: [Ubuntu, JDK]
---

## 下载JDK8
到[oracle](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)网站下载JDK8
<!--more-->
## 解压安装
```
sudo tar xzvf jdk-8u40-linux-x64.tar.gz
mkdir -p /usr/lib/jvm
sudo mv  /usr/lib/jvm jdk1.8.0_40 /usr/lib/jvm
cd /usr/lib/jvm
sudo ln -s jdk1.8.0_40 java-8
```

## 配置环境变量
添加PATH,CLASSPATH,JAVA_HOME环境变量
`gedit ~/.bashrc`
在打开的窗口里添加以下内容
```
export JAVA_HOME=/usr/lib/jvm/java-8
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```
保存退出，执行命令使配置生效
`source ~/.bashrc`

## 配置默认JDK版本
在有的系统中会预装OpenJDK，系统默认使用的是这个，而不是刚才装的。所以这一步是通知系统使用Oracle的JDK，非OpenJDK。
```
sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/java-8/bin/java 300
sudo update-alternatives --install /usr/bin/javac javac /usr/lib/jvm/java-8/bin/javac 300
sudo update-alternatives --config java
sudo update-alternatives --config javac
```

## 验证是否成功
`java -version`