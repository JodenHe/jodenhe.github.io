---
title: Rabbit MQ 安装
date: 2021-10-09 14:37:32
author: Joden_He
tags:
  - software
categories: 
  - software
description: "记录 centos7 下安装 Rabbit MQ"
---

## 前言

Rabbit MQ 是由 Erlang 语言编写的，也正因如此，在安装 Rabbit MQ 之前需要安装 Erlang 。建议采用较新版的 Erlang，这样可以获得较多更新和改进，可以到官网 (https://www.erlang.org/downloads) 下载

## 安装 Erlang

进入 Erlang 下载网址，下载源码包

![Erlang 源码包下载](/images/other/20211009144350.png)

### 解压安装包，并配置安装目录

这里我们安装到 `/opt/erlang` 目录下:

```shell
[joden@localhost workspace]$ tar -zxvf otp_src_24.1.tar.gz
[joden@localhost workspace]$ cd otp_src_24.1
[joden@localhost otp_src_24.1]$ ./configure --prefix=/opt/erlang
=== Running configure in /home/joden/workspace/otp_src_24.1/erts ===
./configure '--prefix=/opt/erlang' --disable-option-checking --cache-file=/dev/null --srcdir="/home/joden/workspace/otp_src_24.1/erts"
checking build system type... x86_64-pc-linux-gnu
checking host system type... x86_64-pc-linux-gnu
checking for gcc... no
checking for cc... no
checking for cl.exe... no
configure: error: in `/home/joden/workspace/otp_src_24.1/erts':
configure: error: no acceptable C compiler found in $PATH
See `config.log' for more details
ERROR: /home/joden/workspace/otp_src_24.1/erts/configure failed!
已杀死
[joden@localhost otp_src_24.1]$
```

按提示报错没有GCC编译器环境）

因此先来安装一下 GCC

```shell
[joden@localhost otp_src_24.1]$ sudo yum install gcc

```

重新执行命令 `./configure --prefix=/opt/erlang`，依然报错 `configure: error: No curses library functions found` 那么此时需要安装 ncurses，安装步骤 (遇到提示输入y 后直接回车即可) 如下:

```shell
[joden@localhost otp_src_24.1]$ sudo yum install ncurses-devel
```

再次执行命令 `./configure --prefix=/opt/erlang` 成功！！

### 编译及安装

```shell
[joden@localhost otp_src_24.1]$ make
[joden@localhost otp_src_24.1]$ sudo make install
```

> 如果在安装的过程中出现类似"No xxx found" 的提示，可根据提示信息安装相应的包，然后重新执行，直到提示安装完毕为止。

### 配置环境变量

修改 `/etc/profile` 配置文件，添加下面的环境变量:

```tex
export ERLANG_HOME=/opt/erlang
export PATH=$PATH:$ERLANG_HOME/bin
```

最后执行 `source /etc/profile` 命令让配置文件生效:

```shell
[joden@localhost otp_src_24.1]$ sudo vim /etc/profile
[joden@localhost otp_src_24.1]$ source /etc/profile
```

可以输入 `erl` 命令来验证 Erlang 是否安装成功，如果出现类似以下的提示即表示安装成功:

```shell
[joden@localhost otp_src_24.1]$ erl
Erlang/OTP 24 [erts-12.1] [source] [64-bit] [smp:3:3] [ds:3:3:10] [async-threads:1]

Eshell V12.1  (abort with ^G)
1>
```

## 安装 Rabbit MQ

Rabbit MQ 的安装直接将下载的安装包解压到相应的目录下即可，官网下载地址: https://www.rabbitmq.com/install-generic-unix.html。

![rabbit mq 下载](/images/other/20211009152620.png)

这里选择将 Rabbit MQ 安装到与 Erlang 同一个目录 (/opt) 下面:

```shell
[joden@localhost workspace]$ tar -xvf rabbitmq-server-generic-unix-3.9.7.tar.xz
[joden@localhost workspace]$ sudo mv rabbitmq_server-3.9.7/ /opt/
[joden@localhost workspace]$ cd /opt/
[joden@localhost opt]$ sudo mv rabbitmq_server-3.9.7/ rabbitmq
```

同样修改/etc/profile 文件， 添加下面的环境变量:

```tex
export RABBITMQ_HOME=/opt/rabbitmq
export PATH=$PATH:$RABBITMQ_HOME/sbin
```

执行 `source /etc/profile` 命令让配置文件生效。

验证是否安装成功

```shell
[joden@localhost opt]$ rabbitmqctl version
3.9.7
```

