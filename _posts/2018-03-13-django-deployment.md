---
layout: post
title:  Django 部署
# gh-repo: lttzzlll/leetcode-java
# gh-badge: [star, fork, follow]
tags: [Django, Nginx, uWSGI, Ubuntu]
---

一次简单的Django本地部署在Ubuntu上的经历.

Ubuntu version: 16.04 LTS
Nginx version: 1.12.2
uWSGI version: 2.0.15
Django version: 1.11

前提 Python, Django, pip, Nginx, uWSGI已经安装好,重点关注配置文件的编写.

Django 新建一个项目,名为mysite

```sh
django-admin startproject mysite
```

本机 mysite项目路径:

```sh
/home/ltt/Code/web/py/mysite
```

进入该项目的目录:

```sh
cd /home/ltt/Code/web/py/mysite
```

在 mysite/urls.py (绝对路径/home/ltt/Code/web/py/mysite/mysite/urls.py) 中新建一个基本的路由用于测试:

```Python
# urls.py
from django.conf.urls import url
from django.contrib import admin
from django.shortcuts import HttpResponse

def index(request):
    return HttpResponse('welcome to this page')

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^$/', index), # 新添加的用于测试
]
```

当前路径下新建一个uwsgi.ini文件,写入下面内容:

```ini
[uwsgi]
chdir=/home/ltt/Code/web/py/mysite
uid=nobody
gid=nobody
module=mysite.wsgi:application
socket=/home/ltt/Code/web/py/mysite/uwsgi.sock
master=true
workers=5
pidfile=/home/ltt/Code/web/py/mysite/uwsgi.pid
vacuum=true
thunder-lock=true
enable-threads=true
harakiri=30
post-buffering=4096
daemonize=/home/ltt/Code/web/py/mysite/uwsgi.log
```

第一行的[uwsgi]必须要有.

然后是nginx配置文件, 在路径/etc/nginx/sites-available/下新建mysite.conf文件,写入以下内容:

```conf
server {
    listen 8080;
    location / {
        include uwsgi_params;
        uwsgi_connect_timeout 30;
        uwsgi_pass unix:/home/ltt/Code/web/py/mysite/uwsgi.sock;
    }
}
```

建立一个软链接:

```sh
ln -s /etc/nginx/sites-available/mysite.conf /etc/nginx/sites-enabled/mysite.conf 
```

重启nginx

```SH
sudo service nginx restart
```

启动uWSGI

```SH
uwsgi --ini uwsgi.ini
```

访问 127.0.0.1:8080

welcome to this page.
