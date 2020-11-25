---
title: 使用Synergy实现多台电脑共享一套键鼠
date: 2015-08-29 23:52:41
categories: [优化辅助]
tags: [Synergy, 共享, 键盘]
---
![Synery-log.png](/media/Synergy-logo.png)

因为家里有一台台式和一台笔记本，台式`Windows10`为日常使用，笔记本`Ubuntu&Windows7`，以前一直在`Windows`上敲代码，现在正转向`Ubuntu`，但家里桌子上摆放两台电脑已经有点挤了，如果再来两套键鼠那就成二手电脑配件甩卖铺了，所以上网查了下看看有没有软件能实现两台电脑共享一套键盘的，还真找到了一款叫`Synergy`的软件，支持`Windows`, `Mac OS X`, `Linux `三大系统，软件是下载收费，使用使用免费，官网上标明基础版$10，高级版$29。下面我说说我自己的配置过程。
<!--more-->

## 安装
在`Ubuntu`里打开终端，输入以下命令进行安装
```
sudo apt-get install synergy
```
在`Windwos`上双击安装。

*`Ubuntu`我安装的是1.6.2，`Windwos`上是1.7.4 x64*

## 配置

### 服务器设置
> `synergy`需要一台电脑做为服务端，其它电脑做为客户端来连接服务端。
> 本来是我想选择`Ubuntu`做为服务端的，但设置好后链接失败提示为`WARNING: failed to connect to server: incompatible client 1.5`，似乎是不兼容，但我的客户端版本是`1.6.2`不是`1.5`啊，所以作罢，只得选用`Windwos`来做服务端。

1. 我们选用`Windwos`来做服务端，在`Windwos`打开软件，选择「server」；
Synergy-Windows服务端配置首页.png)

2. 点击「设置服务端」进行添加客户端操作；
Synergy-Windows服务端添加客户端-01.png)

3. 从右上角手动电脑图标到下方的格子里，这里的格式位置对应你当前几台电脑的实际的以我把电脑图标拖到中间左侧的格子里；
Synergy-Windows服务端添加客户端-02.png)

4. 双击电脑图标进行编辑，这里我们只需要输入客户端电脑的计算机名，其它都默认；
Synergy-Windows服务端添加客户端-03.png)

5. 设置好后点击再次ok，回到设置首页，点击「开始」启动服务端；
Synergy-Windows服务端启动.png)

### 客户端设置
1. 客户端比较简单，在`Ubuntu`上打开`Synergy`，选择「Client」，在`Server IP`里输入服务端的IP，点击「Start」即可；
Synergy-Ubuntu客户端配置首页.png)

### 开机自启
要在`Ubuntu`开机在登录界面前启动`synergy`，编辑`/etc/lightdm/lightdm.conf`文件添加`display-setup-script=/usr/bin/synergyc 192.168.9.102`，把`192.168.9.102`换成你自己的`synergy`服务端IP。

## 完成
到这里你就会发现你可鼠标可以在两个电脑屏幕上移动了，像我刚才配置的是在`Windows`左侧添加了`Ubuntu`，所以当我在`Windows`上把鼠标向左移动，并移到边界，再继续左移时，鼠标就会出现在`Ubuntu`屏幕上，键盘的行为跟随鼠标的，即鼠标在哪个屏幕，键盘输入就对应哪个屏幕的系统。
> 网上说`Synergy`支持在不同电脑间复制粘贴，目前我自己没有试成功，有知道朋友可以和说。

## 相关
项目主页：https://github.com/synergy/synergy

## 更新记录
2015-08-29 初稿
2015-09-03 补充`Ubuntu`客户端自启明说


