---
layout: post
title: 自动生成sln中各project依赖图
date: 2015-07-06 00:00:00
categories:
- code
tags: 
- dependency
- graphviz
- dot
mathjax: true
description: 
---

# 前言

为了提高构建效率，需要分析sln中各project的依赖关系，将相互独立的project分配到不同机器并行构建。如果是一个个打开工程并查看dependency，然后画出依赖图，实在是太繁琐了。这就需要将这一些工作用脚本来实现。

写好的代码放在[https://github.com/CodeJuan/dependency_in_sln](https://github.com/CodeJuan/dependency_in_sln)，使用方法也很简单。

<!--more-->

# 思考

## 采用的技术

> `Graphviz` is open source graph visualization software. Graph visualization is a way of representing structural information as diagrams of abstract graphs and networks. It has important applications in networking, bioinformatics,  software engineering, database and web design, machine learning, and in visual interfaces for other technical domains.


> `DOT` is a plain text graph description language. It is a simple way of describing graphs that both humans and computer programs can use.

graphviz是画图神器，dot可以描述图，二者结合，就能画出各种神奇的图片

## 步骤

1. 解析sln，得出各工程的依赖关系
2. 依据第1步的依赖关系生成dot文件
3. graphviz调用第2步的dot文件，生成图片

# 实施

## sln规律
先看一段例子
```
Project("{8BC9CEB8-8B4A-11D0-8D11-00A0C91BC942}") = "gtest_unittest", "gtest_unittest.vcxproj", "{4D9FDFB5-986A-4139-823C-F4EE0ED481A1}"
	ProjectSection(ProjectDependencies) = postProject
		{24848551-EF4F-47E8-9A9D-EA4D49BC3ECA} = {24848551-EF4F-47E8-9A9D-EA4D49BC3ECA}
		{C8F6C172-56F2-4E76-B5FA-C3B423B31BE7} = {C8F6C172-56F2-4E76-B5FA-C3B423B31BE7}
	EndProjectSection
EndProject
```
- project的开头
```
  Project("{8BC9CEB8-8B4A-11D0-8D11-00A0C91BC942}") = "gtest_unittest", "gtest_unittest.vcxproj", "{4D9FDFB5-986A-4139-823C-F4EE0ED481A1}" # 格式是Project("{sln guid}") = "project name", "relative path", "{project guid}"
```
- project的结尾
```
EndProject
```
- 依赖关系
```
ProjectSection(ProjectDependencies) = postProject # 依赖了哪些工程的开头
{24848551-EF4F-47E8-9A9D-EA4D49BC3ECA} = {24848551-EF4F-47E8-9A9D-EA4D49BC3ECA} #依赖工程的guid
EndProjectSection # 依赖描述结尾
```

发现这样的规律，就能够很方便的解析了

## 代码
由于是在windows上面解析，有同事不会用shell，于是只好用powershell重写一遍

遍历sln的每一行，进行分析，并写入dot
```powershell
    if ($line -like "Project(`"{*") #如果匹配到了project开头
    {
        $array = $line.split("`"");
        $name = $array[3]
        $id = $array[7]
        $script:prj_name_list += $name
        $script:prj_id_list += $id
        append "$name[shape=box,fontname=consolas];" # 在dot写入这个project的描述
    }
```

``` powershell
	if ($line -like "*ProjectSection(*")
    {
        $name = $script:prj_name_list[$script:prj_name_list.length - 1]
        append "$name->{" # 匹配到依赖了哪些工程的开头，开始写入依赖关系
    }
```

``` powershell
	if ($line -like "*EndProjectSection*")
    {
        append "};" # 匹配到了依赖描述结尾，写入};，完成了这个project的依赖描述
    }
```

``` powershell
    if ($line -like "*{*} = {*}*") # 通过大括号识别是否是工程依赖
    {
        $right = $line.lastindexof("}")
        $left = $line.lastindexof("{")
        $dep_id = $line.substring($left+1, $right-$left-1)
        for ($i = 0; $i -lt $script:prj_id_list_query.length; $i++)
        {
            $cur_id = $script:prj_id_list_query[$i]
            if ( $dep_id -like "$cur_id")
            {
                $dep_name = $script:prj_name_list_query[$i]
                append "$dep_name;"
            }
        }
    }
```


## 生成的dot
``` 
digraph G {
rankdir=BT;
gtest[shape=box,fontname=consolas];
gtest_main[shape=box,fontname=consolas];
gtest_unittest[shape=box,fontname=consolas];
gtest_unittest->{
gtest_prod_test;
gtest;
};
gtest_prod_test[shape=box,fontname=consolas];
}
```

## demo
![](https://github.com/CodeJuan/dependency_in_sln/raw/master/gtest.sln.png)

-----------------------

`本博客欢迎转发,但请保留原作者信息`
github:[codejuan](https://github.com/CodeJuan)
博客地址:http://blog.decbug.com/

