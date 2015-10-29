---
layout: post
title: 学习GCC & GDB
date: 2014-11-26 00:00:00
categories:
- code
tags: 
- GCC
- GDB
mathjax: true
description: 
---

### 前言
工作是在windows下开发，业余时间才能玩自己喜欢的东东，一段时间不用就会生疏。随便写两段，加深记忆。

### GCC
从代码到可执行文件，会经历四个阶段，对应的命令是
> gcc -E  test.c -o test.i 中间文件
> gcc -S test.i -o test.s  ASM
> gcc -c test.s -o test.o  OBJ
> gcc test.o -o test       可执行文件

<!--more-->

当然，可以用一行命令搞定

> gcc -o test test.c


### GDB
回忆一下GDB的常用命令吧`l, b, r, watch, bt, n, step`
从[陈皓巨巨](http://coolshell.cn/)那A了段教程
``` cpp
#include <stdio.h>
     
int func(int n)
{
    int sum=0,i;
    for(i=0; i<n; i++)
    {
        sum+=i;
    }
    return sum;
}

main()
{
    int i;
    long result = 0;
    for(i=1; i<=100; i++)
    {
        result += i;
    }
    printf("result[1-100] = %d /n", result );
    printf("result[1-250] = %d /n", func(250) );
}
```

``` sh
gcc -g -o test test.c 	#-g表示gdb
gdb ./test 				#用gdb打开
l 						#显示代码
b linenum				#给指定行号价断点 bp
b function_name			#给函数加断点
i b						#info breakpoint显示断点，类似于windbg的bl
r						#run
n						#next
p var_name				#显示变量的值
step					#下一行
info locals				#显示local
watch i					#watch某变量
d 1						#delete breakpoint 1
c						#continue
q						#quit
```



### conclusion
还是要得写几段代码玩呃，拳不离手，曲不离口。。。。


-----------------------

`本博客欢迎转发,但请保留原作者信息`
github:[codejuan](https://github.com/CodeJuan)
博客地址:http://blog.decbug.com/

