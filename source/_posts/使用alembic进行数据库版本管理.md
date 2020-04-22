---
title: 使用alembic进行数据库版本管理
comments: true
tags: [Python, 教程]
categories: [程序猿]
date: 2018-09-13 18:53:14
---

转自：https://www.cnblogs.com/blackmatrix/p/6236573.html，做了部分修改

## 前言

随着项目业务需求的不断变更，数据库的表结构修改难以避免，此时就需要对数据库的修改加以记录和控制，便于项目的版本管理和随意的升级和降级。

Alembic 就可以很好的解决这个问题。Alembic 是 SQLAlchemy 作者开发的 Python 数据库版本管理工具。
<!--more-->

## 安装

```bash
pip install alembic
```

通过 pip 命令安装，如果使用虚拟环境，记得激活虚拟环境后再执行 pip 命令

同时需要安装的还有 SQLAlchemy 和 PyMysql

```bash
pip install sqlalchemy
pip install pymysql
```

## 初始化

在使用 alembic 之前，需要进行初始化操作。

```bash
alembic init <YOUR_ALEMBIC_DIR>
```

YOUR_ALEMBIC_DIR，可以取一个符合项目名称规范的目录名，如

```bash
alembic init alembic
```

**此时需要注意，如果之前是在虚拟环境中安装的 alembic，需要激活虚拟环境后，在执行上述命令。**

**同时，建议 cd 到项目根目录再执行初始化操作，因为 YOUR_ALEMBIC_DIR 会在当前目录下创建。**

显示类似结果即初始化成功。

```bash
Creating directory D:\Project\py_sqlalchemy_demo\alembic ... done
Creating directory D:\Project\py_sqlalchemy_demo\alembic\versions ... done
Generating D:\Project\py_sqlalchemy_demo\alembic.ini ... done
Generating D:\Project\py_sqlalchemy_demo\alembic\env.py ... done
Generating D:\Project\py_sqlalchemy_demo\alembic\README ... done
Generating D:\Project\py_sqlalchemy_demo\alembic\script.py.mako ... done
Please edit configuration/connection/logging settings in 'D:\\Project\\py_sqlalchemy_demo\\alembic.ini' befor
e proceeding.
```

初始化成功后，会在执行初始化命令的目录下，生成一个 alembic.ini 的配置文件，及一个 alembic 目录，目录名就是之前设置的 YOUR_ALEMBIC_DIR。

## 修改配置文件

接下来对 alembic.ini 的信息进行修改。

主要修改的是配置文件中的数据库连接部分。

```python
sqlalchemy.url = driver://user:pass@localhost:port/dbname
```

将配置文件中，此部分替换成对应的数据库连接，这个数据库连接的写法是与 SQLAlchemy 创建 engine 时是一样的。

如我在 demo 中使用的是 SQLAlchemy 与 PyMysql，那数据库连接就是类似如下

```python
mysql+pymysql://demo_user:demo123456@127.0.0.1:3306/demo_db
```

## 修改 env.py

除修改配置文件外，还需要对 YOUR_ALEMBIC_DIR 目录下的 env.py 文件进行修改。

在 env.py 中，将 target_metadata 设置成项目的 model，使 alembic 能获取到项目中 model 定义的信息。

将原先的

```python
target_metadata = None
```

修改成项目中的 model
```python
import os
import sys

sys.path.append(dirname(dirname(abspath(__file__))))
from app import db
target_metadata = db.metadata
```

## 创建新版本

用 alembic revision -m + 注释 创建数据库版本

```bash
alembic revision --autogenerate -m "init db"
```

运行后，类似如下结果，即创建版本成功

```bash
INFO  [alembic.runtime.migration] Context impl MySQLImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.autogenerate.compare] Detected removed table 'user'
Generating D:\Project\py_sqlalchemy_demo\alembic\versions\7b55b3d83158_create_tables.py ... done
```

每次修改过 SQLAlchemy 的 model，执行此命令即可创建对应的版本。

执行成功后，会在项目根目录下的 alembic/versions / 下生成的对应版本的 py 文件。命令规则是版本号 + 注释。(这个命名规则是在配置文件中定义的)

在每次创建新版本后，需要执行将数据库升级到新版本的命令，才能继续更新版本。

## 变更数据库

在每次创建新版本后，需要执行将数据库升级到新版本的命令，才能继续更新版本

**将数据库升级到最新版本**

```bash
alembic upgrade head
```

运行结果类似

```bash
(venv_win) D:\Project\py_sqlalchemy_demo>alembic upgrade head
INFO  [alembic.runtime.migration] Context impl MySQLImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade 7b55b3d83158 -> b034414f04cd, create tables02
```

其中，命令中的 head 和 base 特指最新版本和最初版本。当需要对数据库进行升级时，使用 upgrade，降级使用 downgrade。

**将数据库降级到最初版本**

```bash
alembic downgrade base
```

**将数据库降级到执行版本**，使用 alembic downgrade + 版本号，不包含注释部分

```bash
alembic downgrade <version>
```

如

```bash
alembic downgrade 7b55b3d83158
```

运行结果

```bash
INFO  [alembic.runtime.migration] Context impl MySQLImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running downgrade b034414f04cd -> 7b55b3d83158, create tables02
```

升级也是同样的道理，alembic upgrade + 版本号

## 离线更新（生成 sql 脚本）

在某些不适合在线更新的情况，可以采用生成 sql 脚本的形式，进行离线更新：

```bash
alembic upgrade <version> --sql > migration.sql
```

如：

```bash
alembic upgrade ae1027a6acf --sql > migration.sql
```

从特定起始版本生成 sql 脚本：

```bash
alembic upgrade <vsersion>:<vsersion> --sql > migration.sql
```

如：

```bash
alembic upgrade 1975ea83b712:ae1027a6acf --sql > migration.sql
```

如果是数据库降级操作，把 upgrade 替换为 downgrade。

## 查询当前数据库版本号

在对数据库进行升级或降级后，会在当前操作的数据库中新增一个表；alembic_version。

表中的 version_num 字段记录了当前的数据库版本号。

## 清除所有版本

如果需要清除所有的版本，将 versions 删除掉，同时删除数据库的 alembic_version 表。

## 参考资料

[http://alembic.zzzcomputing.com/en/latest/tutorial.html](http://alembic.zzzcomputing.com/en/latest/tutorial.html)

[http://www.codeweblog.com/%E5%B8%B8%E8%A7%81%E7%9A%84sqlalchemy%E5%88%97%E7%B1%BB%E5%9E%8B-%E9%85%8D%E7%BD%AE%E9%80%89%E9%A1%B9%E5%92%8C%E5%85%B3%E7%B3%BB%E9%80%89%E9%A1%B9/](http://www.codeweblog.com/%25E5%25B8%25B8%25E8%25A7%2581%25E7%259A%2584sqlalchemy%25E5%2588%2597%25E7%25B1%25BB%25E5%259E%258B-%25E9%2585%258D%25E7%25BD%25AE%25E9%2580%2589%25E9%25A1%25B9%25E5%2592%258C%25E5%2585%25B3%25E7%25B3%25BB%25E9%2580%2589%25E9%25A1%25B9/)

[http://blog.csdn.net/wenxuansoft/article/details/50242957](http://blog.csdn.net/wenxuansoft/article/details/50242957)