---
layout: post
title: 一键式给VC工程创建UT工程
date: 2014-12-24 23:30:09
categories:
- code
tags: 
- UT
- automation
mathjax: true
description:
---


## 前言
项目组要求给已有的几十个VC工程添加配套的UT工程，需要覆盖到每个类(即每个CPP都要有对应的TEST)。简单观察了一下，还是选用add existing item to project添加.cpp.h的方法最为简单。

## 人肉创建
验证此方法是否可行

1. 把某工程的vcxproj及filter拷贝到UT目录
2. 替换掉vcxproj里的CIinclude, resourceInclude的路径为相对路径
3. additional path加入gtest和gmock的头文件及lib
4. def也要改成相对路径
5. additional Include path 要加上原有工程的路径
6. application type 改为 exe
7. link-system-subsystem改为console
8. gtest gmock的runtime lib都改为/mdd

<!--more-->

#### 果然可以

效果如图

![](https://github.com/CodeJuan/codejuan.github.io/raw/master/images/blog/ut_migrate/UT_gtest.png)


## powershell 脚本

说白了就是用脚本处理vcxproj(其实就xml)，把上文提到的几个步骤都用脚本实现。

``` powershell  
$path= gi .\abc.xml
$xmldata = [xml](Get-Content $path)
$xmldata.rss.channel.title
$xmldata.rss.channel.title="abc"
$xmldata.save($path.fullname)
```  

## conclusion

花了半天的时间把创建方法及脚本写好，省去N个人的重复劳动。通过脚本实现，还不容易出错。
YEAH!!


-----------------------

`本博客欢迎转发,但请保留原作者信息`
github:[codejuan](https://github.com/CodeJuan)
博客地址:http://blog.decbug.com/

