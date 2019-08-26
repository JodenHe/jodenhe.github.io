---
title: JSTACK 分析线程
author: Joden_He
tags: 
  - Java
  - Jstack
categories: 
  - Java
description: CPU 占用100%，如何根据 java 线程情况定位到问题所在
---



### 查找 tomcat 线程

可以通过两种方式找：

- jps

  一般Bootstrap都是tomcat的

  ![JPS](/images/1.png)

  

- ps -ef |grep tomcat

  ![PS](/images/2.png)

  

### jstack dump 出线程

```shell
jstack 23524 > jstack.txt
```

### 查看 tomcat 内部线程

```shell
top -H -p 23524
```

> （shift+p 按 cpu 排序，shift+m 按内存排序） 

![tomcat-stack](/images/3.png)



### 将PID换算成16进制，然后在jstack.txt查找nid=0x换算后的数

```shell
cat jstack.txt | grep 'nid=0x5be9' -B 5 -A 40
```

![4.png](/images/4.png)

> 发现频繁的GC，推测是不是数据库在频繁的开关导致，因此检查 tomcat jndi是否没配置数据库连接池参数

### 加上连接池参数

![5.png](/images/5.png)



### 参考

jstack分析cpu负载过高原因：

https://blog.csdn.net/u010248330/article/details/80080605

jstack: Java占用高CPU分析之- GC Task Thread:

https://blog.csdn.net/chenxiusheng/article/details/74007040