---
date: 2019-02-20 13:00:00
title: oracle cloud 如何获取图片 url
author: Joden_He
tags: 
  - Oracle Cloud
  - attachment
categories: 
  - Oracle Cloud
description: oracle cloud 中通过附件来管理图片信息，其中图片信息可以是外链 url, 也可以是上传图片信息，cloud 生成链接信息。
---





以获取物料图片 `url` 作为例子

```sql
(SELECT (decode(datatype_code,
                'FILE',
                (:p_host || '/cs/idcplg?IdcService=GET_FILE&dID=') || dm_version_number,
                'WEB_PAGE',
                url)) AS image
 FROM fnd_attached_documents d,
 fnd_documents_vl       dv
 WHERE d.document_id = d.document_id
 AND d.entity_name = 'ITEM_ENTITY'
 AND d.pk1_value = to_char(esi.organization_id)
 AND d.pk2_value = to_char(esi.inventory_item_id)
 AND rownum = 1) AS item_photo
```

>  其中 `p_host` 为 `cloud` 的主域名

其他类型的图片 `url` 可以通过变换 `d.entity_name = 'ITEM_ENTITY'` 进行查找