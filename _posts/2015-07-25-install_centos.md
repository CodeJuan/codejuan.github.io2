---
layout: post
title: 在家里玩分布式(1)install centos 
date: 2015-07-25 00:00:00
categories:
- code
tags: 
- system
mathjax: true
description: 
---


# 扯淡
自己在家玩，买了一堆二手配件，整了4个台式机，打算
    - 弄个scrapy爬数据
    - 存到分布式内存数据库(redis吧)
    - 再存到分布式文件系统(打算用fastdfs)
    - 同时用storm流计算，弄个什么东西挖掘一下
    - 最后用PHP展示

也勉强算是跟大数据沾点边，几台机器都自动化起来，也get以下运维技能吧。

<!--more-->

# centos
## 下载iso
[http://www.centos.org/download/](http://www.centos.org/download/)
- minimal: The aim of this image is to install a very basic CentOS system, with the minimum of packages needed to have a functional system
- dvd: 一般选择这个
下载x86_64

## 分区
/、/boot、/home、swap 就够了

## 预装软件
选择开发者模式，可以把python jdk都装上

## 设置自动登录
```bash
vi /etc/gdm/custom.conf
```
写上
```
[daemon]
AutomaticLoginEnable=true
AutomaticLogin=你的用户名
```

## 启动网络
```bash
sudo ntsysv
# TAB切换到OK
```
修改`/etc/sysconfig/network-scripts/ifcfg-enp2s0 `,设置为`onboot=yes`


## 安装启动SSH
```sh
yum install ssh
service sshd start
chkconfig sshd on
```

## 路由器设置固定IP
- 查看每台机器的MAC ifconfig
- 进入路由器管理页面，DHCP - 静态地址分配 - 绑定MAC和IP

## 安装vim git
```sh
yum -y install vim
yum -y install git
```

```sh
git clone https://github.com/CodeJuan/config.git
cp config/.vimrc ~/
cat config/.gitconfig >> ~/.gitconfig
```

# 装爬虫
## 装pip
```sh
wget --no-check-certificate  https://github.com/pypa/pip/archive/7.1.0.tar.gz
tar zvxf 7.1.0.tar.gz    #解压文件
cd pip-7.1.0
sudo python setup.py install
```

## 装scrapy
```sh
sudo pip install scrapy
```


-----------------------

`本博客欢迎转发,但请保留原作者信息`
github:[codejuan](https://github.com/CodeJuan)
博客地址:http://blog.decbug.com/

