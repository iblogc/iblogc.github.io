title: ubuntu更新NV显卡驱动
date: 2015-03-27 20:26:27
categories: [其它]
tags: [Ubuntu, 驱动]
---

昨天在笔记本上新安装了ubuntu14.04LTS，顺便更新了下NV驱动，这里做下记录。<!--more-->

## 下载驱动
首页在[http://www.geforce.cn/drivers](http://www.geforce.cn/drivers)选择相应的显卡型号下载对应的驱动，下载完成后重命令为NVIDIA.run

## 关闭X server
输入`sudo /etc/init.d/gdm stop`或`sudo /etc/init.d/lightdm stop`停止X server，这时桌面会消失，按Ctrl+Alt+F1进入文本模式

### 安装驱动
进入驱动所在文件夹，执行`sudo sh NVIDIA.run`，安装驱动过程中会有几次对话框需要确认。

## 启动GDM
`sudo /etc/init.d/gdm start`或`sudo /etc/init.d/lightdm start`

## 重启电脑
sudo reboot

这样NV的驱动就安装好了。

