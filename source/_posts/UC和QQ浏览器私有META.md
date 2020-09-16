---
title: "UC和QQ浏览器私有META"
date: 2015-05-06 00:43:08
categories: [网站]
tags: [meta, 浏览器, 前端]
---
## 什么是META
> META是HTML语言中的一个标签，也称作元标记。<META> 元素可提供有关页面的元信息（meta-information），比如针对搜索引擎和更新频度的描述和关键词。<meta> 标签位于文档的头部，不包含任何内容，<meta> 标签的属性定义了与文档相关联的名称/值对。
<!--more-->
## UC的私有META

### 屏幕方向

强制横屏/强制竖屏
```
<meta name="screen-orientation" content="landscape/portrait">
```

### 全屏
```
<meta name="full-screen" content="yes">
```

### xhtml适应屏幕排版

默认值为uc-fitscreen=no，即不启用此功能，此时浏览器的缩放行为与标准一致。当设置为uc-fitscreen=yes，则当进行缩放操作时，仅放大图片和文字等页面元素，但不放大屏幕宽度，从而避免了左右滚动条的产生。

```
<meta name="viewport" content="uc-fitscreen=yes"/>

```

### 排版模式
Uc浏览器提供两种排版模式，分别是适屏模式及标准模式，其中适屏模式简化了一些页面的处理，使得页面内容更适合进行页面阅读、节省流量及响应更快，而标准模式则能按照标准规范对页面进行排版及渲染。通过新定义的标签及js api接口，可以让网页设计者执行决定采用何种排版方式向用户展现页面。
```
<meta name="layoutmode" content="fitscreen/standard" />
```

### 夜间模式

允许进入夜间模式/禁止进入夜间模式
```
<meta name="nightmode" content="enable/disable"/>
```

### 强制显示图片，不受浏览器无图设置影响
```
<meta name="imagemode" content="force"/>
```

### 应用模式
默认将全屏，禁止长按菜单，禁止手势，标准排版
```
<meta name="browsermode" content="application"/>
```


## QQ浏览器的私有META

### 屏幕方向

强制横屏/强制竖屏/自动（默认）
```
<meta name="x5-orientation" content="landscape/portrait/auto"/>

```

### 全屏

强制全屏/跟随浏览器（默认）
```html 
<meta name="x5-fullscreen" content="true/auto"/>
```

### 页面模式
普通浏览模式（默认）/网页应用模式（定制工具栏，全屏显示）
```html
<meta name="x5-page-mode" content="default/app"/>
```

## 参考资料
> http://www.uc.cn/business/developer/
> http://open.mb.qq.com/doc?id=1201#_1




