---
date: 2021-02-27 11:41:41
title: virtualbox 下 centos7 网络配置
author: Joden_He
tags: 
  - virtualbox
  - centos7
categories: 
  - virtualbox
  - centos7
description: 介绍 virtualbox 下 centos7 网络配置，使虚拟机内网可互相联通
---

### 安装 centos7 虚拟机

- 选中工具--点击新建--填写虚拟机名称--类型选择Linux--版本选Red Hat (64-bit)--下一步

  ![新建虚拟机](/images/virtualbox/centos7/20210227114928.png)

  

- 设置内存大小（可以按默认来，也可以选择合适的内存大小，默认为1G），下一步

  ![设置内存大小](/images/virtualbox/centos7/20210227115820.png)

  
  
- 创建虚拟硬盘，创建

  ![创建虚拟硬盘](/images/virtualbox/centos7/20210227120035.png)

  

- 选择虚拟硬盘文件类型，再点击下一步

  ![虚拟硬盘文件类型](/images/virtualbox/centos7/20210227121221.png)

  

- 这里选择硬盘容量动态分配，下一步

  ![动态分配硬盘容量](/images/virtualbox/centos7/20210227121430.png)

  

- 选择合适的文件存储位置和大小，点击创建

  ![文件位置和大小](/images/virtualbox/centos7/20210227121637.png)

  

### 设置虚拟机

- 选中刚刚创建的虚拟机，点击设置

  ![设置](/images/virtualbox/centos7/20210227122158.png)

- 设置安装镜像（去官网下载centos7的安装镜像即可，这里不进行赘述）

  ![选择安装镜像](/images/virtualbox/centos7/20210227122527.png)

  > 可以在阿里站点下载，速度较快
  >
  > http://mirrors.aliyun.com/centos/7/isos/x86_64/
  >
  > 选择CentOS-7-x86_64-Minimal-2009.iso即可，或者其他的

  

  ![centos7镜像](/images/virtualbox/centos7/20210227122813.png)

- 配置网络驱动（本文关键）

  - 网卡1选择默认 NAT 连接方式，并记下 MAC 地址（后续会用到）

    ![网卡1](/images/virtualbox/centos7/20210227123532.png)

  - 网卡2启用网络连接--选择 "仅主机(Host-Only)网络" 连接方式，记下 MAC 地址（后续会用到）

    ![网卡2](/images/virtualbox/centos7/20210227123803.png)

    

- 点击 ok 进行虚拟机启动安装即可，安装过程和普通的系统安装一样，按着提示一步步来就行

  - 启动进来会提示你选择启动盘，选择刚刚配置的安装镜像即可

    ![](/images/virtualbox/centos7/20210227124207.png)

  - 其余步骤不再展开

### 设置联网（本文关键）

> 参考：https://www.jianshu.com/p/044fc0b85521
>
> 关键操作：
>
> 进入cd /etc/sysconfig/network-scripts/目录，可以看到目前只有ifcfg-enp0s3和ifcfg-enp0s8配置文件（如果安装centos7没有选择两个网卡的话，应该只有ifcfg-enp0s3 一个配置文件,如果只有一个配置文件，则另外一个就用这个copy过来改，需要更改UUID的值）

![配置网络](/images/virtualbox/centos7/20210227130511.png)

- 编辑 ifcfg-enp0s3 设置网卡1，设置动态 ip 进行联网，使得能服务器访问外网

  ```sh
  # vi ifcfg-enp0s3
  ```

  首行添加网卡1的 MAC 地址

  ![网卡1配置](/images/virtualbox/centos7/20210227131309.png)

- 编辑 ifcfg-enp0s8 设置网卡2，固定 ip，后续 ssh 通过此 ip 连接

  ```sh
  # vi ifcfg-enp0s8
  ```

  首行添加网卡2的 MAC 地址，设置静态ip

  ![网卡2配置](/images/virtualbox/centos7/20210227131852.png)

- 重启网络

  ```sh
  # service network restart
  ```

  ![重启网络](/images/virtualbox/centos7/20210227132225.png)

- 测试验证

  ```sh
  # ping baidu.com
  ```

  这时候要是可以上网，那设置ok，如何还是不行就继续下一步操作

- 先把虚拟机关闭，重新设置网卡1的连接方式

  - 管理--全局设定

    ![全局设定](/images/virtualbox/centos7/20210227133343.png)

  - 网络--添加 NAT 网络--OK

    ![添加NAT网络](/images/virtualbox/centos7/20210227133513.png)

- 选中虚拟机，点击设置，进行网卡1连接方式配置

  选择 `NAT 网络` 并选择 刚刚创建的 NAT 网络 `NatNetwork` 点击 OK

  ![网卡1连接方式修改](/images/virtualbox/centos7/20210227133824.png)

- 重新启动虚拟机再进行验证即可

- 假如还是无法 ping 通 baidu.com，尝试 ping 百度的 ip （可以通过一个可以上网的电脑 ping 域名获取 ip）

  ip 能通但是域名无法解析，需要重新编辑 ifcfg-enp0s3 在末尾添加 DNS服务器

  ```text
  DNS1=8.8.8.8
  DNS2=114.114.114.114
  ```

  ![添加dns解析服务器](/images/virtualbox/centos7/20210227142526.png)

- 再进行测试

  ![ping](/images/virtualbox/centos7/20210227142750.png)

  