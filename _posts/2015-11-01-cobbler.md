---
layout: post
title: cobbler自动部署系统
date: 2015-11-01 00:00:00
categories:
- code
tags: 
- deploy
- cobbler
- system
mathjax: true
description: 
---


# 背景

我司竟然还是人肉装系统，太TMD老土了。于是找到了`cobbler`，官网在[http://cobbler.github.io/](http://cobbler.github.io/)

先看一段简介

> Cobbler is a Linux installation server that allows for rapid setup of network installation environments.

很叼吧。

<!--more-->

# 安装cobbler

参考官网的quick start [http://cobbler.github.io/manuals/quickstart/](http://cobbler.github.io/manuals/quickstart/)

## disable SELinux

由于我对SELinux不熟悉，根据官网的建议，还是把SELinux Disable吧

参考[https://www.centos.org/docs/5/html/5.1/Deployment_Guide/sec-sel-enable-disable.html](https://www.centos.org/docs/5/html/5.1/Deployment_Guide/sec-sel-enable-disable.html)

修改`/etc/sysconfig/selinux`，修改`SELINUX`的值为`permissive`，并增加一行`SETLOCALDEFS=0`
```
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=enforcing  # 改为 disabled
# SELINUXTYPE= can take one of three two values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected. 
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted 

```

## Installing Cobbler

### 错误的方法
```bash
sudo yum install cobbler
```
提示没有package，说明要添加源。
按照[http://cobbler.github.io/manuals/2.4.0/3/2_-_Installing_From_Packages.html](http://cobbler.github.io/manuals/2.4.0/3/2_-_Installing_From_Packages.html)说
```bash
# sudo rpm -Uvh http://download.fedoraproject.org/pub/epel/6/x86_64/epel-release-X-Y.noarch.rpm
  sudo rpm -Uvh http://download.fedoraproject.org/pub/epel/7/x86_64/epel-release-7-0.noarch.rpm
```
还是不行，因为我不知道具体的版本号。

只好找到最新release的页面[http://cobbler.github.io/posts/2015/09/30/cobbler_2.6.10_released.html](http://cobbler.github.io/posts/2015/09/30/cobbler_2.6.10_released.html)，根据`Packages will be provided as soon as possible, please check`的提示，找到[http://download.opensuse.org/repositories/home:/libertas-ict:/cobbler26/CentOS_CentOS-7/noarch/](http://download.opensuse.org/repositories/home:/libertas-ict:/cobbler26/CentOS_CentOS-7/noarch/)

添加源
```
sudo rpm -Uvh http://download.opensuse.org/repositories/home:/libertas-ict:/cobbler26/CentOS_CentOS-7/noarch/cobbler-2.6.10-11.2.noarch.rpm
sudo rpm -Uvh http://download.opensuse.org/repositories/home:/libertas-ict:/cobbler26/CentOS_CentOS-7/noarch/cobbler-web-2.6.10-11.2.noarch.rpm
sudo rpm -Uvh http://download.opensuse.org/repositories/home:/libertas-ict:/cobbler26/CentOS_CentOS-7/noarch/koan-2.6.10-11.2.noarch.rpm
```
提示缺少python的一堆库，
```
python-simplejson is needed by cobbler-2.6.10-11.2.noarch
python-cheetah is needed by cobbler-2.6.10-11.2.noarch
```
使用pip安装simplejson和cheetah，还是报这个错，看来此路不通，需要另想它法。

### 正确的方法
找到了这个链接[http://cobbler.readthedocs.org/en/latest/installation-guide.html](http://cobbler.readthedocs.org/en/latest/installation-guide.html)

Make sure you have the EPEL repository enabled on your system:
```bash
yum -y install epel-release
yum repolist
# sudo curl -o cobbler30.repo http://download.opensuse.org/repositories/home:/libertas-ict:/cobbler30/CentOS_CentOS-7/home:libertas-ict:cobbler30.repo
```

接下来
```bash
yum install cobbler cobbler-web
```
就安装成功了

## 启动cobbler

### 改配置
`/etc/cobbler/settings`
```
default_password_crypted: "$1$bfI7WLZz$PxXetL97LkScqJFxnW7KS1" # 123456
openssl passwd -1

next_server: 192.168.161.51
server: 192.168.161.51

我是通过路由分配IP，就不设置manage_dhcp

```

```bash
sudo service httpd start
sudo service xinetd start
sudo service cobblerd start

sudo chkconfig cobblerd on
sudo chkconfig xinetd on
sudo chkconfig httpd on
```


检查配置
```bash
sudo cobbler check
```
提示
```
1 : SELinux is enabled. Please review the following wiki page for details on ensuring cobbler works correctly in your SELinux environment:
    https://github.com/cobbler/cobbler/wiki/Selinux
2 : change 'disable' to 'no' in /etc/xinetd.d/tftp
3 : some network boot-loaders are missing from /var/lib/cobbler/loaders, you may run 'cobbler get-loaders' to download them, or, if you only want to handle x86/x86_64 netbooting, you may ensure that you have installed a *recent* version of the syslinux package installed and can ignore this message entirely.  Files in this directory, should you want to support all architectures, should include pxelinux.0, menu.c32, elilo.efi, and yaboot. The 'cobbler get-loaders' command is the easiest way to resolve these requirements.
4 : file /etc/xinetd.d/rsync does not exist
5 : debmirror package is not installed, it will be required to manage debian deployments and repositories
6 : The default password used by the sample templates for newly installed machines (default_password_crypted in /etc/cobbler/settings) is still set to 'cobbler' and should be changed, try: "openssl passwd -1 -salt 'random-phrase-here' 'your-password-here'" to generate new one
7 : fencing tools were not found, and are required to use the (optional) power management features. install cman or fence-agents to use them
```
根据提示一一修改
解决方法

1. disable selinux
2. 改配置文件
3. 执行cobbler get-loaders
4. 新建/etc/xinetd.d/rsync，增加disable = no
```
修改 rsync 和 tftp 这两个服务的 xinetd 配置：

新建一个
# vi /etc/xinetd.d/rsync
service rsync
{
        disable = no
}

已改
# vi /etc/xinetd.d/tftp
service tftp
{

        disable = no

}
```
5. 不支持debian系，需要装
```
暂时不管，我这里只测试centos
cobbler服务器能同时部署CentOS/Fedora/Debian/Ubuntu系统，所以需要安装debmirror，安装debmirror-20090807-1.el5.noarch.rpm，在此之前，需要先安装一些其他的依赖包：

# yum install ed patch perl perl-Compress-Zlib perl-Cwd perl-Digest-MD5 perl-Digest-SHA1 perl-LockFile-Simple perl-libwww-perl
# wget ftp://fr2.rpmfind.net/linux/epel/5/ppc/debmirror-20090807-1.el5.noarch.rpm
# rpm –ivh debmirror-20090807-1.el5.noarch.rpm

修改/etc/debmirror.conf 配置文件，注释掉 @dists 和 @arches 两行

# vim /etc/debmirror.conf
…
#@dists=”sid”;
@sections=”main,main/debian-installer,contrib,non-free”;
#@arches=”i386″;
```
6. 生成密码
```
修改默认系统密码用 openssl 生成一串密码后加入到 cobbler 的配置文件（/etc/cobbler/settings）里，替换 default_password_crypted 字段：
# openssl passwd -1 -salt ‘bihan’ ‘Abcd1234′
$1$‘bihan$bndMeAmxTpT0ldGYQoRSw0
# vi /etc/cobbler/settings
修改内容如下：
default_password_crypted: “$1$‘bihan$bndMeAmxTpT0ldGYQoRSw0″
```
7. yum install cman或者fence-agents，我装的是fence-agents

改完之后运行
```bash
sudo service cobblerd restart
sudo cobbler sync
# 再check一下
sudo cobbler check
```
就只剩下debmirror的问题了，可以暂时不管

下载并挂载iso
```bash
wget wget http://mirrors.sina.cn/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-1503-01.iso

#sudo mount -t iso9660 -o loop,ro ./CentOS-7-x86_64-Minimal-1503-01.iso /mnt
sudo mount -t iso9660 -o loop,ro /home/i3/save/iso/CentOS-7-x86_64-Minimal-1503-01.iso /mnt/centos
#sudo cobbler import --name=centos7 --arch=x86_64 --path=/mnt
sudo cobbler import --name=centos7 --arch=x86_64 --path=/mnt/centos
#sudo vi /etc/fstab
# 增一行/home/i3/save/iso/CentOS-7-x86_64-Minimal-1503-01.iso   /home/i3/save/cobbler_os iso9660 defaults,ro,loop  0 0

#umount /somedir
```

挂载时报错
```
Adding distros from path /var/www/cobbler/ks_mirror/centos7-x86_64:
creating new distro: centos7-x86_64
trying symlink: /var/www/cobbler/ks_mirror/centos7-x86_64 -> /var/www/cobbler/links/centos7-x86_64
creating new profile: centos7-x86_64
Exception occured: <type 'exceptions.UnicodeEncodeError'>
Exception value: 'ascii' codec can't encode character u'\u2018' in position 3: ordinal not in range(128)
Exception Info:
  File "/usr/lib/python2.7/site-packages/cobbler/remote.py", line 87, in run
    rc = self._run(self)
```

删除
```bash
sudo cobbler profile remove --name=centos7-x86_64
sudo cobbler distro remove --name=centos7-x86_64
```



-----------------------

`本博客欢迎转发,但请保留原作者信息`

github:[codejuan](https://github.com/CodeJuan)

博客地址:http://blog.decbug.com/
