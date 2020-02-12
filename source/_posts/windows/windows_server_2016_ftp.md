---
date: 2020-02-12 00:00:00
title: Windows Server 2016 搭建 FTP
author: Joden_He
tags: 
  - windows server
  - windows server 2016
  - ftp

categories: 
  - windows
  - ftp

description: 从零开始在 Windows Server 2016 上搭建 FTP 服务
---



### 安装 WEB 服务器

通过服务器添加角色和功能向导添加 `Web 服务器(IIS)`

服务器管理器 —— 添加角色功能 —— Web 服务器(IIS)

![web 服务器](/images/windows/server/2016/ftp/1.png)

下一步

![web 服务器](/images/windows/server/2016/ftp/2.png)

下一步

![web 服务器](/images/windows/server/2016/ftp/3.png)

下一步

![web 服务器](/images/windows/server/2016/ftp/4.png)

选择 ftp 并且点击下一步

![web 服务器](/images/windows/server/2016/ftp/5.png)

安装

![web 服务器](/images/windows/server/2016/ftp/6.png)

关闭

![web 服务器](/images/windows/server/2016/ftp/7.png)



### 添加 FTP 站点

在 `计算机管理` 中根据提示添加即可

![FTP 站点](/images/windows/server/2016/ftp/8.png)

填写自定义信息，下一步

![FTP 站点](/images/windows/server/2016/ftp/9.png)

填写信息，下一步

![FTP 站点](/images/windows/server/2016/ftp/10.png)

根据需要配置权限信息

![FTP 站点](/images/windows/server/2016/ftp/11.png)



### 打开IE浏览器，在`Internet 选项` 中设置 `使用被动 FTP`

![Internet Options](/images/windows/server/2016/ftp/12.png)

![Internet Options](/images/windows/server/2016/ftp/13.png)



### 防火墙开放端口

在服务器管理中打开 高级安全 Windows 防火墙，点击 入站规则 —— 新建规则，根据提示添加远程端口（默认为21端口），建议将端口设置为非 21 端口。

![防火墙](/images/windows/server/2016/ftp/14.png)

![新建规则](/images/windows/server/2016/ftp/15.png)

![规则类型](/images/windows/server/2016/ftp/16.png)

![指定端口](/images/windows/server/2016/ftp/17.png)

![指定操作](/images/windows/server/2016/ftp/18.png)

![配置文件](/images/windows/server/2016/ftp/19.png)

![指定名称](/images/windows/server/2016/ftp/20.png)



### 登录验证

在windows资源管理器中输入以下地址即可登陆：

```
ftp://ip地址:端口
```

或者

```
ftp://username:password@ip地址:端口
```

![登录验证](/images/windows/server/2016/ftp/21.png)



### 参考

[Windows Server 2016 搭建 FTP环境](https://blog.csdn.net/h8178/article/details/78349882)

