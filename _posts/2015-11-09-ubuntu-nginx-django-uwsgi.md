---
layout: post
title: 基于nginx和uWSGI在阿里云(Ubuntu)上部署Django
date: 2015-11-09
categories: web
tags: nginx uwsgi ubuntu django web
---

# 0. 前言

在阿里云上面买了一个服务器，于是在上面基于nginx和uWSGI部署Django。这篇博客记录一下其中的历程；

# 1. 一般安装

一般安装可以直接参考如下两篇文章，写的非常详细，自己就不赘述了：

- [基于nginx和uWSGI在Ubuntu上部署Django](http://www.jianshu.com/p/e6ff4a28ab5a)
- [Setting up Django and your web server with uWSGI and nginx](http://uwsgi-docs.readthedocs.org/en/latest/tutorials/Django_and_nginx.html)

基本上自己安装的时候总会遇到各种问题，下面主要是介绍一下自己遇到的问题以及解决方案；仅供参考；

补充说明：

1. Django的安装

{% highlight sh%}
# 安装最新版本的Django
sudo pip install django

# 指定安装版本
sudo pip install -v django==1.7.1  
{% endhighlight sh%}

2. mysql安装

{% highlight sh%}
# 检查是否已经安装了MySQL，若输入一下命令，没有反应，表示没有安装
sudo netstat -tap | grep mysql

# 安装MySQL，安装的时候需要输入root的密码，需要牢记！！！
sudo apt-get install mysql-server mysql-client

# 测试是否安装成功
sudo netstat -tap | grep mysql

# 登陆MySQL，运行命令后直接输入root的密码进入
mysql -uroot -p

# MySQL的一些管理
sudo start mysql								#启动MySQL服务
sudo stop mysql 								#停止MySQL服务
sudo mysqladmin -u root password newpassword	#修改MySQL管理员密码

{% endhighlight sh%}

3. mysite.wsgi && mysite.sock

{% highlight sh%}
# mysite.wsgi需要自己编写，mysite.sock不用管
uwsgi --socket mysite.sock --module mysite.wsgi --chmod-socket=664
{% endhighlight sh%}

mysite.wsgi参考代码如下：

{% highlight py%}
# coding: utf-8
import os
import sys
reload(sys)
sys.setdefaultencoding('utf8')

os.environ.setdefault("DJANGO_SETTINGS_MODULE", "mysite.settings")

# This application object is used by the development server
# as well as any WSGI server configured to use this file.
from django.core.wsgi import get_wsgi_application
application = get_wsgi_application()

{% endhighlight py%}

# 2. 踩过的坑

好多踩过的坑都忘了。。。简单说一下解决的方法吧。

1. 重启nginx

{% highlight sh%}
sudo /etc/init.d/nginx restart
{% endhighlight sh%}

2. 查看nginx日志

{% highlight sh%}
sudo vim /var/log/nginx/error.log
{% endhighlight sh%}

3. 提示“no python application found check your startup logs for errors”

{% highlight sh%}
python manager.py migrate
sudo /etc/init.d/nginx restart
{% endhighlight sh%}

4. 提示“probably another instance of uWSGI is running on the same address (:8000).”

{% highlight sh%}
# kill掉8000端口的进程
sudo fuser -k 8000/tcp
{% endhighlight sh%}

5. django发布后，去掉端口号

把端口号设置为80即可；即修改mysite_nginx.conf的listen为80

其余任何问题，欢迎留言，共同探讨。。。

# 3 其他问题

1. mysql编码问题：

参考这篇文章[Change MySQL 5.5 default character-set to UTF8](http://outofcontrol.ca/blog/comments/change-mysql-5.5-default-character-set-to-utf8)

注：如果之前已经有数据库，建议删除数据库重建，或者参考这篇文章[MySQL “incorrect string value” error when save unicode string in Django](http://stackoverflow.com/questions/2108824/mysql-incorrect-string-value-error-when-save-unicode-string-in-django)

2. 使用django-admin-bootstrap美化后台管理界面

参考这篇文章[Admin](https://andrew-liu.gitbooks.io/django-blog/content/admin.html)。但是这篇文章没有说清楚，配置好settings之后，还需要执行一个操作“./manage.py collectstatic”，作用是requested to collect static files at the destination location as STATIC_ROOT；

3. 博客开发

推荐[一个系列](https://andrew-liu.gitbooks.io/django-blog/content/django_introduction.html)；

# 参考文献

- [基于nginx和uWSGI在Ubuntu上部署Django](http://www.jianshu.com/p/e6ff4a28ab5a)
- [Setting up Django and your web server with uWSGI and nginx](http://uwsgi-docs.readthedocs.org/en/latest/tutorials/Django_and_nginx.html)
- [开发环境和Django安装](https://andrew-liu.gitbooks.io/django-blog/content/kai_fa_huan_jing_he_django_an_zhuang.html)
- [Linux(Ubuntu)下MySQL的安装与配置](http://blog.csdn.net/lizuqingblog/article/details/18423751)
- [python Django 学习笔记（四）—— 使用MySQL数据库](http://www.cnblogs.com/wendoudou/p/mysql.html)
- [Admin](https://andrew-liu.gitbooks.io/django-blog/content/admin.html)
- [Django admin templates styled with Twitter Bootstrap](https://github.com/aobo711/bootstrap-django-admin)