---
title: 代码托管平台码云(Gitee)到Gitea迁移记
comments: true
tags: [Git, Gitee, GitLab, Gite, Gogs, 迁移, 代码, Python]
categories: [程序猿]
toc: true
visible: show
indexing: true
date: 2020-03-01 19:42:34
---
团队的代码托管管理平台之前一直用Gitee的企业版本，但除了代码pull/push操作外，基本不用平台上其它功能，除了要新建一个仓库要打开下网页版，其它时间基本不会访问网页版本，所以经过半天的调研，从GitLab/Gogs/Gitea中选择了Gitea，把迁移过程记录如下。

<!--more-->

### 安装Gitea

因为服务器上刚好装有docker，按照[官方文档](https://docs.gitea.io/zh-cn/install-with-docker/)选择了最简单的docker安装。

```shell
docker pull gitea/gitea:latest
sudo mkdir -p /data/gitea
docker run -d --name=gitea -p 10022:22 -p 10080:3000 -v /data/gitea:/data gitea/gitea:latest
// 重启gitea
docker restart gitea
```

安装完成后遇到了页面有三个静态文件（css/js）加载不成功，导致页面排版混乱，F12查看控制台报错net::ERR_CONTENT_LENGTH_MISMATCH，google之，找到这篇文章

[Nginx 做代理时浏览器报错 net::ERR_CONTENT_LENGTH_MISMATCH](https://github.com/xhlwill/blog/issues/17)，按照此方法解决。



### 配置Nginx

在服务器Nginx上配置反向代理

vi /etc/nginx/conf.d/gitea.conf

```ini
upstream gitea {
    server 127.0.0.1:10080;
    keepalive 2000;
}
server {
    listen       80;
    server_name  git.i.example.com;
    client_max_body_size 1024M;

    location / {
        proxy_pass http://gitea/;
        proxy_set_header Host $host:$server_port;
    }
}
```

重新加载配置

```shell
sudo nginx -s reload
```



### 域名解析

git.i.example.com解析到当前服务器ip，并把服务器防火墙入方向的10022 tcp端口打开，以便使用ssh方式clone仓库时使用。



### Gitea初始化

打开http://.i.example.com，进入初始化界面（如果没进随便点注册或登录就会进），除了数据库根据需要配置，几个域名和网址要修改下，邮箱和其它选项按需配置。以后如果想修改配置，可以直接修改/data/gitea/gitea/conf/app.ini文件[配置说明](https://docs.gitea.io/zh-cn/config-cheat-sheet/)，修改完成后重启下gitea即可生效。



### 仓库迁移

因为我迁移的是团队项目，所以先通过Gitea提供的API把所有仓库以镜像方式（镜像方式同步过来仓库对成员为只读，并且可以设置间隔时间，默认8小时，定时从原始地址Gitee同步最新代码）同步过来**[操作1]**，然后为每个项目配置好协作者/团队/权限等设置，在这期间，团队成员还是往Gitee上提交代码，待全部设置完成后取消告知团队成员不要往Gitee提交代码，并调用Giea api把所有仓库从Gitee上同步一下最新代码**[操作2]**，然后每个仓库从镜像仓库转为普通仓库，并让团队的所有在自己仓库根目录执行修改本地仓库Git远程仓库地址替换操作**[操作3]**



**[操作1]**：登录Gitea后，界面右上角有一个加号，点开了后有一个迁移外部仓库的功能，只要填入外部仓库URL，授权验证信息等信息就可以一键把外部仓库的所有代码（包括所有branch和commit）迁移到Gitea，如果要迁移的仓库比较多，可以使用Gitea提供的Api来操作。对应此迁移操作的api是

```
POST /repos/migrate?access_token=<your gitea admin access token>

Request body
{
    description: MigrateRepoForm form for migrating repository
    auth_password: string
    auth_username: string
    clone_addr*: string
    description: string
    issues: boolean
    labels: boolean
    milestones: boolean
    mirror: boolean
    private: boolean
    pull_requests: boolean
    releases: boolean
    repo_name*: string
    uid*: integer($int64)
    wiki: boolean
}
```

***注：***

1. access_token 请在有管理员权限的账号的设置>应用中创建；

2. Request body 中的uid即管理后台>账户管理/组织管理中的ID列值；

   

找了Gitee没找到可以获取账户下所有仓库信息的API，所以只好手写了一个Gitee仓库地址的文件，类似

vi gitee-url.txt

```
https://gitee.com/example/project_a.git
https://gitee.com/example/project_b.git
```

使用shell脚本逐行读取url，并调用Gitea api迁移仓库。

```shell
#!/bin/bash

for line in $(<gitee-url.txt);
do
		# Windows注释下面这行
    line=$(echo $line | sed -e 's/\r//g');
    tmp=${line#https://gitee.com/xxx/};
    project_name=${tmp%.git};
    curl -X POST "http://git.i.example.com/api/v1/repos/migrate?access_token=<your gitea admin access token>" -H "accept: application/json" -H "Content-Type: application/json" -d "{ \"auth_password\": \"NDY2&F*K!hL75y*z\", \"auth_username\": \"korvin101@gmail.com\", \"clone_addr\": \"$line\", \"issues\": true, \"labels\": true, \"milestones\": true, \"mirror\": true, \"private\": true, \"pull_requests\": true, \"releases\": true, \"repo_name\": \"$project_name\", \"uid\": 2, \"wiki\": true}";
done
```



**[操作2]**：从Gitee上同步最新代码

```shell
for line in $(<gitee-url.txt);
do
    line=$(echo $line | sed -e 's/\r//g');
    tmp=${line#https://gitee.com/xxx/};
    project_name=${tmp%.git};
    curl -X POST "http://git.i.example.com/api/v1/repos/{owner}/$project_name/mirror-sync?access_token=<your gitea admin access token>" -H "accept: application/json"
done
```

***注：***owner为项目拥有者用户名/组织名



**[操作3]**：原本地仓库Git远程仓库地址替换

```shell
// http地址
// 原代码仓库http地址：https://gitee.com/example/project_a.git
// 新代码仓库http地址：http://git.i.example.com/JIANSU/project_a.git
// https://gitee.com/example > http://git.i.example.com/JIANSU
// 本地仓库使用此命令替换，可在包含所有项目的外层文件夹路径下执行批量替换
// Windows删除'.bak'
sed -i '.bak' 's/https:\/\/gitee\.com\/example/http:\/\/git\.i\.example.com\/JIANSU/g' */.git/config

// ssh地址
// 原代码仓库ssh地址：git@gitee.com:example/project_a.git
// 新代码仓库地址：ssh://git@git.i.example.com:10022/JIANSU/project_a.git
// git@gitee.com:example > ssh://git@git.i.example.com:10022/JIANSU
// 本地仓库使用此命令替换，可在包含所有项目的外层文件夹路径下执行批量替换
// Windows删除'.bak'
sed -i '.bak' 's/git@gitee\.com:example/ssh:\/\/git@git\.i\.example\.com:10022\/JIANSU/g' */.git/config
```

1. 如果之前是用http地址进行克隆的仓库的话，现在就是在进行pull和push操作时，把账户密码换成Gitea的就可以了；

2. 如果以前是用ssh克隆的仓库的话，现在在Gitea的设置>SSH / GPG 密钥里添加一下公钥就可以进行git pull/git push等操作了；


### 仓库备份

Gitea有自己的备份与恢复功能[备份与恢复](https://docs.gitea.io/zh-cn/backup-and-restore/#%E5%A4%87%E4%BB%BD%E4%B8%8E%E6%81%A2%E5%A4%8D)，这个备份比较全面，数据/代码/日志都可以备份，正是因为这样，如果仓库比较多这个备份的文件肯定会有点大，而且每次都是全量备份，所以频率肯定不能太高，而我只是想对仓库代码做一个高频率备份，所以写了一个Python3脚本调用Gitea api和 Git命令来进行所有仓库的所有分支代码备份，因为这个备份基于Git机制，所以虽然频率高，但备份始终只有一份。脚本如下：

backup.py

> 如果使用python2运行，分支名里有中文的话，请自行处理字符编码问题。

** python
```python
#!/usr/bin/python3
import os
import platform
import requests


current_dir = os.path.abspath(os.path.dirname(__file__))
access_token = "<your access token>"
repos_url = 'http://git.i.example.com/api/v1/repos/search?access_token={}&page={}&limit={}'
branches_url = 'http://git.i.example.com/api/v1/repos/{}/branches?access_token={}'
repo_key_url = 'http://git.i.example.com/api/v1/repos/{}/{}/keys?access_token={}'


def repos():
    page = 1
    limit = 50
    has_next = True
    while has_next:
        r = requests.get(repos_url.format(access_token, page, limit))
        for repo in r.json()['data']:
            yield repo
        page += 1
        has_next = len(r.json()['data']) == limit


"""拉取项目所有分支代码到本地"""


def sync_repo():
    repo_index = 0
    for repo in repos():
        repo_index += 1
        # 克隆仓库
        os.chdir(current_dir)
        print('克隆第 {} 个仓库 {} '.format(repo_index, repo['name']))
        os.system("git clone {}".format(repo['ssh_url']))
        os.chdir(os.path.join(current_dir, repo['name']))
        # 更新仓库
        print('同步 {} 仓库所有分支'.format(repo['name']))
        os.system('git fetch --all')
        # if platform.system() == 'Windows':
        # Windows
        branches = requests.get(branches_url.format(
            repo['full_name'], access_token)).json()
        for branch in branches:
            branch_name = branch['name']
            os.system('git branch --track {} origin/{}'.format(branch_name, branch_name))
            # 用reset而不用pull是因为如果分支被强推了pull下来会有合并冲突，用rest就不会有冲突问题
            os.system('git checkout {} && git reset --hard origin/{}'.format(branch_name, branch_name))
        # else:
        #     # Linux/macOS
        #     # git branch -r | grep -v '\->' | while read remote; do git branch --track ${remote#origin/} $remote; done && git fetch --all && git pull --all
        #     # os.system("git branch -r | grep -v '\->' | while read remote; do git branch --track ${remote#origin/} $remote; done && git fetch --all && git pull --all")
        #     # # 用reset而不用pull是因为如果分支被强推了pull下来会有合并冲突，用rest就不会有冲突问题
        #     os.system("git branch -r | grep -v '\->' | while read remote; do git branch --track ${remote#origin/} $remote; git checkout ${remote#origin/}; git reset --hard $remote; done")


"""设置项目部署公钥"""


def set_pub_key():
    repo_index = 0
    body = {
        "key": "ssh-rsa aabbcc",
        "read_only": True,
        "title": "SandBox"
    }
    for repo in repos():
        repo_index += 1
        print('==={}. {}==='.format(repo_index, repo['name']))
        r = requests.post(repo_key_url.format(
            repo['owner']['username'], repo['name'], access_token), data=body)
        print(r.json())


if __name__ == '__main__':
    sync_repo()
    # set_pub_key()
```

可以把脚本放在本地，使用cron(Linux/macOS)/计划任务(Windows)定时运行`python backup.py`

*[Windows计划任务运行cmd命令时，可使用非当前登录用户运行，这样就不会弹出小黑窗。](https://blog.csdn.net/flydragon0815/article/details/46006473)*