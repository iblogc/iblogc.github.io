---
title: Ubuntu安装Android Studio出现错unable to run mksdcard sdk tool
date: 2015-03-28 17:54:03
categories: [程序猿]
tags: [Ubuntu, Android]
---
错误信息：
`unable to run mksdcard sdk tool`
原因：缺少库文件
解决方法：
`sudo apt-get install lib32z1 lib32ncurses5 lib32bz2-1.0 lib32stdc++6`