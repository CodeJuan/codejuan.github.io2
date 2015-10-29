---
layout: post
title: 初学docker
date: 2015-08-23 00:00:00
categories:
- code
tags: 
- docker
- centos
mathjax: true
description: 
---
## 背景
虽然久闻docker大名，但一直没有尝试过，固然可以说是比较忙的原因，但忙不是我们不学习新技术的借口。
也许以后工作可能会用到docker，我得未雨绸缪，提前准备，开始学docker了

<!--more-->

## 安装
参考[installation on CentOS](http://docs.docker.com/installation/centos/)

### 查看内核版本

>Docker requires a 64-bit installation regardless of your CentOS version. Also, your kernel must be 3.10 at minimum, which CentOS 7 runs.
要求centos7 64-bit

```sh
uname -r
```
我的是CentOS7，`3.10.0-229.el7.x86_64`，符合要求

### 用脚本安装

1. yum package最新
```sh
sudo yum update
```
2. Run the Docker installation script.
```sh
curl -sSL https://get.docker.com/ | sh
```
This script adds the `docker.repo` repository and installs Docker.
3. Start the Docker daemon.
```sh
sudo service docker start
```
失败了，提示`Job for docker.service failed. See 'systemctl status docker.service' and 'journalctl -xn' for details.`
查了一下，[#15498](https://github.com/docker/docker/issues/15498)有人说要装docker-selinux
```sh
sudo yum install docker-selinux
```
然后
```sh
sudo service docker start
```
果然complete
4. verify
```sh
sudo docker run hello-world
```

```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
535020c3e8ad: Pull complete 
af340544ed62: Already exists 
library/hello-world:latest: The image you are pulling has been verified. Important: image verification is a tech preview feature and should not be relied on to provide security.
Digest: sha256:d5fbd996e6562438f7ea5389d7da867fe58e04d581810e230df4cc073271ea52
Status: Downloaded newer image for hello-world:latest

Hello from Docker.
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker Hub account:
 https://hub.docker.com

For more examples and ideas, visit:
 https://docs.docker.com/userguide/
```

## 进阶



------------------------------

`本博客欢迎转发,但请保留原作者信息`                                                                                                                                                                          
github:[codejuan](https://github.com/CodeJuan)
博客地址:http://blog.decbug.com/

