---
date: 2020-10-16 21:52:00
title: windows 下以local system用户运行程序
author: Joden_He
tags: 
  - windows
  - local system
categories: 
  - windows
description: "在windows服务器下如何以local system账户运行指定程序？"
---



### 背景

最近在使用 windows 服务器部署微服务的时候，启动的 java 程序经常会被无故杀死，后面经过多次验证，发现是账号被 sign off 导致以改账号启动的程序也一律被杀死了。这个时候就想到一个解决方法，把 java 程序注册成 windows 的服务运行就可以避免账号被注销而导致进程被杀死的问题了（如何注册成服务可以参照 [nssm 注册 windows 服务](/2020/06/03/windows/nssm_services/)）。

由于涉及到的服务比较多，而且前期已经定好使用脚本启停的方式进行运维了，因此单个注册成服务需要花费的时间比较多，也不太好运维管理，那么还可以通过什么方式解决呢？

通过观察 windows 服务与我写的脚本启动服务方式的差异，不难发现，仅仅是启动账号不一样，服务是以 [SYSTEM](https://docs.microsoft.com/en-us/windows/win32/services/localsystem-account) 用户启动的，而 SYSTEM 是最高权限的用户，因此宿主用户注销不影响以 SYSTEM 用户运行启动的程序。

那现在的问题就是该怎么以 SYSTEM 用户运行程序了。

### 借助 `PSTools` 工具包以 `LocalSystem` 用户运行程序

> 可以通过该链接进行工具的下载 [https://docs.microsoft.com/en-us/sysinternals/downloads/pstools](https://docs.microsoft.com/en-us/sysinternals/downloads/pstools)，里面也用工具的介绍。我们主要使用到工具包中的 `PsExec.exe` （如果是64位系统也可以使用 `PsExec64.exe`）

以 `LocalSystem` 账户运行 xxx.jar 

```dos
psexec.exe -s java -jar xxx.jar
```

`-s` 就是以 system 身份运行

还有更多可选参数例如：`-i` 以交互模式运行，就是可以看到 java 的黑窗口；`-d` 执行命令后返回，不等待命令结束。更具体的参数可以通过命令 `psexec /h` 查看

> 所有的pstool第一次运行时都会弹框。可以用 `–accepteula` 这个参数

### 如何在当前用户下查找 `LocalSystem` 账号运行程序的 `pid` ?

由于使用脚本维护各服务 jar 包的启停，那么解决了怎么以 `LocalSystem` 身份运行程序了，但是以前的脚本中使用 `jps -l | find "xxx"` 来查找 `xxx` 程序的 `pid` 的操作已经不再适用了，那么应该怎么替换这段脚本代码呢？

可以通过以下命令获取 `pid`

```dos
wmic process where "commandline like '%xxx%' and name='java.exe'" get processid | findStr "[0-9]"
```

