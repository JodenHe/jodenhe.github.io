---
title: Rabbit MQ 集群搭建
date: 2021-11-18 00:48:32
author: Joden_He
tags:
  - RabbitMQ
categories: 
  - RabbitMQ
description: "Rabbit MQ 集群搭建"
---

## 前言

> rabbitMQ 集群对延迟非常敏感，应当只在本地**局域网**内使用。在广域网中不应该使用集群，而应该使用 Federation 或者 Shovel 来代替。

假设有三个物理机

```tex
192.168.56.201 node1
192.168.56.202 node2
192.168.56.203 node3
```

> 每个节点上都需要安装 rabbitMQ，这里不再赘述如何安装，请参考：[rabbitMQ 安装](/2021/10/09/other/rabbit-mq-install/)

## 节点互相识别

在每个节点服务器上配置 ip 地址与节点名称的映射信息，编辑 `/etc/hosts` 文件，添加以下内容

```tex
192.168.56.201 node1
192.168.56.202 node2
192.168.56.203 node3
```

## 设置密钥令牌

编辑 rabbitMQ 的 cookie 文件，确保每个节点的 cookie 文件使用的是同一个值。可以取 node1 节点的 cookie 值，然后复制到 node2 和 node3 节点中。cookie 文件默认路径为 `/var/lib/rabbitmq/.erlang.cookie` 或者 `$HOME/.erlang.cookie`

cookie 相当于密钥令牌，集群中的 rabbitMQ 节点需要通过交换密钥令牌以获得相互认证。如果节点的密钥令牌不一致，在配置节点时会报以下错误。

```bash
[joden@node3 ~]$ rabbitmqctl join_cluster rabbit@node1
Clustering node rabbit@node3 with rabbit@node1
Error: unable to perform an operation on node 'rabbit@node1'. Please see diagnostics information and suggestions below.

Most common reasons for this are:

 * Target node is unreachable (e.g. due to hostname resolution, TCP connection or firewall issues)
 * CLI tool fails to authenticate with the server (e.g. due to CLI tool's Erlang cookie not matching that of the server)
 * Target node is not running

In addition to the diagnostics info below:

 * See the CLI, clustering and networking guides on https://rabbitmq.com/documentation.html to learn more
 * Consult server logs on node rabbit@node1
 * If target node is configured to use long node names, don't forget to use --longnames with CLI tools

DIAGNOSTICS
===========

attempted to contact: [rabbit@node1]

rabbit@node1:
  * connected to epmd (port 4369) on node1
  * epmd reports node 'rabbit' uses port 25672 for inter-node and CLI tool traffic
  * TCP connection succeeded but Erlang distribution failed
  * suggestion: check if the Erlang cookie identical for all server nodes and CLI tools
  * suggestion: check if all server nodes and CLI tools use consistent hostnames when addressing each other
  * suggestion: check if inter-node connections may be configured to use TLS. If so, all nodes and CLI tools must do that
   * suggestion: see the CLI, clustering and networking guides on https://rabbitmq.com/documentation.html to learn more


Current node details:
 * node name: 'rabbitmqcli-54-rabbit@node3'
 * effective user's home directory: /home/joden
 * Erlang cookie hash: 7l9ElKhq8r9jSXhS6DMULA==

[joden@node3 ~]$
```



## 配置集群

配置集群有三种方式：

1. 通过 `rabbitmqctl` 工具配置
2. 通过 `rabbitmq.config` 配置文件配置
3. 通过 `rabbitmq-autocluster` 插件配置

这里只介绍 `rabbitmqctl` 工具配置，这是最常用的方式，其他两种方式在实际应有中很少使用。

### 启动三个节点的 RabbitMQ 服务

```bash
[joden@node1 ~]$ rabbitmq-server -detached
[joden@node2 ~]$ rabbitmq-server -detached
[joden@node3 ~]$ rabbitmq-server -detached
```

这3个节点目前都是以独立的节点存在的单个集群，可以通过 `rabbitmqctl cluster_status` 命令查看各个节点状态。

node1:

```bash
[joden@node1 ~]$ rabbitmqctl cluster_status
Cluster status of node rabbit@node1 ...
Basics

Cluster name: rabbit@node1

Disk Nodes

rabbit@node1

Running Nodes

rabbit@node1

Versions

rabbit@node1: RabbitMQ 3.9.8 on Erlang 24.1

Maintenance status

Node: rabbit@node1, status: not under maintenance

Alarms

(none)

Network Partitions

(none)

Listeners

Node: rabbit@node1, interface: [::], port: 15672, protocol: http, purpose: HTTP API
Node: rabbit@node1, interface: [::], port: 25672, protocol: clustering, purpose: inter-node and CLI tool communication
Node: rabbit@node1, interface: [::], port: 5672, protocol: amqp, purpose: AMQP 0-9-1 and AMQP 1.0

Feature flags

Flag: drop_unroutable_metric, state: enabled
Flag: empty_basic_get_metric, state: enabled
Flag: implicit_default_bindings, state: enabled
Flag: maintenance_mode_status, state: enabled
Flag: quorum_queue, state: enabled
Flag: stream_queue, state: enabled
Flag: user_limits, state: enabled
Flag: virtual_host_metadata, state: enabled
[joden@node1 ~]$
```

node2:

```bash
[joden@node2 ~]$ rabbitmqctl cluster_status
Cluster status of node rabbit@node2 ...
Basics

Cluster name: rabbit@node2

Disk Nodes

rabbit@node2

Running Nodes

rabbit@node2

Versions

rabbit@node2: RabbitMQ 3.9.8 on Erlang 24.1

Maintenance status

Node: rabbit@node2, status: not under maintenance

Alarms

(none)

Network Partitions

(none)

Listeners

Node: rabbit@node2, interface: [::], port: 15672, protocol: http, purpose: HTTP API
Node: rabbit@node2, interface: [::], port: 25672, protocol: clustering, purpose: inter-node and CLI tool communication
Node: rabbit@node2, interface: [::], port: 5672, protocol: amqp, purpose: AMQP 0-9-1 and AMQP 1.0

Feature flags

Flag: drop_unroutable_metric, state: enabled
Flag: empty_basic_get_metric, state: enabled
Flag: implicit_default_bindings, state: enabled
Flag: maintenance_mode_status, state: enabled
Flag: quorum_queue, state: enabled
Flag: stream_queue, state: enabled
Flag: user_limits, state: enabled
Flag: virtual_host_metadata, state: enabled
[joden@node2 ~]$
```

node3:

```bash
[joden@node3 ~]$ rabbitmqctl cluster_status
Cluster status of node rabbit@node3 ...
Basics

Cluster name: rabbit@node3

Disk Nodes

rabbit@node3

Running Nodes

rabbit@node3

Versions

rabbit@node3: RabbitMQ 3.9.8 on Erlang 24.1

Maintenance status

Node: rabbit@node3, status: not under maintenance

Alarms

(none)

Network Partitions

(none)

Listeners

Node: rabbit@node3, interface: [::], port: 15672, protocol: http, purpose: HTTP API
Node: rabbit@node3, interface: [::], port: 25672, protocol: clustering, purpose: inter-node and CLI tool communication
Node: rabbit@node3, interface: [::], port: 5672, protocol: amqp, purpose: AMQP 0-9-1 and AMQP 1.0

Feature flags

Flag: drop_unroutable_metric, state: enabled
Flag: empty_basic_get_metric, state: enabled
Flag: implicit_default_bindings, state: enabled
Flag: maintenance_mode_status, state: enabled
Flag: quorum_queue, state: enabled
Flag: stream_queue, state: enabled
Flag: user_limits, state: enabled
Flag: virtual_host_metadata, state: enabled
[joden@node3 ~]$
```

### 组成集群

> 以 node1 节点为基准，讲 node2 和 node3 节点加入到 node1 节点集群中。

以 node2 为例，仅需要执行以下步骤即可

- 停止

  ```bash
  [joden@node2 ~]$ rabbitmqctl stop_app
  Stopping rabbit application on node rabbit@node2 ...
  [joden@node2 ~]$
  ```

- 重置

  ```bash
  [joden@node2 ~]$ rabbitmqctl reset
  Resetting node rabbit@node2 ...
  [joden@node2 ~]$
  ```

- 加入 node1 节点集群

  ```bash
  [joden@node2 ~]$ rabbitmqctl join_cluster rabbit@node1
  Clustering node rabbit@node2 with rabbit@node1
  ```

- 启动

  ```bash
  [joden@node2 ~]$ rabbitmqctl start_app
  Starting node rabbit@node2 ...
  [joden@node2 ~]$
  ```

  

这时候在 node1 或者 node2 执行 `rabbitmqctl cluster_status`，都可以看到以下输出

```bash
[joden@node2 ~]$ rabbitmqctl cluster_status
Cluster status of node rabbit@node2 ...
Basics

Cluster name: rabbit@node2

Disk Nodes

rabbit@node1
rabbit@node2

Running Nodes

rabbit@node1
rabbit@node2

Versions

rabbit@node1: RabbitMQ 3.9.8 on Erlang 24.1
rabbit@node2: RabbitMQ 3.9.8 on Erlang 24.1

Maintenance status

Node: rabbit@node1, status: not under maintenance
Node: rabbit@node2, status: not under maintenance

Alarms

(none)

Network Partitions

(none)

Listeners

Node: rabbit@node1, interface: [::], port: 15672, protocol: http, purpose: HTTP API
Node: rabbit@node1, interface: [::], port: 25672, protocol: clustering, purpose: inter-node and CLI tool communication
Node: rabbit@node1, interface: [::], port: 5672, protocol: amqp, purpose: AMQP 0-9-1 and AMQP 1.0
Node: rabbit@node2, interface: [::], port: 25672, protocol: clustering, purpose: inter-node and CLI tool communication
Node: rabbit@node2, interface: [::], port: 15672, protocol: http, purpose: HTTP API
Node: rabbit@node2, interface: [::], port: 5672, protocol: amqp, purpose: AMQP 0-9-1 and AMQP 1.0

Feature flags

Flag: drop_unroutable_metric, state: enabled
Flag: empty_basic_get_metric, state: enabled
Flag: implicit_default_bindings, state: enabled
Flag: maintenance_mode_status, state: enabled
Flag: quorum_queue, state: enabled
Flag: stream_queue, state: enabled
Flag: user_limits, state: enabled
Flag: virtual_host_metadata, state: enabled
[joden@node2 ~]$
```

同样操作对 node3 执行即可。 

## 关闭节点

如果关闭了集群中所有节点，需要确保在启动的时候**最后关闭的那个节点是第一个启动的**。如果第一个启动的并不是最后关闭的节点，那么这个节点会等待最后那个节点启动，等待时间为30秒，如果没有等到，这个先启动的节点也会失败。

如果最后一个关闭的节点由于某些异常无法启动，可以通过 `rabbitmqctl forget_cluster_node` 命令来将这个节点踢出当前集群。如果所有节点由于非正常因素关闭（比如断电），那么集群中的节点都会认为还有其他节点在它后面关闭。这时需要调用 `rabbitmqctl force_boot` 命令来启动一个节点，之后集群才能正常启动。

