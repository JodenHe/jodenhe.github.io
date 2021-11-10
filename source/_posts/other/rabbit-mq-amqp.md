---
title: AMQP 0-9-1 常用命令
date: 2021-11-11 00:14:32
author: Joden_He
tags:
  - RabbitMQ
categories: 
  - RabbitMQ
description: "AMQP 0-9-1 中常用的命令与 RabbitMQ 客户端中方法的映射关系"
---

## AMQP 0-9-1 中常用的命令与 RabbitMQ 客户端中方法的映射关系

| 名称                | 是否包含内容体 | 对应客户端的方法        | 简要描述                         |
| ------------------- | -------------- | ----------------------- | -------------------------------- |
| Connection.Start    | 否             | factory.newConnection   | 建立连接相关                     |
| Connection.Start-OK | 否             | factory.newConnection   | 建立连接相关                     |
| Connection.Tune     | 否             | factory.newConnection   | 建立连接相关                     |
| Connection.Tune-OK  | 否             | factory.newConnection   | 建立连接相关                     |
| Connection.Open     | 否             | factory.newConnection   | 建立连接相关                     |
| Connection.Open-OK  | 否             | factory.newConnection   | 建立连接相关                     |
| Connection.Close    | 否             | connection.close        | 关闭连接                         |
| Connection.Close-OK | 否             | connection.close        | 关闭连接                         |
| Channel.Open        | 否             | connection.openChannel  | 开启信道                         |
| Channel.Open-OK     | 否             | connection.openChannel  | 开启信道                         |
| Channel.Close       | 否             | channel.close           | 关闭信道                         |
| Channel.Close-OK    | 否             | channel.close           | 关闭信道                         |
| Exchange.Declare    | 否             | channel.exchangeDeclare | 声明交换器                       |
| Exchange.Declare-OK | 否             | channel.exchangeDeclare | 声明交换器                       |
| Exchange.Delete     | 否             | channel.exchangeDelete  | 删除交换器                       |
| Exchange.Delete-OK  | 否             | channel.exchangeDelete  | 删除交换器                       |
| Exchange.Bind       | 否             | channel.exchangeBind    | 交换器与交换器绑定               |
| Exchange.Bind-OK    | 否             | channel.exchangeBind    | 交换器与交换器绑定               |
| Exchange.Unbind     | 否             | channel.exchangeUnbind  | 交换器与交换器解绑               |
| Exchange.Unbind-OK  | 否             | channel.exchangeUnbind  | 交换器与交换器解绑               |
| Queue.Declare       | 否             | channel.queueDeclare    | 声明队列                         |
| Queue.Declare-OK    | 否             | channel.queueDeclare    | 声明队列                         |
| Queue.Bind          | 否             | channel.queueBind       | 队列与交换器绑定                 |
| Queue.Bind-OK       | 否             | channel.queueBind       | 队列与交换器绑定                 |
| Queue.Purge         | 否             | channel.queuePurge      | 清除队列中的内容                 |
| Queue.Purge-OK      | 否             | channel.queuePurge      | 清除队列中的内容                 |
| Queue.Delete        | 否             | channel.queueDelete     | 删除队列                         |
| Queue.Delete-OK     | 否             | channel.queueDelete     | 删除队列                         |
| Queue.Unbind        | 否             | channel.queueUnbind     | 队列与交换器解绑                 |
| Queue.Unbind-OK     | 否             | channel.queueUnbind     | 队列与交换器解绑                 |
| Basic.Qos           | 否             | channel.basicQos        | 设置未被确认消费的个数           |
| Basic.Qos-OK        | 否             | channel.basicQos        | 设置未被确认消费的个数           |
| Basic.Consume       | 否             | channel.basicConsume    | 消费消息（推模式）               |
| Basic.Consume-OK    | 否             | channel.basicConsume    | 消费消息（推模式）               |
| Basic.Cancel        | 否             | channel.basicCancel     | 取消                             |
| Basic.Cancel-OK     | 否             | channel.basicCancel     | 取消                             |
| Basic.Publish       | 是             | channel.basicPublish    | 发送消息                         |
| Basic.Return        | 是             | 无                      | 未能成功路由的消息返回           |
| Basic.Deliver       | 是             | 无                      | Broker 推送消息返回              |
| Basic.Get           | 否             | channel.basicGet        | 消费消息（拉模式）               |
| Basic.Get-OK        | 是             | channel.basicGet        | 消费消息（拉模式）               |
| Basic.Ack           | 否             | channel.basicAck        | 确认                             |
| Basic.Reject        | 否             | channel.basicReject     | 拒绝（单条拒绝）                 |
| Basic.Recover       | 否             | channel.basicRecover    | 请求Broker重新发送未被确认的消息 |
| Basic.Recover-OK    | 否             | channel.basicRecover    | 请求Broker重新发送未被确认的消息 |
| Basic.Nack          | 否             | channel.basicNack       | 拒绝（可批量拒绝）               |
| Tx.Select           | 否             | channel.txSelect        | 开启事务                         |
| Tx.Select-OK        | 否             | channel.txSelect        | 开启事务                         |
| Tx.Commit           | 否             | channel.txCommit        | 事务提交                         |
| Tx.Commit-OK        | 否             | channel.txCommit        | 事务提交                         |
| Tx.Rollback         | 否             | channel.txRollback      | 事务回滚                         |
| Tx.Rollback-OK      | 否             | channel.txRollback      | 事务回滚                         |
| Confirm.Select      | 否             | channel.confirmSelect   | 开启发送端确认模式               |
| Confirm.Select-OK   | 否             | channel.confirmSelect   | 开启发送端确认模式               |

