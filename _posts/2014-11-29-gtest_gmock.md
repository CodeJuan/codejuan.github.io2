---
layout: post
title: 学习gtest&gmock
date: 2014-11-29 00:00:00
categories:
- code
tags: 
- UT
- GTEST
- GMOCK
mathjax: true
description: 
---

### UT
单元测试是代码的第一道防线，尽量将问题在前期暴露出来，发现越早，解决的成本就越低。

所以，作为码农，必须掌握单元测试的方法，并将UT集成到本地构建及CI服务器上，这样无论是新开发，还是维护重构等等，都能对我们的改动进行检测，及时反馈。

以前用过JUnit和CppUnit，已不太适合当前开发。很多同僚都推荐gtest，我自然不能错过。

<!--more-->

### 下载
1. [csdn](http://www.csdn.net/)下载，有好人一生平安，不用积分。
2. [googlecode](http://code.google.com)

### make
有帖子说1.5之后的版本无法用make构建，其实并不是这样。

1.7版本还是支持make的，cd到make文件夹，然后make即可。

以gtest为例，会在make目录生成一个库文件，以及一个测试可执行文件`sample1_unittest`。

执行`sample1_unittest`

```bat
Running main() from gtest_main.cc
[==========] Running 6 tests from 2 test cases.
[----------] Global test environment set-up.
[----------] 3 tests from FactorialTest
[ RUN      ] FactorialTest.Negative
[       OK ] FactorialTest.Negative (0 ms)
[ RUN      ] FactorialTest.Zero
[       OK ] FactorialTest.Zero (0 ms)
[ RUN      ] FactorialTest.Positive
[       OK ] FactorialTest.Positive (0 ms)
[----------] 3 tests from FactorialTest (0 ms total)
```

接下来就是怎么链接到我们的工程，makefile如下:

```makefile
PPFLAGS += -isystem $(GTEST_DIR)/include 
CXXFLAGS += -g -Wall -Wextra -pthread 
TESTS = test 

#将gtest加入到include
GTEST_HEADERS = ../include
#引入静态库
LIB_DIR = ../lib/gtest_main.a
test :   
	g++ $(CPPFLAGS) $(CXXFLAGS) -lpthread -I $(GTEST_HEADERS) $(LIB_DIR) test.cpp   -o test 
clean : 
	rm -f test
```


-----------------------

`本博客欢迎转发,但请保留原作者信息`
github:[codejuan](https://github.com/CodeJuan)
博客地址:http://blog.decbug.com/

