---
layout: post
title: 抄代码引发的句柄泄漏
date: 2015-08-06 00:00:00
categories:
- code
tags: 
- leak
- handle
- process
- C++
mathjax: true
description: 
---


# 背景

## 现象
主进程不断调用7z.exe进行解压，当文件数量较小时，顺利运行。当文件数量达到几十万的时候，运行过程中7z报错，fatal error 2。
网上很多同僚说这是由于系统资源不足造成的。观察此时的内存及磁盘使用情况，都很充裕，但为何还说资源不足呢？于是开启了蛋疼的定位之旅，至于为什么说很蛋疼呢，这是因为是一个低级错误导致的问题。

心急的朋友可以直接看原理:[https://github.com/CodeJuan/HandleLeak](https://github.com/CodeJuan/HandleLeak)

不着急的朋友可以慢慢看定位过程

<!--more-->

# 过程

## 句柄泄漏

鉴于内存及硬盘都充足，那么猜测可能算句柄泄漏。先简单说下句柄泄漏的概念：

- 句柄，可以简单理解为某个资源的名字。
- 泄漏，跟内存泄漏一个泄漏，用了忘记释放，导致可用资源越来越少。
- A handle leak is a type of software bug that occurs when a computer program asks for a handle to a resource but does not free the handle when it is no longer used. If this occurs frequently or repeatedly over an extended period of time, a large number of handles may be marked in-use and thus unavailable, causing performance problems or a crash.

## 验证泄漏

把有问题的代码抠出来，写一个demo，循环跑。观察句柄情况。

```cpp
while(1)
{
    SHELLEXECUTEINFO ShExecInfo = {0};
    ShExecInfo.cbSize = sizeof(SHELLEXECUTEINFO);
    ShExecInfo.fMask = SEE_MASK_NOCLOSEPROCESS;
    ShExecInfo.hwnd = NULL;
    ShExecInfo.lpVerb = _T("open");
    ShExecInfo.lpFile = _T("cmd");
    ShExecInfo.lpParameters = _T("/c echo 111");
    ShExecInfo.lpDirectory = NULL;
    ShExecInfo.nShow = SW_HIDE;
    ShExecInfo.hInstApp = NULL;

    ShellExecuteEx(&ShExecInfo);

    WaitForSingleObject(ShExecInfo.hProcess,INFINITE); 
}
```

查看任务管理器中性能页显示的总句柄数，果然是不停在上涨，说明猜测成立

## 7z泄漏？

猜测可能算7z自身有泄漏，然而很快又否决了。
就像内存泄漏一样，当进程结束的时候，所有资源都会被系统回收，不会继续作恶下去。

> When the program terminates, all its open handles are closed
> Yes, a "memory leak" is simply memory that a process no longer has a reference to, and thus can no longer free. The OS still keeps track of all the memory allocated to a process, and will free it when that process terminates.

说明只是我们自己的主进程有泄漏

## 查看主进程句柄

通过任务管理器查看进程的句柄数，方法`选项-查看列-选中句柄计数`

句柄情况如图所示

运行一段时间后400+句柄

![](https://github.com/CodeJuan/HandleLeak/raw/master/pic/Leak1.JPG)

再过一段时间1000+句柄

![](https://github.com/CodeJuan/HandleLeak/raw/master/pic/Leak2.JPG)

接下来2000+句柄

![](https://github.com/CodeJuan/HandleLeak/raw/master/pic/Leak3.JPG)

看来真的是泄漏了

## 查看代码

看一下`SHELLEXECUTEINFOW`的定义
```cpp
typedef struct _SHELLEXECUTEINFOW
{
    DWORD cbSize;               // in, required, sizeof of this structure
    ULONG fMask;                // in, SEE_MASK_XXX values
    HWND hwnd;                  // in, optional
    LPCWSTR  lpVerb;            // in, optional when unspecified the default verb is choosen
    LPCWSTR  lpFile;            // in, either this value or lpIDList must be specified
    LPCWSTR  lpParameters;      // in, optional
    LPCWSTR  lpDirectory;       // in, optional
    int nShow;                  // in, required
    HINSTANCE hInstApp;         // out when SEE_MASK_NOCLOSEPROCESS is specified
    void *lpIDList;             // in, valid when SEE_MASK_IDLIST is specified, PCIDLIST_ABSOLUTE, for use with SEE_MASK_IDLIST & SEE_MASK_INVOKEIDLIST
    LPCWSTR  lpClass;           // in, valid when SEE_MASK_CLASSNAME is specified
    HKEY hkeyClass;             // in, valid when SEE_MASK_CLASSKEY is specified
    DWORD dwHotKey;             // in, valid when SEE_MASK_HOTKEY is specified
    union
    {
        HANDLE hIcon;           // not used
#if (NTDDI_VERSION >= NTDDI_WIN2K)
        HANDLE hMonitor;        // in, valid when SEE_MASK_HMONITOR specified
#endif // (NTDDI_VERSION >= NTDDI_WIN2K)
    } DUMMYUNIONNAME;
    HANDLE hProcess;            // out, valid when SEE_MASK_NOCLOSEPROCESS specified
} SHELLEXECUTEINFOW, *LPSHELLEXECUTEINFOW;
```

注意看`HANDLE hProcess;// out, valid when SEE_MASK_NOCLOSEPROCESS specified`

也就是说，如果指定了`SEE_MASK_NOCLOSEPROCESS`，`hProcess`就是返回的句柄。如果不关闭，就会造成句柄泄漏。

我们的代码为了等待进程结束，设置了`SEE_MASK_NOCLOSEPROCESS`，然后WaitForSingleObject(`ShExecInfo.hProcess`,INFINITE); 

然后又没有关闭`ShExecInfo.hProcess`，导致句柄不断上涨

## CloseHandle
加上`CloseHandle`之后 
```cpp
CloseHandle(ShExecInfo.hProcess);
```

句柄始终保持在100左右

![](https://github.com/CodeJuan/HandleLeak/raw/master/pic/CloseHandle.JPG)

修改正式代码，运行50万次依旧稳定，问题解决。




# 结论
询问组内同事，说这段代码是从CSDN上抄来的，没有深入了解代码的意思，没有注意到`SEE_MASK_NOCLOSEPROCESS`。
1. 抄代码一定要小心谨慎，需要仔细阅读官方说明，把参数的意义都理解清楚
2. 仅仅跑起来，凑合着用是不够的，需要做一做压力测试
3. 这次看句柄的方式比较落伍，需要整理一下如何用windbg查句柄泄漏的方法，下一篇就写这个吧。

# 参考
[HandleLeak](https://en.wikipedia.org/wiki/Handle_leak)
[Pushing the Limits of Windows: Handles](http://blogs.technet.com/b/markrussinovich/archive/2009/09/29/3283844.aspx)
[Is leaked memory freed up when the program exits?](http://stackoverflow.com/questions/2975831/is-leaked-memory-freed-up-when-the-program-exits)

-----------------------

`本博客欢迎转发,但请保留原作者信息`
github:[codejuan](https://github.com/CodeJuan)
博客地址:http://blog.decbug.com/

