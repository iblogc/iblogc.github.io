---
title: Git stash的简单运用
comments: true
date: 2015-10-20 22:16:18
categories: [程序猿]
tags: [Git]
---

当我们在分支上改代码改到一半时，突然项目发现了一个bug需要修复，这时需要切换到另一个分支进行修改，以前的做法可能是把工作区改到一半的代码先commit，然后切换分支修复bug，再切换回来继续写代码，但这样会生成很多不必要的提交，这时你就需要使用`git stash`命令。
<!--more-->
`git stash`命令可将工作区的改动存储git栈，运行`git stash`之后，可以再运行`git status -s`验证下发现目录和上交commit时是一致的，没有任何修改，这时你就可以切换到其它分支进行工作，当你完成工作后，再切换回来，使用`git stash pop`可以从Git栈中读取最近一次保存的内容，恢复到工作区。

```
git stash: 备份当前的工作区的内容，从最近的一次提交中读取相关内容，让工作区保证和上次提交的内容一致。同时，将当前的工作区内容保存到Git栈中。
git stash save "message": 备份当前的工作区的内容，并添加备注信息
git stash list: 显示git栈内的所有备份，可以利用这个列表来决定从那个地方恢复。
git stash pop stash@{0}: 从git栈中读取并恢复工作区，然后删除对应的记录，默认恢复最新的（stash@{0}为最新）
git stash apply stash@{0}: 同git stash pop，但不会删除对应的记录
git stash drop: 删除最新的一个备份
git stash clear: 清空git栈。此时使用gitg等图形化工具会发现，原来stash的哪些节点都消失了。
```

## 参考
http://www.tuicool.com/articles/rUBNBvI
及`git stash --help`


