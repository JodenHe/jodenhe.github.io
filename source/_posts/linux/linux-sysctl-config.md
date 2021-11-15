---
title: linux 操作系统部分内核参数配置
date: 2021-11-16 01:00:32
author: Joden_He
tags:
  - Linux
categories: 
  - Linux
description: "在 /etc/sysctl.conf 文件(Linux 操作系统)中配置的部分内核参数"
---



## 一些可配置的内核选项

| 参数项                          | 描述                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| fs.file-max                     | 内核分配的最大文件句柄数。极限值和当前值可以通过 `/proc/sys/file/nr` 来查看。<br />例如：<br />[joden@localhost ~]$ cat /proc/sys/fs/file-nr<br/>1280    0       585816 |
| net.ipv4.ip_local_port_range    | 本地 IP 端口范围，定义为一对值。该范围必须为并发连接提供足够的条目 |
| net.ipv4.tcp_tw_reuse           | 当启用时，允许内核重用 TIME_WAIT 状态的套接字。当用在 NAT 时，此选项是很危险的 |
| net.ipv4.tcp_fin_timeout        | 降低此值到 5~10 可减少连接关闭的时间，之后会停留在 TIME_WAIT 状态，建议用在有大量并发连接的场景 |
| net.core.somaxconn              | 监听队列的大小（同一时间建立过程中有多少个连接）。默认为128，增大到 4096 或更高，可以支持入站连接的爆发，如 clients 集体重连 |
| net.ipv4.tcp_max_syn_backlog    | 尚未收到连接客户端确认的连接请求的最大数量。默认为128，最大值为65535。优化吞吐量时，4096 和 8192 是推荐的起始值 |
| net.ipv4.tcp_keepalive_*        | net.ipv4.tcp_keepalive_time, net.ipv4.tcp_keepalive_intvl 和 net.ipv4.tcp_keepalive_probes 用于配置 TCP 存活时间 |
| net.ipv4.conf.default.rp_filter | 启用反向地址过滤。如果系统不关心 IP 地址欺骗，那么就禁用它   |

