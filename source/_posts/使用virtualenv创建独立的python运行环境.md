---
title: 使用Virtualenv创建独立的Python运行环境
date: 2015-01-01 20:39:14
categories: [程序猿]
tags: [Virtualenv,virtualenvwrapper,独立,虚拟环境,Python,requirements]
---

## 准备工作

- python环境
- pip

## 安装

```
pip install virtualenv
```
或
```
pip install https://github.com/pypa/virtualenv/tarball/develop
```

<!--more-->

## 创建虚拟环境

```
virtualenv myVE
```

指定python解释器

```
 -p PYTHON_EXE, --python=PYTHON_EXE
```

*创建虚拟环境时默认会自动安装setuptools和pip*

不安装setuptool

```
--no--setuptools
```

不安装pip

```
--no--pip
```

*更多Options请参考[官方文档](https://virtualenv.pypa.io/en/latest/reference.html)* 

## 启动虚拟环境

 Mac OS

```
cd myVE
source ./bin/activate
```

Windows

```
cd myVE
scripts\activate
```

启动成功后可以在开头显示"(myVE)"，说明已经进入刚刚创建的虚拟环境了

## 退出

```
deactivate
```

## virtualenvwrapper 

### 安装

> Virtaulenvwrapper是virtualenv的扩展包，用于更方便管理虚拟环境，它可以做：
> 1. 将所有虚拟环境整合在一个目录下
> 2. 管理（新增，删除，复制）虚拟环境
> 3. 切换虚拟环境

```
pip install virtualenvwrapper
```
Windows下还需额外安装virtualenvwrapper-win
```
pip install virtualenvwrapper-win
```
ubuntu需要将下面这句加入到`~/.bashrc`里面
```
if [ -f /usr/local/bin/virtualenvwrapper.sh ]; then
    source /usr/local/bin/virtualenvwrapper.sh
fi
```
加入后需要重启才能生效，如果想要立即生效，输入命令
```
source ~/.bashrc
```


### 常用命令
*部分命令在windows下无效*

- `workon myEnv`: 切换虚拟环境
- `mkvirtualenv`: 新建工作环境
- `rmvirtualenv`: 删除工作环境
- `cdproject`: 切换到工程目录
- `workon`/`lsvirtualenv`: 列出所有虚拟环境
- `deactivate`: 退出虚拟环境
- `cpvirtualenv [source] [dest]` 复制一份虚拟环境。
- `cdvirtualenv [subdir]` 把当前工作目录设置为所在的环境目录。
- `cdsitepackages [subdir]` 把当前工作目录设置为所在环境的sitepackages路径。
- `add2virtualenv [dir] [dir]` 把指定的目录加入当前使用的环境的path中，这常使用于在多个project里面同时使用一个较大的库的情况。
- `toggleglobalsitepackages -q` 控制当前的环境是否使用全局的sitepackages目录。

---
## 参考资料

https://virtualenv.pypa.io/en/latest/

http://virtualenvwrapper.readthedocs.org/en/latest/

https://github.com/davidmarble/virtualenvwrapper-win