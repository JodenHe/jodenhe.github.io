---
date: 2019-03-11 00:00:00
title: ADFS 安装及配置
author: Joden_He
tags: 
  - SSO
  - ADFS
categories: 
  - SSO
  - ADFS
description: "本文介绍 Windows Server 2012R2 下安装接配置 ADFS。ADFS (Active Directory Federation Services) 活动目录联合服务将活动目录拓展到 Internet，当用户通过活动目录认证时，域控制器检查用户的证书。证明是合法用户后，用户就可以随意访问 Windows 网络的任何授权资源，而无需在每次访问不同服务器时重新认证。"

---

### 安装前准备

在安装 ADFS 之前，我们需要准备号 DC，参考 [ADDS 活动目录安装详解](https://blog.joden123.top/2019/03/09/sso/adfs/adds-install/)

![DC](/images/sso/adfs/adfs-install/dc.png)



### 安装配置证书服务

#### 在 DC 上安装 CA

- 打开服务器管理器，点击 “添加角色和功能”

![step 1](/images/sso/adfs/adfs-install/step1.png)



- 保持默认，并连续点击三次“下一步”

![step 2](/images/sso/adfs/adfs-install/step2.png)



- 勾选 “Active Directory 证书服务”，然后选择添加所需的功能，最后点“下一步”继续

![step 3](/images/sso/adfs/adfs-install/step3.png)



- 功能选择页面保持默认，连续点击2次“下一步”

![step 4](/images/sso/adfs/adfs-install/step4.png)



- 勾选 CA 相关服务, 一直点下一步，最后点击安装

![step 5.1](/images/sso/adfs/adfs-install/step5.png)



- 安装完成

![step 6](/images/sso/adfs/adfs-install/step6.png)



#### 配置证书服务

- CA 安装完成后，如图点击提示图标后，选择“配置目标服务器上的 Active Directory 证书服务”

![step 7](/images/sso/adfs/adfs-install/step7.png)



- 根据向导进行配置

![step 8](/images/sso/adfs/adfs-install/step8.png)



- 勾选所需要配置的服务

![step 9](/images/sso/adfs/adfs-install/step9.png)



- 选择企业根

![step 10.1](/images/sso/adfs/adfs-install/step10.1.png)



![step 10.2](/images/sso/adfs/adfs-install/step10.2.png)



- 创建新的私钥

![step 11.1](/images/sso/adfs/adfs-install/step11.1.png)



![step 11.2](/images/sso/adfs/adfs-install/step11.2.png)



- 指定 CA 名称

![step 12](/images/sso/adfs/adfs-install/step12.png)



- 指定有效期

![step 13](/images/sso/adfs/adfs-install/step13.png)



- 指定数据库位置

![step 14](/images/sso/adfs/adfs-install/step14.png)



- 确认配置信息

![step 15](/images/sso/adfs/adfs-install/step15.png)



- 配置完成

![step 16](/images/sso/adfs/adfs-install/step16.png)



- 验证 CA

![step 17.1](/images/sso/adfs/adfs-install/step17.1.png)



![step 17.2](/images/sso/adfs/adfs-install/step17.2.png)



以上环境准备好后就可以配置 ADFS 服务器了。在配置前我们需要给计算机或者指定的用户或者计算机授权证书颁发。

### 安装ADFS服务

在安装 ADFS 之前，我们需要创建和配置证书

#### 创建证书

账户准备好后，然后申请证书，其实有很常见的两种方法，第一种就是通过iis进行证书申请，第二种通过 mmc 控制台进行申请，这里使用IIS的申请方法。

- 打开IIS管理器，选择【服务器证书】 
  创建证书申请 

![step 23.1](/images/sso/adfs/adfs-install/step23.1.png)



![step 23.2](/images/sso/adfs/adfs-install/step23.2.png)



![step 23.3](/images/sso/adfs/adfs-install/step23.3.png)



![step 23.4](/images/sso/adfs/adfs-install/step23.4.png)



![step 23.5](/images/sso/adfs/adfs-install/step23.5.png)

保存到桌面上：cert.txt

#### 配置证书

- 服务器浏览器，输入 <http://127.0.0.1/certsrv/> 如下：

![step 24.1](/images/sso/adfs/adfs-install/step24.1.png)

- 选择申请证书——>高级证书申请 

![step 24.2](/images/sso/adfs/adfs-install/step24.2.png)

- 选择： 使用 base64 编码的 CMC 或 PKCS #10 文件提交 一个证书申请，或使用 base64 编码的 PKCS #7 文件续订证书申请

![step 24.3](/images/sso/adfs/adfs-install/step24.3.png)

- 打开cert.txt ，粘贴进去，证书模板选择 web服务器，提交 

![step 24.4](/images/sso/adfs/adfs-install/step24.4.png)

- 下载证书，保存到本地：

![step 24.5](/images/sso/adfs/adfs-install/step24.5.png)

- 回到iis管理器，选择完成证书申请

![step 24.6](/images/sso/adfs/adfs-install/step24.6.png)

![step 24.7](/images/sso/adfs/adfs-install/step24.7.png)

![step 24.8](/images/sso/adfs/adfs-install/step24.8.png)

- 将 default web site 的 https 绑定证书改为刚才完成的证书

  > 选择 default web site , 右键 编辑绑定，选择 https（没有则添加） ， 点击编辑，选择 ssl 证书:（ 这个证书在安装adfs时使用该证书） 

![step 25.2](/images/sso/adfs/adfs-install/step25.2.png)



#### 开始安装

- 打开服务器管理器，点击 “添加角色和功能”

![step 1](/images/sso/adfs/adfs-install/step1.png)



- 保持默认，并连续点击三次“下一步”

![step 2](/images/sso/adfs/adfs-install/step2.png)



- 勾选 “Active Directory Federation Services” 安装服务角色

![step 26.1](/images/sso/adfs/adfs-install/step26.1.png)



- 功能选择页面保持默认，连续点击2次“下一步”

![step 26.2](/images/sso/adfs/adfs-install/step26.2.png)



- 安装

![step 26.3](/images/sso/adfs/adfs-install/step26.3.png)



- 安装完成

![step 26.4](/images/sso/adfs/adfs-install/step26.4.png)



- ADFS 安装完成后，如图点击提示图标后，选择“在此服务器上配置联合身份验证服务”

![step 26.5](/images/sso/adfs/adfs-install/step26.5.png)



- 功能选择页面保持默认，连续点击2次“下一步”

![step 26.6](/images/sso/adfs/adfs-install/step26.6.png)



- 选择证书(不是前面创建的证书)，填写显示的名称

![step 26.7](/images/sso/adfs/adfs-install/step26.7.png)



- 选择 adfs 服务帐号

![step 26.8](/images/sso/adfs/adfs-install/step26.8.png)



- 指定配置数据库

![step 26.10](/images/sso/adfs/adfs-install/step26.10.png)



- 下一步

![step 26.11](/images/sso/adfs/adfs-install/step26.11.png)



- 配置

![step 26.12](/images/sso/adfs/adfs-install/step26.12.png)



- 配置完成

![step 26.13](/images/sso/adfs/adfs-install/step26.13.png)

至此 ADFS 安装配置完成

![step 27](/images/sso/adfs/adfs-install/step27.png)



### 参考

[Windows Server 2016 安装及配置 ADFS 4.0](https://blog.51cto.com/gaowenlong/1920117)

[Server2012 下 部署adfs IFD](https://blog.csdn.net/xu_guowei/article/details/47781727)