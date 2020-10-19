---
title: Git后悔药
comments: true
tags: [Git,reset,恢复,后悔]
categories: [程序猿]
toc: true
date: 2019-07-25 18:33:09
---
使用git提交代码过程中有时会手抖提交错误代码，这时就需要用到git的后悔药reset操作。

<!--more-->

```sequence
title: git reset
participant 工作区
participant 暂存区
participant 本地仓库
participant 远程仓库

 工作区->暂存区:(1) git add
 暂存区->本地仓库:(2) git commit
 本地仓库->远程仓库:(3) git push
```

### 差异（diff）

工作区vs暂存区: `git diff`

暂存区vs本地仓库: `git diff —cached`

本地仓库vs远程仓库: `git diff <分支名> origin/<分支名>`



### 撤消（reset）

撤消工作区修改: `git reset —hard`

撤消(1)`git add`: `git reset && git checkout .`或`git reset —hard`(会还原所有修改)

撤消(2) `git commit`: `git reset --hard origin/master`(使用远端的master分支恢复到本地)

撤消(3) `git push`: `git reset --hard HEAD^ && git push -f`(先在本地回到上一个版本，然后强推到远端)