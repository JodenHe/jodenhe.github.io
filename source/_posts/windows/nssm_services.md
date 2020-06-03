---
date: 2020-06-03 15:56:00
title: nssm 注册 windows 服务
author: Joden_He
tags: 
  - windows service
  - nssm
  - 服务

categories: 
  - windows
  - services

description: "介绍如何使用 nssm 注册 windows 服务"
---



### 背景

在 Windows 服务器中启用 jar 应用，通常使用的命令是

```sh
java -jar xx.jar
```

但是这种情况启动了，我们需要保持命令提示框的打开状态，所以萌生了个想法，要是我们将启动bat脚本注册成windows 服务是不是就可以优雅的管理我们的 java 应用了。

### NSSM 介绍

**[NSSM](https://nssm.cc/)**(the Non-Sucking Service Manager)是Windows环境下一款**免安装**的服务管理软件，它可以将应用封装成服务，使之像windows服务可以设置自动启动等。并且可以监控程序运行状态，程序异常中断后自动启动，实现守护进程的功能。不仅支持图形界面操作，也完全支持命令行设置。

### NSSM 下载与配置

- 进入下载界面，选择最新稳定版进行下载，https://nssm.cc/download

  ![NSSM 下载界面](/images/windows/services/20200603161326.png)

- 解压到特定的目录，例如 `D:\nssm-2.24`，文件夹里面又 `win32` 和 `win64` 两个文件夹，根据系统的位数选择合适的应用，这里采用 `win64`

  ![nssm 目录](/images/windows/services/20200603161728.png)

- 配置环境变量

  将 nssm 解压的路径配置到环境变量的 path 里面，如：`D:\nssm-2.24\win64`

  ![nssm 环境变量](/images/windows/services/20200603162145.png)

- 验证安装

  cmd 命令窗输入 `nssm`

  ![nssm 验证安装](/images/windows/services/20200603162750.png)

### 将bat注册成服务

- 新建文件 `start.bat` 用来启动 jar 文件

  ```bat
  @echo off
  
  rem 启动jar
  java -jar xx.jar
  ```

  

- 新建文件 `install.bat` 用来注册服务

  ```bat
  @echo off
  
  rem 当前路径
  set currpath=%cd%
  rem 服务名，自己修改
  set servicename=xxx
  rem 服务描述
  set servicedesc=xxxx
  rem 启动路径，对应start.bat路径
  set binpath=%currpath%\start.bat
  rem 启动参数
  set appParams=
  
  rem 注册服务
  nssm install %servicename% %binpath%
  rem 修改服务描述
  sc description %servicename% "servicedesc"
  
  rem 修改注册表
  set regpath=HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\%servicename%\Parameters\
  
  reg edit %regpath% /v AppParameters /t REG_SZ /d "%appParams%" /f
  reg edit %regpath%\AppExit\ /t REG_SZ /d "Ignore" /f
  ```

  

- 双击 `install.bat`，即可

