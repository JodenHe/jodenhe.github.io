---

date: 2021-06-06 17:33:46
title: k8s 的 liveness 与 readiness
author: Joden_He
tags: 
  - k8s
categories: 
  - k8s
description: 介绍 k8s 的 liveness 与 readiness 探针
---

## 说明

liveness 和 readiness 是 k8s 用于健康探测的探针。

他们支持的探测方式相同，主要包括：

1. http 探测
2. 执行命令探测
3. tcp 探测

## 区别

- liveness 主要用来确定何时重启容器

- readiness 主要用来确定容器是否已经就绪。

- 初始值不同。

  - liveness 的初始值为成功。这样防止在应用还没有成功启动前，就被误杀。如果在规定时间内还未成功启动，才将其设置为失败，从而触发容器重建。

  - readiness 的初始值为失败。这样防止应用还没有成功启动前就向应用进行流量的导入。如果在规定时间内启动成功，才将其设置为成功，从而将流量向应用导入。

- liveness 与 readiness 二者作用不能相互替代。

  - 只配置了 liveness，那么在容器启动，应用还没有成功就绪之前，这个时候 pod 是 ready 的（因为容器成功启动了）。那么流量就会被引入到容器的应用中，可能会导致请求失败。虽然在 liveness 检查失败后，重启容器，此时 pod 的 ready 的 condition 会变为 false。但是前面会有一些流量因为错误状态导入。
  - 只配置了 readiness 是无法触发容器重启的。

## 参考

[详解k8s中的liveness和readiness的原理和区别](https://www.cnblogs.com/xuxinkun/p/11785521.html)