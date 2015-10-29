---
layout: post
title: 学习R语言 
date: 2015-03-04 23:30:09
categories:
- code
tags: 
- R
mathjax: true
description:
---

# 安装


1. [mirriors](http://cran.r-project.org/mirrors.html)

2. select China， [bjtu](http://mirror.bjtu.edu.cn/cran)

3. [resouce](http://mirror.bjtu.edu.cn/cran/sources.html) , current version is [R-3.1.2](http://mirror.bjtu.edu.cn/cran/src/base/R-3/R-3.1.2.tar.gz)

<!--more-->

4. extract,    `sudo tar xf r-3.1.2.tar.gz -C extract/`

5. view the build document,`vim INSTALL`

6. `./configure`,  *error: No F77 compiler found*。[学自这里http://laymantech.blogbus.com/logs/80761679.html](http://laymantech.blogbus.com/logs/80761679.html)

	 会产生错误：configure: error: No F77 compiler found
	
	 R语言需要fortran compiler，也就是说， 在上面尝试寻找了若干种Fortran 编译器未果之后，提示你没有安装任何一种可以使用的fortran 77 编译器。随便装个gfortran就行了。
	
	 $ sudo apt-get install gfortran
	
	 再次运行./configure
	
	 $ ./configure
	
	 会产生错误：configure: error: con--with-readline=yes (default) and headers/libs are not available
	
	 首先检查是否安装readline.
	
	 $ sudo apt-get install readline-common
	
	 $ ./configure --with-readline=no
	
	 会出现错误：configure: error: --with-x=yes (default) and X11 headers/libs are no t available
	
	 $ ./configure --with-x=no --with-readline=no
	
	 配置通过，但是会产生如下warning：
	
	 configure: WARNING: you cannot build DVI versions of the R manuals
	
	 configure: WARNING: you cannot build DVI versions of all the help pages
	
	 configure: WARNING: you cannot build info or HTML versions of the R manuals
	
	 configure: WARNING: you cannot build PDF versions of the R manuals
	
	 configure: WARNING: you cannot build PDF versions of all the help pages
	
	 这是缺少生成相应格式manuals的插件，如果有需要可以依次安装。


7. sudo make，安装完毕。

# 简单试用

### 
cd 到bin，sudo ./R，进入R控制台
按照http://developer.51cto.com/art/201305/393121.htm试用一下
```
install.packages('quantmod') # 安装quantmod包 ，
```
会提示选择哪个镜像，19号beijing离我最近，于是输入19

q()是退出


-----------------------

`本博客欢迎转发,但请保留原作者信息`
github:[codejuan](https://github.com/CodeJuan)
博客地址:http://blog.decbug.com/

