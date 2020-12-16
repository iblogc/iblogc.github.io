---
title: Git命令使用指南
date: 2015-01-16 22:33:30
categories: [程序猿]
tags: [Git, 教程]
---

![Git工作流图示](/media/Git工作流图示.jpg)

Git是软件开发人员在开发中常用的一种工具,是开发之利器。
> Git是一个开源的分布式版本控制系统，用以有效、高速的处理从很小到非常大的项目版本管理。
<!--more-->

## 工作流图示

![工作流图示](/media/git-reset_drbfhd.png)


## 命令

### 配置

- `git config --global user.name 'Your Name'` 设置git提交显示的名字

- `git config --global user.email your_email@example.com` 设置git提交显示的邮箱

- `git config --global alias.unstage "reset HEAD"` 替换命令 `git reset HEAD`命令改为 `git unstage`

- `ssh-keygen -t rsa -C your_email@example.com` 生成SSH Key

- `git config --global core.editor emacs` 设置文件编辑器

- `git config --global merge.tool vimdiff` 设置差异分析工具

- `git config --list` 查看配置信息



### 简洁版

初始化仓库
`git init`

添加远程仓库
`git remote add <自定义名字> <远程仓库url> `

给某个仓库名再添加另一个远程仓库url（可实现一次提交到两个远程仓库）
`git remote set-url --add <自定义名字> <远程仓库url>`    

更新项目
`git pull`

合并分支到当前分支
`git merge <分支名>`

创建标签
`git tag <标签名字> <提交id前10位字符>` *可通过`git log`获取*

获取log
`git log`

切换分支
`git checkout <分支名>`

创建分支并切换过去
`git checkout -b <分支名>`

删除分支
`git branch -D <分支名>`

推送
`git push origin <分支名/标签名>`

强制推送更新
`git push -f origin <分支名/标签名>`

推送所有分支
`git push origin --all`

推送所有标签
`git push origin --tags`

撤消本地改动（新文件和提交到缓存区的改动，不受影响）
`git checkout -- <目录><文件名>`

撤消本地所有提交与改动
***假如你想要丢弃你所有的本地改动与提交，可以到服务器上获取最新的版本并将你本地主分支指向到它***
`git fetch origin`
`git reset --hard origin/master`

其它命令
- `gitk ` 获取当前分支图形个界面
  - 参数`<分支名>`: 获取某分支图形界面
  - 参数`=--all`: 获取所有分支图形个界面
  - `cat <目录><文件名> ` 查看文件内容

---

### 详细版

初始

- `git init` 初始化仓库

- `ls` 显示目录下文件及文件夹（不包含隐藏文件即名字前带点的）

  - 参数`-a`显示目录下所有文件及文件夹

- `git clone <url>` 克隆项目

提交

- `git add <目录><文件名>` 添加文件到版本库，*可以多个文件一起添加，中间用空格隔开*

- `git add *` 或 `git add .` 添加所有文件到版本库

![status示例图](http://iblogc.qiniudn.com/iblogcd60500d5-addf-4022-ae4f-c1a57d1f5dd1112.png)

- `git status` 查看项目当前状态，详细信息

  - 参数`-s`: 显示简洁版

  > 绿色表示已经提交的缓存区，红色表示在工作区未提交到缓存区的
  > A新增  M修改  D删除 U冲突 R重命名？
  > push会把绿色部分提交，红色部分不提交
  > 已有记录文件做过改动和新文件，需要`git add`

- `git diff ` 查看整个项目里的文件改动情况（工作区和缓存区比较）

  - 参数`<目录><文件名>`: 查看单个文件改动情况（工作区和缓存区比较）
  -参数`<标签名>`: 查看自当前标签发布之后项目的改动情况
  - 参数`--cached`: 查看整个项目里的文件改动情况（缓存区和本地仓库比较）
  - 参数 `HEAD`: 查看整个项目里的文件改动情况（工作区和本地仓库比较）
  - 参数`--stat`: 显示摘要，而非完整diff

- `git commit`: 提交到缓存

  - 参数`-m`: 后面空格接提交信息
  - 参数`-a`: 为所有已有记录文件执行`git add`（新添加文件还是需要手动`git add`）

- `git reset HEAD` 取消缓存已缓存的内容
  - 参数`<目录><文件名>`: 单个文件取消缓存已缓存内容

- `git rm <目录><文件名>`:  将文件从缓存区和硬盘上移除

  - 参数`--cached`: 删除缓存中的文件，保留硬盘上的文件

- `git mv` 不推荐用

- `git log` 显示当前分支提交记录

  - 参数`--author=<authorname>`: 只寻找某个特定作者的提交
  - 参数`--oneline`: 显示简洁版
    - 参数`--oneline -<数字N>`: 显示简洁版，显示最近N次提交的记录
  - 参数`--graph`: 显示拓扑图（查看历史中什么时候出现了分支、合并）
  - 参数`--grep=<关键字>`: 根据提交注释关键字过滤提交记录
  > Git 会对所有的 --grep 和 --author 参数作逻辑或。 如果你用 --grep 和 --author 时，想看的是某人写作的并且有某个特殊的注释内容的提交记录， 你需要加上 --all-match 选项。 在这些例子中，我会用上 --format 选项，这样我们就可以看到每个提交的作者是谁了。详细参考：[Git参考手册:检查与比较](http://gitref.org/zh/inspect/)
  - 参数`<分支名>`:显示指定分支“可及”的提交记录
  - 参数`<分支名1> ^<分支名1>`: 查看在分支1不在分支2中的提交记录
  > 分支可以是本地的也可以是远端的
  - 参数`--decorate`: 显示带tag的记录
  - 参数`-p`: 显示每个提交引入的补丁
  - 参数`--stat`: 显示每个提交引入的差值统计
  - 其它参数 `--since` `--before` `--until` `--after`
  > git log --since --before 根据日期过滤提交记录
如果你要指定一个你感兴趣的日期范围以过滤你的提交，可以执行几个选项 —— 我用 --since 和 --before，但是你也可以用 --until 和 --after。 例如，如果我要看 Git 项目中三周前且在四月十八日之后的所有提交，我可以执行这个（我还用了 --no-merges 选项以隐藏合并提交）[Git参考手册:检查与比较](http://gitref.org/zh/inspect/)：

  
      $ git log --oneline --before={3.weeks.ago} --after={2010-04-18} --no-merges
      5469e2d Git 1.7.1-rc2
      d43427d Documentation/remote-helpers: Fix typos and improve language
      272a36b Fixup: Second argument may be any arbitrary string
      b6c8d2d Documentation/remote-helpers: Add invocation section
      5ce4f4e Documentation/urls: Rewrite to accomodate transport::address
      00b84e9 Documentation/remote-helpers: Rewrite description
      03aa87e Documentation: Describe other situations where -z affects git diff
      77bc694 rebase-interactive: silence warning when no commits rewritten
      636db2c t3301: add tests to use --format="%N"

分支

- `git branch`列出当前项目的可用分支，并显示当前工作目录当前分支

- 参数`<分支名>`: 创建分支

- `git checkout <分支名>` 切换到对应分支

  - 参数`-b` 创建分支并立即切换到新分支

- `git merge <分支名>` 合并指定分支到当前分支

标签

- `git tag` 显示当前项目的标签

  - 参数`<标签名>` 给某个历史记录打标签
  - 参数`-a`: 添加注解
  - 参数`<SHA>`: 提交id前n位字符，可通过`git log`获取，n位基于SHA唯一就行（建议5~7位）

远程

- `git remote` 列出远端别名
  -参数`-v`: 列出远端别名及链接
  > 一般一个别名会看到两个相同的链接（fetch和push）分别是获取和推送的链接
  -`add <仓库别名> <仓库链接>`: 为项目添加一个新的远端仓库
  - `rm <仓库别名>`: 为项目删除一个远端仓库
  > 只是本地删掉和远端仓库的链接，不会对远端仓库造成影响

- `git fetch` 从远端仓库下载最新的分支与数据

- `git pull` 从远端仓库下载最新数据，并尝试合并到当前分支
  - 参数`<仓库别名>`: 从哪个仓库拉取更新，默认为origin
> `git pull`实际是先`git fetch`后`git merge`

- `git push` 推送更新
  - 参数`<仓库别名> <分支名>`: 推送新分支与数据到某个远端仓库
  - 参数`<仓库别名> --all`: 推送所有分支
  - 参数`<仓库别名> --tagsl`: 推送所有标签



## 参考资料
> [Git 参考手册](http://gitref.org/zh)
> [git - 简易指南](http://www.bootcss.com/p/git-guide/)



