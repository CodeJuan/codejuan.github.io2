---
layout: post
title: 比较WinPE文件(忽略时间戳)
date: 2014-11-19 00:00:00
categories:
- code
tags: 
- WinPE
mathjax: true
description: 
---
## 背景
项目组出现过比较有意思的情况：在同一台持续集成服务器，从同一个SVN上取指定版本的代码进行打包，但生成的两个版本有差异，即有DLL不相同导致运行结果不正确。经过定位，推测可能是update代码的过程中出现了异常，没有获取到期望的代码。
因此，CI工程师需要一种方法能够对DLL，EXE等二进制文件进行比对，以验证其正确性。但是即使用普通二进制比较工具（BeyondCompare）进行比较，发现总是有几个字节不相同。于是向我求助。

## 过程
我曾看过windows的PE文件结构，依稀记得里边会有几个字节存放TimeStamp和CheckSum，所以会有这么几个字节的差异。为避免误人子弟，特找来微软的定义看看：

``` cpp
typedef struct _IMAGE_NT_HEADERS {  
    DWORD Signature;  
    IMAGE_FILE_HEADER FileHeader;  
    IMAGE_OPTIONAL_HEADER32 OptionalHeader;  
} IMAGE_NT_HEADERS32, *PIMAGE_NT_HEADERS32;
```

<!--more-->

其中FileHeader的定义是，第三个变量就是时间戳，表示文件的创建时间。

``` cpp
typedef struct _IMAGE_FILE_HEADER {  
    WORD    Machine;  
    WORD    NumberOfSections;  
    DWORD   timeDateStamp;  //timeDateStamp
    DWORD   PointerToSymbolTable;  
    DWORD   NumberOfSymbols;  
    WORD    SizeOfOptionalHeader;  
    WORD    Characteristics;  
} IMAGE_FILE_HEADER, *PIMAGE_FILE_HEADER;
```

再来看看所谓的可选头OptionalHeader，其实一点都不可选，里边藏了好多东西，其中的CheckSum也会导致DLL差异。  

```cpp
typedef struct _IMAGE_OPTIONAL_HEADER {  
    WORD    Magic;     
    //...
    DWORD   CheckSum;  //CheckSum
    //...
    IMAGE_DATA_DIRECTORY DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES];  
} IMAGE_OPTIONAL_HEADER32, *PIMAGE_OPTIONAL_HEADER32;  
```

## 去除TimeStamp
既然时间戳确实存在，那么只能想办法在比较的时候忽略时间戳，或是在比较之前去除之。秉承一贯不重复造轮子的作风，先google一下是否有解决方案。

### BinDiff
微软大力推荐的BinDiff，可惜找了一大圈都没找到下载源，网上另有说法是BinDiff属于商用软件，license很贵。只能放弃，另寻他法。

### dumpbin /rawdata
VS2010自带了将PE文件导出的工具Dumpbin，据说可以将PE文件导出来。需要注意的是，不能直接在`CMD里输入dumpbin`，会提示找不到某个DLL，必须在开始菜单-VS2010-Tools-prompt里打开。区别在于tools里会先调用一个bat环境变量。
用法

```bat
dumpbin /rawdata 1.dll 1.txt
dumpbin /rawdata 2.dll 2.txt
```

用BeyondCompare比较1.txt和2.txt，发现差异还是在TimeStamp处，看来此法不通。

### dumpbin /disasm
查看dumpbin命令

>     /ALL  
      /ARCHIVEMEMBERS  
      /CLRHEADER  
      /DEPENDENTS  
      /DIRECTIVES  
      /DISASM[:{BYTES|NOBYTES}]  
      /ERRORREPORT:{NONE|PROMPT|QUEUE|SEND}  
      /EXPORTS  
      /FPO  
      /HEADERS  
      /IMPORTS[:文件名]  
      /LINENUMBERS  
      /LINKERMEMBER[:{1|2}]  
      /LOADCONFIG  
      /OUT:文件名  
      /PDATA  
      /PDBPATH[:VERBOSE]  
      /RANGE:vaMin[,vaMax]  
      /RAWDATA[:{NONE|1|2|4|8}[,#]]  
      /RELOCATIONS  
      /SECTION:名称  
      /SUMMARY  
      /SYMBOLS  
      /TLS  
      /UNWINDINFO   

其中有个disasm，看着像反汇编。如果能把PE反汇编出来，应该就能把TimeStamp去掉。

```bat
dumpbin /disasm 1.dll 1.txt
dumpbin /disasm 2.dll 2.txt
```

比较1.txt和2.txt果然一致。

## 后续
需要写一个bat，输入参数为两个文件夹的路径，将这两个文件夹内的PE文件都反汇编出来，然后一一进行比较，输出有差异的文件名。



-----------------------

`本博客欢迎转发,但请保留原作者信息`
github:[codejuan](https://github.com/CodeJuan)
博客地址:http://blog.decbug.com/

