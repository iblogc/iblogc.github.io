---
title: JS笔记
date: 2019-04-24 16:35:13
categories: [程序猿]
tags: [js]
---
js中要用变量作为key的话使用方括号括住
例：`this.searchKeyword`
```javascript
this.$http({
    url: this.searchUrl,
    method: this.remoteRequestMethod,
    params: Object.assign({}, this.searchParams, this.pager),
    data: Object.assign({ [this.searchKeyword]: query }, this.searchBody, this.pager)
    })
```
<!--more-->
全文完🙈