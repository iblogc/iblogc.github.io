---
title: Android真机调试常用命令
comments: true
tags: [Android,macOS,命令,效率,无线,调试,adb,USB]
categories: [程序员]
date: 2019-04-28 20:39:32
---

使用USB连接Android真机调试时，使用无线连接调试会方便很多，并使用电脑端用adb命令实现截图和录屏，方便调试和问题反馈。
<!--more-->


## 无线调试

```
# 前提条件：手机和电脑处理同一网段
# 第一次手机先使用USB连接电脑执行以下命令让手机上的某一端口处于监听状态
adb tcpip <port>

# 在手机上查看ip地址或使用以下命令查看ip
adb shell ifconfig 
# 连接手机（在同一个环境下，一般手机/电脑不重启就会一直连接着）
adb connect <ip> :<port>
# 查看连接的设备
adb devices
```

## 截图

```
# 截图并保存到手机sd卡的下
adb shell screencap -p /sdcard/screenshot.png
```

便捷脚本（截图并自动复制到电脑剪切板/保存到电脑本地）

> 因脚本里调用了linux/macOS的命令，所以只适用于macOS系统，windows请自行修改脚本。

`vi shot.sh`

```
#!/bin/bash
# Android截图，定位和预览默认关闭，请取消注释

dd=`date +%Y-%m-%d-%H-%M-%S`
pwd=`pwd`
adb shell screencap -p /sdcard/screenshot.png
adb pull /sdcard/screenshot.png
adb shell rm /sdcard/screenshot.png
mv screenshot.png $dd.png
echo "截图已保存为当前目录下的"$dd.png
# 修改图片尺寸，长或宽最大不超过960，等比缩放
echo "压缩图片..."
sips -Z 960 $pwd/$dd.png
# 定位到文件
open ./$dd.png -R
# 打开预览
open -a Preview $dd.png
# 复制到剪切板
osascript -e 'on run args' -e 'set the clipboard to POSIX file (first item of args)' -e end $pwd/$dd.png
echo "截图已复制到剪切板"
```

授予执行权限
```
chmod a+x shot.sh
```

使用方法

```
./shot.sh
```

⌘+v试试

*可把命令添加alias别名*

## 录屏

```
# 执行录屏并保存到手机sd卡目录下（默认时长180s）
# 可配置参数
# --time-limit: 录制时长，单位秒
# --size: 分辨率，如1280*720，不指定默认使用手机的分辨率
# --bit-rate: 视频的比特率，如6Mbps为6000000
# --verbose: 命令行显示log
adb shell screenrecord /sdcard/demo.mp4
```

便捷脚本（录屏并自动复制到电脑剪切板/保存到电脑本地）

> 因脚本里调用了linux/macOS的命令，所以只适用于macOS系统，windows请自行修改脚本。


`vi record.sh`

```
#!/bin/bash
# Android录屏
dd=`date +%Y-%m-%d-%H-%M-%S`"-$1s"
pwd=`pwd`
adb shell screenrecord --time-limit $1 /sdcard/screenrecord.mp4
adb pull /sdcard/screenrecord.mp4
adb shell rm /sdcard/screenrecord.mp4
mv screenrecord.mp4 $dd.mp4
echo "$1秒视频已保存为当前目录下的"$dd.mp4
# 定位到文件
open ./$dd.mp4 -R
# 复制到剪切板
osascript -e 'on run args' -e 'set the clipboard to POSIX file (first item of args)' -e end $pwd/$dd.mp4
echo "$1秒视频已复制到剪切板"
```

授予执行权限
```
chmod a+x record.sh
```

使用方法
```
# 3为录制秒数，可修改
./record.sh 3
```