---
layout: post
title: openstack性能测试器(3):移植rabbitmq-c
date: 2015-10-26 00:00:00
categories:
- code
tags: 
- openstack
- AMQP
- rabbitmq
mathjax: true
description: 
---

# rabbitmq
rabbitmq是AMQP的一个具体实现。与之类似的还有apache的qpid。rabbitmq的官网是[http://www.rabbitmq.com/](http://www.rabbitmq.com/)

我要做的是一个收发消息的模拟器，那么就用client就可以了。rabbitmq提供了各种语言的client版本，其中JAVA、C#、ErLang是官方维护的亲儿子版本。至于C语言的版本，则被归类到了other languages，下载链接是[http://www.rabbitmq.com/devtools.html](http://www.rabbitmq.com/devtools.html)，C版本的链接是[https://github.com/alanxz/rabbitmq-c](https://github.com/alanxz/rabbitmq-c)

<!--more-->



# 移植

## 编译rabbitmq-c
生产环境用的是suse，那边哥们给我的测试环境无法mount到我的本机，所以，只能在我docker ubuntu14.04来先验证一下。

### CMake
下载[https://cmake.org/files/v3.4/cmake-3.4.0-rc2.tar.gz](https://cmake.org/files/v3.4/cmake-3.4.0-rc2.tar.gz)
```sh
wget https://cmake.org/files/v3.4/cmake-3.4.0-rc2.tar.gz
tar -xzvf cmake-3.4.0-rc2.tar.gz 
cd cmake-3.4.0-rc2
./bootstrap && make && sudo make install
```

#### ./bootstrap
```sh
-- Check if the system is big endian
-- Searching 16 bit integer
-- Using unsigned short
-- Check if the system is big endian - little endian
Curses libraries were not found. Curses GUI for CMake will not be built.
-- Looking for elf.h
-- Looking for elf.h - found
-- Looking for a Fortran compiler
-- Looking for a Fortran compiler - NOTFOUND
qmake: could not exec '/usr/lib/x86_64-linux-gnu/qt4/bin/qmake': No such file or directory
qmake: could not exec '/usr/lib/x86_64-linux-gnu/qt4/bin/qmake': No such file or directory
-- Performing Test run_pic_test
-- Performing Test run_pic_test - Success
-- Performing Test run_inlines_hidden_test
-- Performing Test run_inlines_hidden_test - Success
-- Configuring done
-- Generating done
-- Build files have been written to: /home/xh/save/code/cmake-3.4.0-rc2
---------------------------------------------
CMake has bootstrapped.  Now run make.

```
#### make
```sh
[ 61%] Built target cmjsoncpp
Scanning dependencies of target CMakeLib
[ 61%] Building CXX object Source/CMakeFiles/CMakeLib.dir/cmArchiveWrite.cxx.o
[ 61%] Building CXX object Source/CMakeFiles/CMakeLib.dir/cmBootstrapCommands1.cxx.o
[ 61%] Building CXX object Source/CMakeFiles/CMakeLib.dir/cmBootstrapCommands2.cxx.o
[ 61%] Building CXX object Source/CMakeFiles/CMakeLib.dir/cmCacheManager.cxx.o
[ 61%] Building CXX object Source/CMakeFiles/CMakeLib.dir/cmCommands.cxx.o
[ 61%] Building CXX object Source/CMakeFiles/CMakeLib.dir/cmCLocaleEnvironmentScope.cxx.o
[ 62%] Building CXX object Source/CMakeFiles/CMakeLib.dir/cmCommandArgumentLexer.cxx.o
[ 62%] Building CXX object Source/CMakeFiles/CMakeLib.dir/cmCommandArgumentParser.cxx.o
[ 62%] Building CXX object Source/CMakeFiles/CMakeLib.dir/cmCommandArgumentParserHelper.cxx.o

```

#### make install
```sh
-- Installing: /usr/local/share/cmake-3.4/Templates/UtilityHeader.dsptemplate
-- Installing: /usr/local/share/cmake-3.4/Templates/CTestScript.cmake.in
-- Installing: /usr/local/share/cmake-3.4/Templates/CPack.GenericDescription.txt
-- Installing: /usr/local/share/cmake-3.4/Templates/CPack.GenericLicense.txt
```

#### version
```sh
cmake --version
# cmake version 3.4.0-rc2
```

### openssl
```sh
wget http://www.openssl.org/source/openssl-1.0.2d.tar.gz
tar -xzvf openssl-1.0.2d
cd openssl-1.0.2d
./config
make
make test
make install
```
version一下看看
```sh
openssl version
# OpenSSL 1.0.1f 6 Jan 2014
```

### make
```sh
git clone git@github.com:alanxz/rabbitmq-c.git
cd rabbitmq-c
mkdir build && cd build
cmake ..
cmake --build . --config BUILD_TOOLS=OFF
```
`cmake ..`时会提示找不到xmlto等等，因为我不需要生成辅助工具（文档、命令行工具等等)，暂时不管它。
`BUILD_TOOLS=OFF`就表示不生成辅助工具

一路编译没有报错，至此，我们的rabbitmq就编译成功了，接下来就是要把`librabbitmq`移植到现有代码中。

## 移植过程

- 拷贝`librabbitmq`到原有代码中
- 在makefile里加上`librabbitmq`的路径
- make，提示说找不到`amqp_framing.h`，原因是include用的是`<>`，而amqp_framing.h就在同一个目录。
- sed -i s/<amqp_framing.h>/\"amqp_framing.h\"/g改成相对路径，再次make，不报这个错了。
- 找不到`threads.h`，原因是没有把`librabbitmq/unix`-I，所以需要改以下路径为`unix/threads.h`
- 找不到`AMQP_PLATFORM`,这个是在config.h里定义的。看了下报错的c文件，确实有include了config.h，但为什么还报这个错呢？
```c
#ifdef HAVE_CONFIG_H
#include "config.h"
#endif
```
- 原因就在于`HAVE_CONFIG_H`了，在makefile的gcc后加上`-DHAVE_CONFIG_H`，再次编译，就OK了。

在ubuntu上已经搞定，接下来就是迁移到suse上。
关键点就在于config.h，因为config.h包含了内核版本以及epoll等等，所以需要在suse上编译一下rabbitmq-c，然后把suse上的config.h替换ubuntu上的config.h，编译就OK了。


----------------------------

`本博客欢迎转发,但请保留原作者信息`
github:[codejuan](https://github.com/CodeJuan)
博客地址:http://blog.decbug.com/

