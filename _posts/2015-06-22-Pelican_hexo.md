---
layout: post
title: hexo and pelican
date: 2015-06-22 00:00:00
categories:
- code
tags: 
- blog
mathjax: true
description: 用pelican或hexo建一个漂亮的博客
---

用pelican或hexo建一个漂亮的博客
<!--more-->

## install

### python 2.7
[https://www.python.org/downloads/release/python-2710/](https://www.python.org/downloads/release/python-2710/)

### easy_install
[https://pypi.python.org/pypi/setuptools](https://pypi.python.org/pypi/setuptools)
[https://bootstrap.pypa.io/ez_setup.py](https://bootstrap.pypa.io/ez_setup.py)
```batch
E:\Python27\python.exe ez_setup.py
```

### pip
easy_install pip 提示找不到命令
设置环境变量 E:\Python27;E:\Python27\Scripts
其实新版的python都自带了pip，无需安装

### make
http://pan.baidu.com/s/1hqzJBBe
解压到python目录

### pelican
pip install pelican

### markdown
```batch
pip install markdown
```

### quick-start
[https://en.wikipedia.org/wiki/List_of_tz_database_time_zones](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)
timezone Asia/Shanghai

### run
```batch
pelican content
cd ~/projects/yoursite/output
python -m pelican.server
```

### themes
下载themes
``` batch
pelican theme -i "你喜欢的主题"
```

# hexo
## install
[download](https://nodejs.org/dist/v0.12.5/node-v0.12.5-x86.msi)
``` batch
npm install -g hexo
mkdir blog
cd blog
hexo init
npm install
hexo generate
hexo server
```

## theme
clone theme to folder `themes`
_config.yml theme: jacman

-----------------------

`本博客欢迎转发,但请保留原作者信息`
github:[codejuan](https://github.com/CodeJuan)
博客地址:http://blog.decbug.com/

