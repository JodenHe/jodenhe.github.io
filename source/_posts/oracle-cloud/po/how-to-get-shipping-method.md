---
title: 如何获取 oracle cloud 的 PO 的 shipping method
author: Joden_He
tags: 
  - Oracle Cloud
  - PO
categories: 
  - Oracle Cloud
  - PO
description: Oracle Cloud PO 界面上显示的 shipping method 在后台是通过几个字段来存储的，要获取该字段得使用函数获取。
---



### 版本信息

> Oracle Cloud Application 19B (11.13.19.04.0)

### example

- 获取 po 头上的 shipping method

![po head shipping method](/images/oracle_cloud/20190826222844.png)

```sql
SELECT po_bip_helper.get_shipping_method(carrierpartyname.party_name,
                                         modeoftransportlookuppeo.meaning,
                                         servicelevellookuppeo.meaning) AS shippingmethod
  FROM wsh_carriers   carrier,
       hz_parties     carrierpartyname,
       fnd_lookups    modeoftransportlookuppeo,
       fnd_lookups    servicelevellookuppeo,
       po_headers_all header
 WHERE (header.mode_of_transport = modeoftransportlookuppeo.lookup_code(+) AND modeoftransportlookuppeo.lookup_type(+) = 'WSH_MODE_OF_TRANSPORT')
   AND (header.service_level = servicelevellookuppeo.lookup_code(+) AND servicelevellookuppeo.lookup_type(+) = 'WSH_SERVICE_LEVELS')
   AND header.carrier_id = carrier.carrier_id(+)
   AND carrier.carrier_id = carrierpartyname.party_id(+)
   AND header.segment1 = '20000028'
```



- 获取 po 行上的 shipping method

![po line shipping method](/images/oracle_cloud/20190826223054.png)

```sql
SELECT po_bip_helper.get_shipping_method(carrierpartyname.party_name,
                                         modeoftransportlookuppeo.meaning,
                                         servicelevellookuppeo.meaning) AS shippingmethod
  FROM wsh_carriers          carrier,
       hz_parties            carrierpartyname,
       fnd_lookups           modeoftransportlookuppeo,
       fnd_lookups           servicelevellookuppeo,
       po_line_locations_all plla
 WHERE (plla.mode_of_transport = modeoftransportlookuppeo.lookup_code(+) AND modeoftransportlookuppeo.lookup_type(+) = 'WSH_MODE_OF_TRANSPORT')
   AND (plla.service_level = servicelevellookuppeo.lookup_code(+) AND servicelevellookuppeo.lookup_type(+) = 'WSH_SERVICE_LEVELS')
   AND plla.carrier_id = carrier.carrier_id(+)
   AND carrier.carrier_id = carrierpartyname.party_id(+)
   AND plla.po_header_id = pla.po_header_id
   AND plla.po_line_id = pla.po_line_id

```

