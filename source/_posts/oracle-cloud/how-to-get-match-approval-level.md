---
date: 2019-02-20 00:00:00
title: 如何获取 oracle cloud 的 Match Approval Level
author: Joden_He
tags: 
  - Oracle Cloud
  - Match Approval Level
categories: 
  - Oracle Cloud
description: oracle cloud 中后台表是通过 receipt_required_flag inspection_required_flag 两个字段的关系获取 Match Approval Level 的值，po, supplier, supplier site 获取的原理同样
---



### 版本信息

**Oracle Cloud Application**

**Revision 13.18.10 (11.13.18.10.0)**

| Title                                         | Description                                                  |
| --------------------------------------------- | ------------------------------------------------------------ |
| Oracle Application Development Framework      | JDEVADF_11.1.1.9.2ADF-REL131810-PROD-BP_GENERIC_181116.1437.7206 |
| Oracle Middleware Extensions for Applications | ATGPF_PT.REL13.18.10_GENERIC_181113.1603.REL13BP16.7         |
| Database Compatibility                        | TRUE (REL13BP16.7)                                           |



### 提示

- When Receipt Required Flag is N, then Match Approval level is 2 way.

- When Receipt Required Flag is Y and Inspection Required Flag is N, then Match Approval Level is 3 way.

- When Receipt Required Flag is Y and Inspection Required Flag is Y, then Match Approval Level is 4 way.

> 2-way 不需要检测收货 receipt_required_flag 为 N
>
> 3-way 需要在 match invoice 的时候验收货 所以 receipt_required_flag 为Y , inspection_required_flag 为 N
>
> 4-way 还要检测验货 所以 两个都为 Y

### SQL

```sql
decode(poz_supplier_sites_v.receipt_required_flag,
       'Y',
       decode(poz_supplier_sites_v.inspection_required_flag,
              'Y',
              '4-Way',
              'N',
              '3-Way'),
       'N',
       '2-Way') AS matching_level
```

### 参考

[http://j178.mtgbb.com/?p=713](http://j178.mtgbb.com/?p=713)



