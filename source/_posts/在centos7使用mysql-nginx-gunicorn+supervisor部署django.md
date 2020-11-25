---
title: 在CentOS7上用MySQL+Nginx+Gunicorn+Supervisor部署Django
comments: true
date: 2016-12-08 23:19:59
categories: [程序猿]
tags: [centOS, Django, Nginx, Gunicorn, MySQL, Supervisor]
sticky: 
---
本文记录下在CentOS下部署Django项目的步骤。
<!--more-->

## MySQL

### 安装mysql和mysql-devel
```
yum install mysql
yum install mysql-devel
```

### 安装mysql-server
```
wget http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm
rpm -ivh mysql-community-release-el7-5.noarch.rpm
yum install mysql-community-server
```

### 重启mysql服务
```
service mysqld restart
```

### 设置root密码
初次安装mysql需要设置root密码
```
mysql -uroot
set password for 'root'@'localhost' =password('password');
```

### 配置mysql
在`/etc/my.cnf`文件中[mysql]和[mysql]中添加以下内容
```
[mysql]
default-character-set=utf8

[mysqld]
character-set-server=utf8
```
字符编码保持和`/usr/share/mysql/charsets/Index.xml`中的一致。

### 远程连接设置
把在所有数据库的所有表的所有权限赋值给位于所有IP地址的root用户。
```
mysql> grant all privileges on *.* to root@'%'identified by 'password';
```
如果是新用户而不是root，则要先新建用户
```
mysql>create user 'username'@'%' identified by 'password';
```
此时就可以进行远程连接了。

## Virtualenv
安装epel扩展源
```
yum install epel-release
```
安装pip
```
yum install python-pip
```
安装virtualenv和virtualenvwrapper
```
pip install virtualenv virtualenvwrapper
```
编辑`~/.bashrc`文件，结尾添加以下内容
```
export WORKON_HOME=~/.virtualenvs
source /usr/bin/virtualenvwrapper.sh
```
然后执行以下命令使配置生效
```
source ~/.bashrc
```
创建env
```
mkvirtualenv explame
```
使用pip安装项目需要的包

## WSGI
在项目目录下新建`nginx_wsgi.py`文件
```
touch nginx_wsgi.py
```
添加如下内容
```
import sys
import site
import os
 
# site-packages
site.addsitedir('/home/nginxuser/.virtualenvs/example/lib/python2.7/site-packages')
# Add the  project  directory
# sys.path.append('/home/nginxuser/nginxuser')
PROJECT_DIR = '/home/nginxuser/projects/example'
sys.path.insert(0, PROJECT_DIR)
os.environ['DJANGO_SETTINGS_MODULE'] = 'example.settings.prod'
# Activate your virtual env
activate_env = os.path.expanduser("/home/nginxuser/.virtualenvs/example/bin/activate_this.py")
execfile(activate_env, dict(__file__=activate_env))
 
# after activite env
from django.core.wsgi import get_wsgi_application
 
application = get_wsgi_application()
```

## Nginx

### 安装
```
yum install nginx
```
### 检查配置是否有错
```
nginx -t -c /etc/nginx/nginx.conf
```

### 启动nginx
```
service nginx start
```

### 设置开机自启
```
systemctl enable nginx
```

### 创建用户
```
useradd nginxuser
passwd nginxuser
```
### 修改nginx主配置
```
vim /etc/nginx/nginx.conf
```
非注释首行
```
user nginx
```
改为
```
user nginxuser
```
不然可能会出现网站静态文件访问报403问题。
### 新建网站运行配置
```
vim /etc/nginx/conf.d/example.conf
```
```
server {                                                               
    listen      80;                                                    
    server_name example.com;                            
    charset     utf-8;                                                 
    client_max_body_size 75M;                                          
    access_log /home/nginxuser/projects/example/nginxlogs/access.log;
    error_log /home/nginxuser/projects/example/nginxlogs/error.log;          
 
    location /static {                                                 
        alias /home/nginxuser/projects/explame/static;                
    }                                                                  
 
    location / {                                                       
        proxy_pass http://127.0.0.1:8000;                              
        proxy_set_header Host $host;                                   
        proxy_set_header X-Real-IP $remote_addr;                       
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;   
    }                                                                  
}                                                                       jk
```

## Gunicorn
### 安装
```
pip install gunicorn
```
项目根目录下添加gunicorn运行配置文件gunicorn.conf.py
```
import multiprocessing
bind = "127.0.0.1:8000"
workers = 2
errorlog = "/home/nginxuser/example/gunicorn.error.log"
#loglevel = "debug"
proc_name = "gunicorn_example"
```
### 启动
```
sudo gunicorn example.nginx_wsgi:application -c /home/nginxuser/projects/example/gunicorn.conf.py
```
后台运行
```
sudo nohup gunicorn example.nginx_wsgi:application -c /home/nginxuser/projects/example/gunicorn.conf.py&
```
如果运行报错先使用以下命令检查下nginx配置是否有错
```
nginx -t -c /etc/nginx/nginx.conf
```
## Supervisor
### 安装
```
pip install supervisor
```
### 创建管理进程配置文件
```
vim /etc/supervisord.d/example.ini
```
（需要注意：用 supervisord 管理时，gunicorn 的 daemon 选项需要设置为 False）
```
[program:example]
directory = /home/nginxuser/projects/example ; 程序的启动目录
command = gunicorn example.nginx_wsgi:application -c /home/nginxuser/projects/example/gunicorn.conf.py  ; 启动命令，可以看出与手动在命令行启动的命令是一样的
autostart = true     ; 在 supervisord 启动的时候也自动启动
startsecs = 5        ; 启动 5 秒后没有异常退出，就当作已经正常启动了
autorestart = true   ; 程序异常退出后自动重启
startretries = 3     ; 启动失败自动重试次数，默认是 3
user = nginx         ; 用哪个用户启动
redirect_stderr = true  ; 把 stderr 重定向到 stdout，默认 false
stdout_logfile_maxbytes = 20MB  ; stdout 日志文件大小，默认 50MB
stdout_logfile_backups = 20     ; stdout 日志文件备份数
; stdout 日志文件，需要注意当指定目录不存在时无法正常启动，所以需要手动创建目录（supervisord 会自动创建日志文件）
stdout_logfile = /data/logs/usercenter_stdout.log

; 可以通过 environment 来添加需要的环境变量，一种常见的用法是修改 PYTHONPATH
; environment=PYTHONPATH=$PYTHONPATH:/path/to/somewhere
```
**冒号后面要有空格**

### 启动
使用`-c`指定配置文件。
```
supervisord -c /etc/supervisord.conf
```
如果启动时遇到以下报错信息
```
Error: Another program is already listening on a port that one of our HTTP servers is configured to use. Shut this program down first before starting supervisord.
For help, use /use/bin/supervisord -h
```
可以使用以下命令解决
```
sudo unlink /var/run/supervisor/supervisor.sock
```

### 命令行客户端工具supervisorctl
启动时需要使用和`supervisorctl`使用一样的配置文件。
```
supervisorctl -c /etc/supervisord.conf
```
启动后进入`supervisorctl`的shell，在此shell里可以执行以下命令
```
status # 查看程序状态
start example # 启动example程序
stop example # 关闭example程序
restart example # 重启example程序
reread # 读取有更新（增加）的配置文件，不会启动新添加的程序
update # 重启配置文件修改过的程序
```
也可以不进shell执行以上命令
```
supervisorctl status # 查看程序状态
supervisorctl start example # 启动example程序
supervisorctl stop example # 关闭example程序
supervisorctl restart example # 重启example程序
supervisorctl reread # 读取有更新（增加）的配置文件，不会启动新添加的程序
supervisorctl update # 重启配置文件修改过的程序
```

### 开启web管理界面
如果要开启web管理界面，打开`/etc/supervisord.conf`把下面几行取消注释即可
```
:[inet_http_server]         ; inet (TCP) server disabled by default
:port=127.0.0.1:9001        ; (ip_address:port specifier, *:port for all iface)
:username=user              ; (default is no username (open server))
:password=123               ; (default is no password (open server))
```


