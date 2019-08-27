---
date: 2019-08-24 00:00:00
title: 如何使用 webservice 备份报表 catalog 对象
author: Joden_He
tags: 
  - Oracle Cloud
  - catalog
categories: 
  - Oracle Cloud
  - webservice
description: 本文介绍使用 oracle cloud webservice 备份报表 catalog 对象
---

## 手动备份

如果不是经常需要进行备份可以通过手动进行备份，没必要通过 `webservice` 备份，可以使用以下两种方式手动操作

- BIP: [Downloading and Uploading Catalog Objects](http://docs.oracle.com/cd/E28280_01/bi.1111/e22257/manage_obj_bi_pub_cat.htm#BIPUG261)
- OBIEE: [Archiving Objects](http://docs.oracle.com/cd/E28280_01/bi.1111/e10544/mancat.htm#BIEUG321)

> ps: 进入`BI Publisher` `https://xxx.com/xmlpserver`

## webservice 备份

其实只是用 webservice 的方式进行 BIP 备份的两步操作

主要使用 webservice: `https://xxx.com/xmlpserver/services/v2/CatalogService?wsdl ` 中的 `downloadObject` 和 `uploadObject` 或者 `updateObject`

通常情况下，我们只需要调用 `downloadObject`, 获取 `catalog objects` 然后手动通过 `BIP` 界面上传还原即可完成整个备份还原流程了。简单介绍一下流程

### 请求报文

![payload](/images/oracle_cloud/20190824172652.png)

```xml
<Envelope xmlns="http://schemas.xmlsoap.org/soap/envelope/">
    <Body>
        <downloadObject xmlns="http://xmlns.oracle.com/oxp/service/v2">
            <reportAbsolutePath>/Custom/Financials/General Ledger</reportAbsolutePath>
            <userID>****</userID>
            <password>***</password>
        </downloadObject>
    </Body>
</Envelope>
```

- reportAbsolutePath: 报表路径，假设我们要备份共享目录客制化的 `/Custom/Financials/General Ledger` 目录，填写 `/Custom/Financials/General Ledger`
- userID：用户名
- password: 密码

### 响应报文

![response](/images/oracle_cloud/20190824163707.png)

通过获取的 `base64binary Code` 解密生成文件就完成了文件备份的步骤了

![base64 to file](/images/oracle_cloud/20190824172457.png)

> 测试的话可以通过 `https://www.hitoy.org/tool/file_base64.php` 在线对 `base64binary Code` 转换成文件，这里下载的是文件夹因此后缀名写 `xdrz`, 后缀名对应请往下看

### 文件后缀名确定

`catalog` 对象文件后缀名的对应如下，在转文件时需要使用正确的后缀名，不然 BIP 上传还原报错

| Catalog Object | Extension Assigned to Downloaded Files |
| -------------- | -------------------------------------- |
| Data Model     | .xdmz                                  |
| Folder         | .xdrz                                  |
| Report         | .xdoz                                  |
| Style Template | .xssz                                  |
| Subtemplate    | .xsbz                                  |

### BIP 上传 `catalog` 对象进行还原

输入网址 `https://xxx.com/xmlpserver` 进入 BIP 界面

进入 `catalog`  管理界面，下图两个框框位置都可以进入

![catalog manage](/images/oracle_cloud/20190824170436.png)

进入要还原到的目录下进行以下操作

![upload catalog](/images/oracle_cloud/20190824172916.png)

还原成功

![success](/images/oracle_cloud/20190824173101.png)

## 测试结果分析

后期测试发现，下载的文件夹，如果里面的内容过多（猜测是文件夹过多）会导致upload的时候失败，原因未明，另外发现通过下载上传的方式无法保留原文件的权限等设置。因此，感觉如非必要还是通过 `OBIEE` 方式进行备份吧（也就是归档与反归档）

