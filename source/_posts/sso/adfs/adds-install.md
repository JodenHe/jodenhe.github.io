---
title: ADDS 活动目录安装详解
author: Joden_He
tags: 
  - SSO
  - ADFS
  - ADDS
categories: 
  - SSO
  - ADFS
description: 本文演示如何使用 windows server 2012 R2  安装 ADDS 服务
---

### 安装前准备

确保域控制器 (DC) 的计算机名符合企业命名规范，因为 AD 安装完成后再更改 DC 计算机相对复杂而且存在风险。DC 需要使用固定的静态IP地址，并且把DNS设置指向本机IP。

这台电脑 (右键) --> 属性，更改设置：

![computer name](/images/sso/adfs/adds-install/cp-name.png)



网络 (右键) --> 属性 --> 以太网 --> 属性 --> Internet 协议版本 4（TCP/IPV4） --> 属性

![network](/images/sso/adfs/adds-install/network.png)



### 开始安装

- 打开服务器管理器，点击 “添加角色和功能”

![step 1](/images/sso/adfs/adds-install/step1.png)



- 保持默认，并连续点击三次“下一步”。

![step 2](/images/sso/adfs/adds-install/step2.png)



- 勾选 “ActiveDirectory域服务”，然后选择添加所需的功能，最后点“下一步”继续。

![step 3](/images/sso/adfs/adds-install/step3.png)



- 功能选择页面保持默认，连续点击2次“下一步”。

![step 4](/images/sso/adfs/adds-install/step4.png)



- 勾选自动重启服务器，点击“安装”。

![step 5.1](/images/sso/adfs/adds-install/step5.1.png)



![step 5.2](/images/sso/adfs/adds-install/step5.2.png)



- 完成ADDS安装后，如图点击提示图标后，选择“将此服务器提升为域控制器”。

![step 6](/images/sso/adfs/adds-install/step6.png)



- 由于这是第一台域控制器(DC)，也叫林根域。选择“添加新林”，然后输入根域名。

![step 7](/images/sso/adfs/adds-install/step7.png)



- 选择林和域功能级别，勾选DNS(AD需要高度集成DNS)，并输入目录还原模式密码。

![step 8](/images/sso/adfs/adds-install/step8.png)



- 由于现在还没有DNS服务器，所以会提示无法创建委派。直接“下一步”。

![step 9](/images/sso/adfs/adds-install/step9.png)



- NetBIOS名以及ADDS数据库、日志、Sysvol保存位置，如果企业没有特殊要求，一般保持默认即可。

![step 10.1](/images/sso/adfs/adds-install/step10.1.png)



![step 10.2](/images/sso/adfs/adds-install/step10.2.png)



- 检查配置无误后点击“下一步”，然后点击“安装”，等待安装完成后重启即可。

![step 11.1](/images/sso/adfs/adds-install/step11.1.png)



![step 11.2](/images/sso/adfs/adds-install/step11.2.png)



至此完成 `ActiveDirectory` 的全部安装。已经可以对外提供服务了。

### 参考

[Active Directory管理之一:活动目录安装详解](https://blog.51cto.com/labixiaoniu/1253454)