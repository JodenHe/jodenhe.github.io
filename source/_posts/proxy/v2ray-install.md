---
title: 一键搭建 v2Ray 
date: 2022-02-20 20:13:10
author: Joden_He
tags: 
  - 科学上网
  - v2ray
categories:
  - 科学上网
  - v2ray
description: "一键搭建 v2Ray 梯子" 
---

## 概述

- 为什么使用 v2Ray

  由于笔者之前使用 ssr，但是每次搭建完第二天就会被封端口，进行了一波百度，发现采用 v2Ray 会相对稳定，没那么容易被墙

- 相关概念

  https://toutyrater.github.io/

## 开干

<font color="red"><b>该脚本只适用于 centos7 服务器</b></font>

### 申请免费域名

> 这里使用 [freenom.com](http://freenom.com/) 进行域名申请
>
> <font color="red"><b>记得使用 firefox 浏览器</b></font>

- 先注册帐号

  这里笔者采用的是谷歌帐号登录（需要翻墙才可以，可以随便网上找个免费的vpn软件连接登录一下）

  ![登录 freenom](/images/proxy/v2ray/20220220210108.png)

- 完善信息（不然后面无法完成免费域名订单）

  ![Edit Account Details](/images/proxy/v2ray/20220220212043.png)

  填入必要的信息

  ![填入必要信息](/images/proxy/v2ray/20220220212257.png)

- Services --> Register a New Domain, 注册新域名

  ![Register a New Domain](/images/proxy/v2ray/20220220210615.png)

- 输入你想要的域名，并且点击 `Check Availability`

  tk, ml, ga, cf, gq 都是是免费的后缀，可以按自己喜好选择，比如说 sindy123.tk, 如果检测出需要收费，那可以重新输入直到合适

  ![Check Availability](/images/proxy/v2ray/20220220211024.png)

  ![the result of Check Availability](/images/proxy/v2ray/20220220211441.png)

- 点击 Checkout 

  ![Checkout](/images/proxy/v2ray/20220220212914.png)

- 选择12个月免费，并点击 Continue

  ![12 months and click continue](/images/proxy/v2ray/20220220213136.png)

- 完成订单

  ![Complete Order](/images/proxy/v2ray/20220220213700.png)

- 很遗憾，无法完成订单

  ![无法完成订单](/images/proxy/v2ray/20220220213857.png)

  > 别着急，网上百度了一下解决方法，https://baijiahao.baidu.com/s?id=1689185764130254565&wfr=spider&for=pc
  >
  > 貌似是校验ip导致的问题，需要使用 Firefox 的代理插件（这就是前面说需要使用 Firefox 的原因了），步骤如下

- Firefox 浏览器安装 Gooreplacer 插件

  ![打开扩展](/images/proxy/v2ray/20220220214314.png)

  点击 “扩展” 并输入`Gooreplacer` 回车

  ![搜索插件](/images/proxy/v2ray/20220220214445.png)

  点击并安装即可

  ![安装 Gooreplacer](/images/proxy/v2ray/20220220214642.png)

- 再重新来一遍注册域名的步骤即可

- 有时候其实已经注册成功了，但是还是显示失败，可以点击 Services --> My Domains 检查一下

  ![My Domains](/images/proxy/v2ray/20220220215532.png)

### DNS 解析到梯子服务器

> 这里使用 cloudflare

- 管理域名使用自定义的 Nameservers

  ![管理域名](/images/proxy/v2ray/20220220215916.png)

  ![manage nameserver](/images/proxy/v2ray/20220220220059.png)

  使用 cloudflare 的nameservers，`DAVE.NS.CLOUDFLARE.COM`, `GINA.NS.CLOUDFLARE.COM`

  ![使用 cloudflare 的 nameservers](/images/proxy/v2ray/20220220220617.png)

- 登入 cloudflare，添加站点

  ![add site](/images/proxy/v2ray/20220220221026.png)

- 选择免费的 plan，如果想更高的配置可以考虑收费的

  ![free plan](/images/proxy/v2ray/20220220221230.png)

- 添加 DNS 解析记录，解析到梯子服务器

  ![add dns record](/images/proxy/v2ray/20220220221512.png)

  ![dns record](/images/proxy/v2ray/20220220221632.png)

  > 如果想使用二级域名，那就把 `@` 改成二级域名前缀即可

- 校验 dns 解析是否生效，一般半小时内生效

  > 如果发现 ping 出的地址不对，那是因为上一步添加代理把 `proxy status` 开启了，不影响后续操作，你也可以关闭，生效后会直接显示你配置的地址

  ![ping test](/images/proxy/v2ray/20220220222111.png)

### 一键搭建

- 在梯子服务器，执行一键搭建脚本

  ```bash
  bash <(curl -sL https://gist.githubusercontent.com/JodenHe/815dd91277b722d36a860d39c2296083/raw/7f2b5ac0f8137b245d44741fc4a9f40cffa36755/v2Ray-install.sh)
  ```

  ![run v2ray-install.sh](/images/proxy/v2ray/20220220222717.png)

- 这里选择 `4.   安装V2ray-VMESS+WS+TLS(推荐)`，输入 4 回车

  ![choose a mode](/images/proxy/v2ray/20220220223045.png)

- 这里提示需要一个域名，域名的dns解析到本服务器（前面步骤已经做了），直接输入 y 进入安装

- 输入前面准备的域名：sindy123.tk，回车

  > 因为脚本里面的dns校验已经被去除了，所以请自行保证dns解析的正确

  ![输入伪装域名](/images/proxy/v2ray/20220220223321.png)

- 输入 v2Ray 的端口，回车，这里使用 39554

  ![v2ray 端口](/images/proxy/v2ray/20220220223551.png)

- 其他一路回车即可，也可以按提示输入其他

  ![other options](/images/proxy/v2ray/20220220223746.png)

- 最终输出 v2Ray 配置信息如下

  ![v2ray config](/images/proxy/v2ray/20220220232318.png)

  > 如果显示失败，那就看一下报错是怎样的，百度一下应该可以解决，![img](/images/proxy/v2ray/qqpyimg1645370688.gif)

- 最后最后，记得在你的服务器开放 v2Ray 的端口，这里设置的是：39554，否则无法连接通

### v2Ray 客户端

参考：https://tlanyan.pp.ua/v2ray-clients-download/

笔者使用的windows客户端为：`V2rayN`, 安卓客户端为：`V2rayNG`

## 参考

https://toutyrater.github.io/

https://tlanyan.pp.ua/v2ray-clients-download/

https://baijiahao.baidu.com/s?id=1689185764130254565&wfr=spider&for=pc
