---
layout: post
title: 在github上面搭建个人博客
date: 2015-09-30
categories: other
tags: other
---


## 0. 前言

一直想把之前搭建博客的过程详细记录下来，可惜一直未能成行；如今再次有这个念头，不如一鼓作气写将出来；


## 1. 步骤

- 配置和使用Github；
- 建立类似username.github.io的用户&组织页（站）；
- 安装Jekyll，配置模板；
- 上传到github上，访问username.github.io就可以了；

下面以wmk26这个github账号为例，详细说明如何搭建；

注：在Ubuntu 14.04 LTS上面操作，windows和Mac下应该类似；


## 2 配置和使用github

- 安装git；
- 配置git，并连接github；

### 2.1 安装

- 推荐使用Linux下面的git；
- 安装教程参考[这篇文章](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/00137396287703354d8c6c01c904c7d9ff056ae23da865a000)(Linux，Mac，Windows都有)；
	
### 2.2 初始化（注意把用户名和邮箱改成自己的）
	
{% highlight sh %}
git config --global user.name "wmk26"
git config --global user.email "1549359482@qq.com"
{% endhighlight sh %}
	
### 2.3 初始化Git仓库
	
{% highlight sh %}
# Mytest目录是工作目录，仅作示意
$ mkdir ~/Mytest
$ cd ~/Mytest        
$ git init  

# 然后会显示： 初始化空的 Git 版本库于 /home/wmk/Mytest/.git/ 
{% endhighlight sh %}

### 2.4 连接github并上传repository

1). 新建github项目

在github上新建一个repositories，命名规则为：xxx.github.io，其中xxx是自定义的名称；每个用户名下面只能建立一个；

2). 在本地创建ssh key：

{% highlight sh %}
ssh-keygen -t rsa -C "1549359482@qq.com"
# 输入命令之后，一路回车即可，即使用默认的配置；
{% endhighlight sh %}

3). 成功创建之后，会在~/下面生成.ssh文件夹，进去，打开id_rsa.pub，复制其中的key；

4). 回到github主页，点击右上角的账户的setting选项，进入Account Settings，左边选择SSH Keys，Add SSH Key，title随便填，粘贴key；

5). 验证

{% highlight sh %}
# 在终端输入
ssh - T git@github.com
# 如果是第一次的会提示是否continue，输入yes就会看到：You’ve successfully authenticated, but GitHub does not provide sh access 。这就表示已成功连上github。
{% endhighlight sh %}

6). 设置远程连接github

{% highlight sh %}
# 设置远程连接wmk26.github.io这个repository
git remote add origin https://github.com/wmk26/wmk26.github.io.git
# 把本地仓库的文件上传到github中；
git push origin master
{% endhighlight sh %}

注：自己曾经遇到的一些问题[
ubuntu下git&github使用](http://www.wangmingkuo.com/linux/ubuntu%E4%B8%8Bgitgithub%E4%BD%BF%E7%94%A8/)


## 3. 安装Jekyll，配置模板

- 直接参考[官方文档](http://jekyllcn.com/docs/installation/)
- [这篇文档](http://trefoil.github.io/2013/10/05/jekyll.html)也不错；
- 获取并修改别人的博客，参考[这篇文章](http://trefoil.github.io/2013/10/05/jekyll.html)
- [自己博客的模板](http://pan.baidu.com/s/1kTnLxUf)（百度网盘）或者直接fork别人的模板；
- 配置，修改_config.yml文件；其中包括markdown的配置，网站的一些基本配置，以及多说评论的配置；
- 安装完成之后，可以运行jekyll serve --watch在本地先查看一下网站的雏形（运行没问题之后，访问：http://127.0.0.1:4000 ）可能会遇到下面javascript的问题，解决方法参考下面；

注1：

- 个人推荐使用多说的评论框架，去[多说](http://duoshuo.com/)注册一个账号，之后修改_config.yml文件中的duoshuo选项，即可；
- 其他评论使用可以参考[这篇文章](http://beiyuu.com/github-pages/)

可能会遇到的问题

1) ruby版本问题

{% highlight sh %}
$ sudo gem install jekyll    

ERROR:  Error installing jekyll:
ERROR: Failed to build gem native extension.
{% endhighlight sh %}

解决，参考[这个回答](http://stackoverflow.com/questions/10725767/error-installing-jekyll-native-extension-build)

{% highlight sh %}
sudo apt-get install ruby1.9.1-dev
sudo gem install jekyll
{% endhighlight sh %}

原因：

- 网上所是ruby的版本不够新，但是明显机器上面安装了1.9.3版本的。所以猜想估计是版本不匹配，需要1.9.1版本的。


2) javascript问题

{% highlight sh %}
Could not find a Javascript runtime.
{% endhighlight sh %}

问题分析：没有javascript解释器

解决方法：

{% highlight sh %}
sudo apt-get update
sudo apt-get install nodejs
{% endhighlight sh %}

## 4. 上传github

进入Jekyll模板文件夹下，上传模板，之后就可以访问自己设置的域名了，如我的是wmk26.github.io，后来改成自己的域名，改自己的域名可以参考[这篇文章](http://beiyuu.com/github-pages/)

一个例子：
{% highlight sh %}
# clone别人的模板，或者直接下载别人的模板
cd ~/Mytest
git clone https://github.com/wmk26/wmk26.github.com.git
rm -rf .git
# git初始化，提交
git init
git add -A
git commit -m "my first commit"
# USERNAME换成自己的github repository
git remote add origin https://github.com/USERNAME/USERNAME.github.io.git
# 上传github
git push -u origin master
{% endhighlight sh %}

如果没有意外，应该就可以访问了；

注：写文章就可以直接在_post文件夹下新建文件写即可；

- 命名规则，日期-文章名，如：2015-09-30-build-a-blog-on-github.md
- 文章规范：

{% highlight sh %}
---
layout: post
title: 在github上面搭建个人博客
date: 2015-09-30
categories: other
tags: other
---
{% endhighlight sh %}

- 其他就使用Markdown语法即可；
- 推荐使用[Visual Studio Code编辑器](https://code.visualstudio.com/)，支持Markdown语法，可以预览；


## 5. 其他配置

- [有推荐的简洁明快的jekyll模板吗？](http://www.zhihu.com/question/20223939)（一些人在使用的模板）
- [全站链接新标签页/新窗口打开](http://www.tangblog.com/80.html)
- [Jekyll 中的语法高亮](http://havee.me/internet/2013-08/support-pygments-in-jekyll.html)


## 6. 参考资料&&不错的资源

- [Blogging Like a Hacker](http://tom.preston-werner.com/2008/11/17/blogging-like-a-hacker.html)(安利一下，jekyll的作者写的)
- [使用Github Pages建独立博客](http://beiyuu.com/github-pages/)(主要参考文章)
- [每个人都应该有一个Jekyll博客](http://www.cellier.me/2015/01/04/jekyll%E6%90%AD%E5%BB%BA%E5%8D%9A%E5%AE%A2%E6%95%99%E7%A8%8B/)(文章中列出了很多其他资源)
- [用Github + Jekyll搭建博客--以Ubuntu为例](http://trefoil.github.io/2013/10/05/jekyll.html)(从这篇文章中学会如何“偷”别人的模板，2333)
- [搭建一个免费的，无限流量的Blog----github Pages和Jekyll入门](http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html)
- [Jekyll博客搭建](http://segmentfault.com/a/1190000002539546)
- [献给写作者的Markdown新手指南](http://www.jianshu.com/p/q81RER)（非常好的入门指南）
- [Git教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)(git入门很好的资源)
- [涂亮](http://www.tuliang.org/)（博客写的很好，有时间学习一下）
- [Keep on Fighting!|谢益辉](http://yihui.name/cn/)（就是被这个博客吸引过来的，很喜欢其博客的主题，可惜“偷”了一会儿，感觉太麻烦了，遂另寻出路，还好，找到了）
- [飘过的小牛](http://github.thinkingbar.com/)（非常感谢这位作者，模板就是从这里“偷”的，文章也写的很不错，和自己还是有很多共同兴趣的，找时间一定好好学习）
- [我为什么写博客？](http://beiyuu.com/why-blog/)（文章写的不错，里面的一些链接也不错）