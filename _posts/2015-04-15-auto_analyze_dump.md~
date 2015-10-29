---
layout: post
title: 自动调用windbg分析dump
date: 2015-04-15 23:30:09
categories:
- code
tags: 
- windbg
- powershell
- dump
mathjax: true
description:
---

## 前言

最近开始整并发，项目组的年轻人对共享资源的控制不够熟悉，导致经常core dump。虽然我在文档里写了，用windbg打开dmp，然后输入`!analyze -v`就能看到挂在哪一行，但是经常有小朋友来问命令怎么用。

老夫一怒之下，就开始反思。是我的文档写的不够清晰？还是操作太繁琐了？其实都不是，说白了就是现在的年轻人都太懒了。

竟然都这么懒惰，那么我只好想出个更简便的方法来分析dump，一键式傻瓜操作，这样应该可以了吧。

<!--more-->

## 初稿

通过powershell启动windbg，然后调用sendwait输入`!analyze -v`

``` powershell
[System.Reflection.Assembly]::LoadWithPartialName("'Microsoft.VisualBasic")
start-process windbg.exe -z "dump file 路径"
# {~}表示ENTER 
[System.Windows.Forms.SendKeys]::SendWait("!analyze -v{~}")
```

通过模拟键盘消息实现，在输入过程中必须保持焦点在windbg上，不允许动键盘鼠标，不够人性化。

还需要进一步完善


## 完善

微软关于windbg命令行的说明[https://msdn.microsoft.com/zh-cn/library/ff561306](https://msdn.microsoft.com/zh-cn/library/ff561306)


### initial command -c

`-c command`可以给windbg设置启动后的初始命令。

- Specifies the initial debugger command to run at start-up. 
- Multiple commands can be separated with semicolons.

既然可以设置命令，那么就可以抛弃powershell发键盘消息的方式了，直接写个batch搞定。

脚本

``` batch
"安装目录\windbg.exe" -z "dump file 路径" -c "!analyze -v"
```


### logo

有时候需要保存log，以前的做法是选中windbg的output，然后CTRL+C，CTRL+V到一个文本里。多次键盘鼠标操作，太麻烦，看看能否有命令行可以实现。

找到一个`-log{o|a} LogFile`

- Begins logging information to a log file. 
- If the specified log file already exists, it will be overwritten if -logo is used. 
- If loga is used, the output will be appended to the file. For more details, see Keeping a Log File.

脚本升级为

``` batch
"安装目录\windbg.exe" -z "dump file 路径" -c "!analyze -v" -logo "dump file 路径.log"
```


## 效果

每人每天分析3次dump，每次敲命令+拷贝log需要花1~3分钟的时间，团队里共有150个开发，一天就能节省150人*3次*2分钟=900分钟。

很可观的收益啊，不由得老怀大慰。

程序员就是要从机械繁琐的工作中超脱出来，投入到更有意义的事情上去。

-----------------------

`本博客欢迎转发,但请保留原作者信息`

github:[codejuan](https://github.com/CodeJuan)

博客地址:http://blog.decbug.com/

