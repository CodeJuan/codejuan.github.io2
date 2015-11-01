---
layout: post
title: 絮叨ansible做持续交付,闲扯分布式
date: 2015-11-01 00:00:00
categories:
- code
tags: 
- distributed
- ansible
- continous delivery
mathjax: true
description: 
---


# 随手记录

杂记而已，随便记录


-----------------------


# ansbile做持续交付

突然想到，ansible可以控制一大堆机器，那么是否可以像jenkins的master/slave那样，做分布式构建呢。
应该是可以的吧
- 把代码同步到各个agent上，可以用git/svn都行
- 通过一堆参数，控制各个agent做不同的事情，并行构建。比如编译，静态检测，单元测试，自动化测试等等
- artifact收集
- 部署

无缝接入呀，有时间可以试试。

-----------------------

# 分布式的一些感想

想到一句话，`不谋全局者，不足以谋一域；不谋万世者，不足以谋一时`。
- 以前把玩有限的几台服务器，更多的是关注单点的CPU/IO/内存等等，现在得关注整体的性能，找出拖慢整体性能的瓶颈。
- 在考虑方案的时候，哪怕当前只有10台服务器，最好是按照1000台的规模去设计。

哈哈哈～～

-----------------------

`本博客欢迎转发,但请保留原作者信息`

github:[codejuan](https://github.com/CodeJuan)

博客地址:http://blog.decbug.com/
