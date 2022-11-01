---
title: Git常用命令别名设置
comments: true
categories: [程序猿]
tags: [Git, 教程]
date: 2019-06-28 17:17:06
---
如果平时使用git使用git命令多于GUI工具，则设置一些常用命令的别名有且于效率提升，以下是我平时使用较多的一些命令的别名设置

<!--more-->
Git别名设置

```bash
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.ci commit
git config --global alias.br branch
git config --global alias.cp cherry-pick
git config --global alias.unstage 'reset HEAD'
# 可用git pull -r代替
git config --global alias.fr '!f() { git fetch && git rebase $@; }; f'; 
# git提交日志
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset - %Cgreen(%cd)%C(yellow)%d%Creset %s %C(blue)[%an/%cn]%Creset' --date=format:'%Y-%m-%d %H:%M:%S' --abbrev-commit"
```

删除别名

```bash
git config --global --unset alias.xxx
```

以下两个命令设置git alias和zsh alias都失败，暂没找到方法可以设置别名

```bash
# 查看仓库提交者排名前 5
git log --pretty='%aN' | sort | uniq -c | sort -k1 -n -r | head -n 5
```

```bash
# 统计每个人增删行数
git log --format='%aN' | sort -u | while read name; do echo -en "$name\t"; git log --author="$name" --pretty=tformat: --numstat | awk '{ add += $1; subs += $2; loc += $1 - $2 } END { printf "added lines: %s, removed lines: %s, total lines: %s\n", add, subs, loc }' -; done
```



git lg命令效果图
![git lg命令效果图](/media/git-lg效果图.png)