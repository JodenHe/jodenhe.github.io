---
date: 2020-06-03 10:00:00
title: windows 下 cronolog 进行日志切割
author: Joden_He
tags: 
  - log
  - cronolog
categories: 
  - Java
  - cronolog
description: "在linux下我们可以使用 cronolog 对 tomcat 服务器的日志进行切割，那么在windows下是否同样可以使用 cronlog 呢？jar 包启动的方式怎么可以对日志进行切割呢？本文主要针对windows下jar包启动方式的日志
进行演示讲解，同时简要介绍一下linux系统下的操作。"
---

### 软件包下载地址

```txt
链接: https://pan.baidu.com/s/1xhhBKCthOxZuYBhEL4pb-Q 提取码: 9mxj
```

### jar 包日志分割

> 以 springboot 的jar为例，其他 jar 包类似

#### windows 环境

- 从上述地址👆下载对应系统的 cronolog 包

  ![cronolog-1.6.1-win32.zip](/images/java/cronolog/20200603133210.png)

- 解压存放到指定目录，例如 `d:\cronolog`

  ![](/images/java/cronolog/20200603133932.png)

- 执行 jar 启动命令

  ```sh
  java -jar xxx.jar | d:\cronolog\cronolog.exe 日志输出路径\xxx-%%Y%%m%%d.log
  ```

  > 要是想后端运行可以编写bat启动脚本

  ```bat
  @echo off
  
  javaw -jar xxx.jar | d:\cronolog\cronolog.exe 日志输出路径\xxx-%%Y%%m%%d.log
  
  exit
  ```

#### linux 环境

- 下载安装包 `cronolog-1.6.2.tar.gz`

  ```sh
  wget http://cronolog.org/download/cronolog-1.6.2.tar.gz 
  # 或者
  wget https://files.cnblogs.com/files/crazyzero/cronolog-1.6.2.tar.gz
  ```

- 解压

  ```sh
  tar zxvf cronolog-1.6.2.tar.gz
  ```

- 安装

  ```sh
  cd cronolog-1.6.2
  ./configure
  make
  sudo make install
  ```

- 验证

  ```sh
  which cronolog
  # 输出 /usr/local/sbin/cronolog
  ```

- 执行 jar 启动命令

  ```sh
  java -jar xxx.jar | /usr/local/sbin/cronolog 日志输出路径\xxx-%Y%m%d.log
  ```

  > 要是想后端运行可以使用以下命令

  ```sh
  nohub java -jar xxx.jar | /usr/local/sbin/cronolog 日志输出路径\xxx-%Y%m%d.log &
  ```

### tomcat 进行日志分割

> 这个内容网上很多资源，在此不展开说，例如：https://blog.csdn.net/fly910905/article/details/78528652