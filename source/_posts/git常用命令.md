title: Git常用命令
date: 2015-09-11 20:05:13
categories: [程序猿]
tags: [Git]
---
![Git工作流图示](/media/Git工作流图示.jpg)

之前写过一篇[Git命令使用指南](/2015/01/16/Git命令使用指南/)，但感觉那个写的太乱，不接地气，有时我自己找一个命令都难找，所以今天写一篇文章整理一些比较基础的，但又不常用的一些命令，后面会慢慢更新。
<!--more-->

## 查看修改

```shell
# 查看文件修改/变动情况
git status -s
```

## 提交

```shell
# 把所有已跟踪的文件添加到暂存区
git add -u
# 把所有已跟踪并有更新的文件提交到本地仓库
git commit -am "update message"
```

## 列出分支
```
# 列出本地分支
git branch
或
git branch -v

# 列出本地和远程所有分支
git branch -a
或
git branch -va
```

## 删除分支
```
# 删除本地分支
git branck -D/-d <branch>

# 删除远程分支，注意冒号前有空格
git push origin :<分支名>
# 等价于
git push origin -d <branch>
```

## 推送分支
```
# 推送当前分支到默认remote上，remote上没有对应分支则自动创建
git push

# 推送当前分支到指定remote，remote上没有对应分支则自动创建
git push <remote>

# 推送到指定分支到remote的指定分支上
git push <remote> <remote_branch>:<loclal_branch>
```

## 拉取远程分支到本地
```
git checkout -b <branch> <remote>/<branch>
```
或
```
git checkout --track <remote>/<branch>
```

## 撤消提交（未`push`情况下）
```
git reset --mixed <SHA1> # 此SHA1之后的commit全部撤消，并回退index，工作空间代码不变，--mixed可省略
git reset --soft <SHA1> # 此SHA1之后的commit全部撤消，工作空间代码和index不变
git reset --hard <SHA1> # 此SHA1之后的commit全部撤消，工作空间代码和index全部退回
```

## 查看`commit`记录
```
git log --oneline -n # 单行显示最后n个commit的记录
```

## 本地分支和远程分支做关联
```
git branch --set-upstream-to=<remote>/<remote_branch> <loclal_branch>
```

## 推送本地当前新分支到远程
```
git push -u origin <branch_name>
```

## 查看日志
```
# 当前分支日志
git log
# 所有本地分支日志
git log --all
# 指定本地分支日志
git log <branch_name>
# 指定远程分支日志
git log origin/<branch_name>
# 所有远程分支日志
git log --all origin
```

## 重命名分支
```
git branch (-m | -M) [<old-branch>] <new-branch>
```