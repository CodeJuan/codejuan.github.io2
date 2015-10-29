---
layout: post
title: 多线程同步镜像
date: 2015-09-07 00:00:00
categories:
- code
tags: 
- mirror
- rsync
- wget
- python
mathjax: true
description: 
---

## 背景
为了提高工作效率，我司的软件青年们搭建了一组内网mirror，包括apache，pip，npm，jenkins，ubuntu等等。
想用得爽，就得保持与官方一致，需要很频繁的同步。
以前的同步是单线程的，感觉没有完全发挥带宽的优势，所以想尝试一下多线程同步。


## 大概的思路
- 爬index，如hust的镜像[http://mirrors.hust.edu.cn/ubuntu-releases/](http://mirrors.hust.edu.cn/ubuntu-releases/)，找出所有`href`。
- 每个href就启动一个rsync或者wget，同步到对应的文件夹
- 记录每个href的同步状态
- 汇总全部状态，看所有href是否都同步成功
<!--more-->


## shell爬
### 步骤
- wget镜像的index.html
- 正则匹配，找到二级目录href-
- xargs -P 八个线程同时wget -r
- 有个问题，如何获取每个href的同步状态，成功还是失败？

### 代码
放在[https://github.com/CodeJuan/multi_thread_rsync_mirror](https://github.com/CodeJuan/multi_thread_rsync_mirror)
#### wget index.html
```sh
# get index.html
wget $link -O $file
```

#### regex得到二级目录
```sh
# grep "href" "<a" "/$"
# cut "\""
cat $bak_file | grep "href*=*\"" | grep "<a" | cut -d "\"" -f 2 | grep "/$" | grep -v "\.\."
```

#### xargs -P 八个线程同时wget -r
```sh
echo ${whole_url_list[@]} | xargs -n 1 -P 8 ./single_down.sh "$des"
```

#### 还没实现获取同步状态
如果下载失败了，完全没有办法知道。
下一步实现

## 另一种玩法：python爬
### 步骤

#### 装beautifulsoup
```sh
#下载并安装pip
wget https://pypi.python.org/packages/source/p/pip/pip-7.1.2.tar.gz
tar zxvf pip-7.1.2.tar.gz
cd pip-7.1.2
sudo python setup.py install

#装beautifulsoup4
sudo pip install beautifulsoup4
```



#### 开始爬
由于index的页面都很简洁，爬起来还是相对比较容易。

以后有空搞定～～

----------------------------

`本博客欢迎转发,但请保留原作者信息`                                                                                                                                                                          
github:[codejuan](https://github.com/CodeJuan)
博客地址:http://blog.decbug.com/

