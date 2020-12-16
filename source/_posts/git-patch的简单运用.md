---
title: Git patch的简单运用
date: 2015-10-15 20:49:40
categories: [程序猿]
tags: [Git]
---

## 生成PATCH

往前n个提前内容的patch
```
git format-patch -n
```

某个commit（含）的及之前的n-1次提交的patch
```
git format-patch -n SHA
```
<!--more-->
某个commit的patch
```
git format-patch -1 SHA
```

当前分支所有超前master提交的patch
```
git format-patch -M master
```

两个commit之间的所有patch（不包含较早SHA1提交的内容）
```
git format-patch SHA1...SHA1
```

某个commit之后的所有patch
```
git format-patch -s SHA
```

## 应用PATCH
检查patch
```
git apply --stat xxx.patch
```

检查能否应用成功
```
git apply --check xxx.patch
```

打补丁
```
git am -s xxx.patch
```

如果有冲突，整个PATCH都不会被集成，接来来解决冲突问题
```
# 把没有冲突的文件先合并了，剩下有冲突的作了标记
git apply PATCH --reject
# 这里手动解决冲突
# 把解决冲突的和PATCH里新加的文件全部add进来，因为git am并不会改变index
git add FIXED_FILES
git am --resolved
```

## 参考
http://blog.csdn.net/daydring/article/details/42676987
