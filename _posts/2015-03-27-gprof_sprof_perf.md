---
layout: post
title: Linux C++性能调优笔记
date: 2015-03-27 23:30:09
categories:
- code
tags: 
- optimization
- linux
- C++
mathjax: true
description:
---

## 前言

上周领导在群里问谁会linux C开发，我曾在业余时间自己捣鼓过，于是回答略懂。这周就被派到别的项目组紧急支援开发。大体工作是开发一个so供前台调用，开发过程中对makefile、跨平台的理解越发深刻了。相比于自娱自乐，正规开发更能涨知识。

此后数据量较大，很快就出现性能问题，作为折腾砖家，我当仁不让地接下性能调优的活。

每当遇到难题，都是我比较开心的时候，因为又是一次get新技能的好机会。

<!--more-->

## profiler

当系统的性能不能满足要求的时候，便要对其进行调优。方法有千千万万种，但无论如何，我们都要想办法精确识别到瓶颈，然后有的放矢进行优化。如何精确识别呢？这个时候profiler就闪亮登场了。

*尝试了很多profiler之后，最终还是决定使用google perf tools*

### linux常用profiler

gprofile & perf & sprof

#### gprofile

老牌劲旅，长久不衰，很多人用，功能也比较强大，使用简单。

- 在编译时加入参数 -pg就可以打开GProfile的开关
- 运行你的可执行文件，结束后会生成一个gmon.out
- 分析结果：gprof -b '你的可执行文件名' gmon.out
- 会按函数热度进行排序，百分比越大的函数就越热，可以针对TOPN函数进行分析。

但是由于我开发的是一个so，被其他进程调用。虽然在so的编译时加上了-pg，并且通过标志位将进程用exit退出，但是还是没有生成gmon.out，不知道怎么回事，姑且先放下，有机会再研究。


#### perf
perf功能很强大，而且被收录到内核(2.6.31)，可以记录page fault， cache miss，看起来真的很不错。

可惜我们的目标机内核版本太老，2.6.17，stackoverflow上有人回答说无法安装，

> Q: Does the old linux kernel support perf.

> A: No, it does not. The performance counters subsystem has undergone significant recent changes, and you are exceedingly unlikely to get perf working on any kernel below 2.6.31.

只好就此作罢。以后有机会再尝试。


#### sprof

继续搜索'Profiling shared library'，找到sprof。看了下简介，大概能满足需求，先拿来用用。

首先创建一个so，里边写一段比较耗时的代码

``` cpp
#include "lib.h"
#include <unistd.h>
#include <iostream>
#include <vector>
using namespace std;

void A::work(void)
{
	int a = 0;
	int b = 0;
	vector<int> v;
	while(a < 10)
	{
		b += a;
		++a;
		v.push_back(a);
		sleep(1);
	}
	cout << "work" << endl;
	return;
}
```


``` makefile
ll : libso
	g++ -L ./ -Wall -o test main.cpp -llib

libso : libo
	g++ -shared -o liblib.so liblib.o

libo : lib.cpp
	g++ -Wall -Werror -fpic -c lib.cpp -o liblib.o

clean:
	rm -f test

# export LD_LIBRARY_PATH=/home/username/foo:$LD_LIBRARY_PATH
```


``` sh
#set the environment variable LD_PROFILE to the name of the shared obj
export LD_PROFILE=my_obj
#run your application
my_app
#this should create a file /var/tmp/my_sobj.profile
#now run sprof
sprof my_sobj my_sobj.profile
```


查了一下，sprof只能采集可执行文件的性能，无法采集so的，还是需要放弃。



### gperf-tools

内网的文章带不出来，只能简单回忆，记录一下。
继续折腾之下，找到了google-perf-tools，功能很强大，可以采集so的性能，还能采集内存。

决定结合gtest，用采用单元测试来测性能内存。

#### 为什么要用单元测试呢？


1. 以前的做法比较麻烦。需要运行服务，然后kill平台进程等一系列操作。

2. 方便。项目已有单元测试框架，可以根据我们的需要自由组合场景，对外部环境依赖较小。

3. 运行快，几分钟就能看到结果。及时反馈，人脑在不断反馈的刺激下，会更专注。

4. 如果使用得当，可能做到自动化。

#### 安装libunwind



#### 安装gperftools



#### 设置环境变量



CPUPROFILE; exprot




#### enable单元测试




#### 改makefile

在单元测试可执行文件tester的makefile和我们so的makefile加上



``` makefile

CXXFLAGS += -g

LIBRARY += profiler

```



然后make

#### 写测试代码


1. 增加一个TEST，执行需要测性能的代码

2. 在setup里加上profilerStart

3. 在teardown加上profilerstop



#### 运行，然后分析结果



用 pprof --text (PATH OF TESTER) (CPUPROFILER执行的log)，得到文本格式的分析，包括每个函数的时间占有比例



用 pprof --callgrind可以生成图形化，很炫目。



-----------------------

`本博客欢迎转发,但请保留原作者信息`
github:[codejuan](https://github.com/CodeJuan)
博客地址:http://blog.decbug.com/

