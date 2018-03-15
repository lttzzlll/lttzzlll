---
layout: post
title: Django Using Mysql
# gh-repo: lttzzlll/leetcode-java
# gh-badge: [star, fork, follow]
tags: [Django, Mysql]
---

Django 默认而且自带的数据库是sqlite，适用于开发阶段的快速迭代，但是生产环境上还是要使用Mysql或者postgresql这样功能齐全的数据库。下面是用mysql替换的sqlite的配置。

1. 安装 pymysql 和 mysqlclient

```SH
pip install pymsql mysqlclient
```

2. 修改settings.py

```Python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}
```

替换为

```Python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'myblog',
        'USER': 'root',
        'PASSWORD': 'lttzzlll',
        'HOST': 'localhost',
        'PORT': 3306
    }
}
```

> Django支持mysql，所以ENGINE可以使用 'django.db.backends.mysql'
> HOST 为本地安装的mysql 所以为 localhost 或者 '127.0.0.1'
> PORT 默认为 3306
> USER 和 PASSWORD 自己设置，并且最好不要用 root 角色，因为具有最高权限。

3. 在本地mysql的客户端链接中新建一个数据库名为 blog;

```SQL
create database blog;
```

4. 数据库迁移到mysql

```SH
python manage.py migrate
```

如果是sqlite，执行这条命令会自动创建一个名为blog的数据库，但是用mysql则须先创建一个数据库。执行该命令后效果如下：

![数据迁移后的数据库表](../img/mysql-database-tables.png)


5. 比如这里有一个Tag实例

```Python
# models.py
class Tag(models.Model):
    name = models.CharField(max_length=100)

    def __str__(self):
        return self.name
```

在交互式命令行新建一些数据，然后在mysql客户端查询一下，结果如下:

![执行查询后的结果](../img/mysql-database-blog_tags.png)


明确步骤后，便不会出错。