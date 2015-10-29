---
layout: post
title: 万网免费主机搭博客 
date: 2015-01-31 23:30:09
categories:
- code
tags: 
- virtual
- zblog
mathjax: true
description:
---





最近万网搞活动，域名29，虚拟主机免费，有小朋友动了建博客的念头，于是申请好了域名、虚拟主机。但是不知道后续该如何处理，乐于助人的我自然是要帮他一把的。

## 选框架

小朋友选的是windows主机，简单搜索了一下，zblog貌似不错，好吧，就选它了，官网链接[http://www.zblogcn.com/](http://www.zblogcn.com/)，下载ASP版[http://www.zblogcn.com/zblog/](http://www.zblogcn.com/zblog/)，当前最新版的Z-Blog 2.2 Prism Build 140101。

解压后会看到一个release文件夹，由于后续要将release文件夹里的文件放在主机根目录，所以还要再压缩一次，格式还必须是rar，如下图所示
![](https://raw.githubusercontent.com/CodeJuan/codejuan.github.io/master/images/blog/zblog/zlog_rar.png)

<!--more-->

## 通过FTP上传安装包

把刚才压缩的rar，上传到FTP根目录，上传方法[http://help.www.net.cn/KnowledgeDetail.html?knowledgeId=5868398&categoryId=8311136](http://help.www.net.cn/KnowledgeDetail.html?knowledgeId=5868398&categoryId=8311136)


## 解压到根目录


![](https://raw.githubusercontent.com/CodeJuan/codejuan.github.io/master/images/blog/zblog/unzip.png)


![](https://raw.githubusercontent.com/CodeJuan/codejuan.github.io/master/images/blog/zblog/extract2root.png)

如果不解压到根目录，也是可以的，只是访问的时候不能直接输入域名了，将会是`你的域名/你的目录`


## 安装ZBLOG

打开http://你的域名/zb_install/default.asp(`万网的免费主机要备案，如果不备案，此时的网址就输入临时域名即可，临时域名还是在站点信息查询`)。


## 效果图
一路next，完成。效果如图


![](https://raw.githubusercontent.com/CodeJuan/codejuan.github.io/master/images/blog/zblog/blog.png)

-----------------------

`本博客欢迎转发,但请保留原作者信息`
github:[codejuan](https://github.com/CodeJuan)
博客地址:http://blog.decbug.com/

