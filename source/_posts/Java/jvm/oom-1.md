---
title: 初识 OOM
author: Joden_He
tags: 
  - Java
  - jvm
categories: 
  - Java
  - jvm
description: 在进行 java 编程的时候，难免会遇到 java.lang.OutOfMemoryError (简称 OOM)，也就是程序内存不够用，这里让我们简单的了解一下 OOM。
---

### 什么是 OOM

OOM，Out of Memory，也就是超出了预设内存。

> java.lang.OutOfMemoryError，官方说明： Thrown when the Java Virtual Machine cannot allocate an object because it is out of memory, and no more memory could be made available by the garbage collector. 意思就是说，当 JVM 没有足够的内存来为对象分配空间，并且垃圾回收器也已经没有空间可回收时，就会抛出这个error（注：非exception，因为这个问题已经严重到不足以被应用处理）。



### 为什么会 OOM

主要原因有两点

1. 分配内存不足，可用通过调整启动 VM 参数控制。
2. 程序逻辑问题，占用过多内存并且使用完没有及时释放，造成内存泄露或者内存溢出。

其中：

- 内存泄漏：申请使用完的内存没有释放，导致虚拟机不能再次使用该内存，此时这段内存就泄露了，因为申请者不用了，而又不能被虚拟机分配给别人用。（往往是编码不当导致）

- 内存溢出：申请的内存超出了JVM能提供的内存大小，此时称之为溢出。



### JVM 内存模型

上面频繁提到的内存指的就是 JVM 内存模型。

JVM 在运行会管理以下的内存区域：

- 程序计数器：当前线程执行的字节码的<font color="red">行号指示器</font>，<font color="red">线程私有</font>，记录每个线程当前执行字节码的位置，因为会有多个线程来并发执行各种不同的代码，所有每个线程会有自己的一个程序计数器，专门记录这个线程目前执行到哪一条字节码指令，也就是线程私有；
- java 虚拟机栈：保存方法内局部变量数据的区域，线程私有。线程执行了一个方法，那么就会为这个方法调用创建对应的一个<font color="red">栈帧</font>。栈帧里有这个方法的局部变量表 、操作数栈、动态链接、方法出口等东西，调用执行任何方法的时候，都会给方法创建栈帧，然后入栈，调用完再出栈；
- 本地方法栈：类似“java 虚拟机栈”，但是为 native 方法的运行提供内存环境，调用本地操作系统里面的一些方法，可能是调用 c 语言写的底层类库提供的内存环境；
- java 堆内存：对象内存分配的地方，内存垃圾回收的主要区域，所有线程共享。对象按不同的 “生存” 情况分为新生代，老年代等；
- 方法区/Metaspace：JDK 1.8 后叫做 Metaspace，JDK 8 开始把类的元数据放到本地堆内存(native heap)中，这一块区域就叫 Metaspace，详情见[深入探究 JVM | 探秘 Metaspace](https://www.sczyh30.com/posts/Java/jvm-metaspace/)；
- 堆外内存：并不是 JVM 运行时数据区的一部分，可直接访问的内存，比如 NIO 会用到这部分；



### OOM 类型

- `java.lang.OutOfMemoryError: Java heap space`，比较常见，java 堆内存溢出。详情见[OutOfMemoryError系列（1）: Java heap space](https://blog.csdn.net/renfufei/article/details/76350794)
- `java.lang.OutOfMemoryError: GC overhead limit exceeded`，程序基本上耗尽了所有的可用内存, GC也清理不了。详情见[OutOfMemoryError系列（2）: GC overhead limit exceeded](https://blog.csdn.net/renfufei/article/details/77585294)
- `java.lang.OutOfMemoryError: PermGen space`，java 永久代溢出，也就是方法区溢出，jdk 7 之前。详情见[OutOfMemoryError系列（3）: Permgen space](https://blog.csdn.net/renfufei/article/details/77994177)
- `java.lang.OutOfMemoryError: Metaspace`，相当于 jdk 7以前的 `java.lang.OutOfMemoryError: PermGen space`。详情见[OutOfMemoryError系列（4）: Metaspace](https://blog.csdn.net/renfufei/article/details/78061354)
- `java.lang.StackOverflowError`，不会抛 OOM ERROR，但也是比较常见的 Java 内存溢出



### 查看应用进程id（pid）及启动 VM 参数

`jps -v` 查找应用进程 id 

```bash
jps -v

29591 Jps -Dapplication.home=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.191.b12-1.el7_6.x86_64 -Xms8m
9066 Bootstrap -Djava.util.logging.config.file=/xxx/tomcat/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Xms512m -Xmx2048m -Djdk.tls.ephemeralDHKeySize=2048 -Xms4096m -Xmx10240m -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/xxx/dump -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dorg.apache.catalina.security.SecurityListener.UMASK=0027 -Dignore.endorsed.dirs= -Dcatalina.base=/xxx/tomcat -Dcatalina.home=/xxx/lib/tomcat -Djava.io.tmpdir=/xxx/tomcat/temp
```

`jinfo -flags <pid>` 查看 VM 启动参数

```bash
jinfo -flags 9066

Attaching to process ID 9066, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.191-b12
Non-default VM flags: -XX:CICompilerCount=3 -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=null -XX:InitialHeapSize=4294967296 -XX:MaxHeapSize=10737418240 -XX:MaxNewSize=3578789888 -XX:MinHeapDeltaBytes=524288 -XX:NewSize=1431306240 -XX:OldSize=2863661056 -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseParallelGC
Command line:  -Djava.util.logging.config.file=/xxx/tomcat/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Xms512m -Xmx2048m -Djdk.tls.ephemeralDHKeySize=2048 -Xms4096m -Xmx10240m -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/xxx/dump -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dorg.apache.catalina.security.SecurityListener.UMASK=0027 -Dignore.endorsed.dirs= -Dcatalina.base=/xxx/tomcat -Dcatalina.home=/xxx/lib/tomcat -Djava.io.tmpdir=/xxx/tomcat/temp
```

`jmap -heap <pid>`   查看 VM 启动参数，以及堆内存情况

```bash
jmap -heap 9066

Attaching to process ID 9066, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.191-b12

using thread-local object allocation.
Parallel GC with 4 thread(s)

Heap Configuration:
   MinHeapFreeRatio         = 0
   MaxHeapFreeRatio         = 100
   MaxHeapSize              = 10737418240 (10240.0MB)
   NewSize                  = 1431306240 (1365.0MB)
   MaxNewSize               = 3578789888 (3413.0MB)
   OldSize                  = 2863661056 (2731.0MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
Exception in thread "main" java.lang.reflect.InvocationTargetException
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:498)
        at sun.tools.jmap.JMap.runTool(JMap.java:201)
        at sun.tools.jmap.JMap.main(JMap.java:130)
Caused by: java.lang.RuntimeException: unknown CollectedHeap type : class sun.jvm.hotspot.gc_interface.CollectedHeap
        at sun.jvm.hotspot.tools.HeapSummary.run(HeapSummary.java:157)
        at sun.jvm.hotspot.tools.Tool.startInternal(Tool.java:260)
        at sun.jvm.hotspot.tools.Tool.start(Tool.java:223)
        at sun.jvm.hotspot.tools.Tool.execute(Tool.java:118)
        at sun.jvm.hotspot.tools.HeapSummary.main(HeapSummary.java:50)
        ... 6 more
```

上面报错解决 [Exception when taking a heapdump using JMAP](https://stackoverflow.com/questions/20109503/exception-when-taking-a-heapdump-using-jmap)

解决再运行

```bash
jmap -heap 9066

Attaching to process ID 9066, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.191-b12

using thread-local object allocation.
Parallel GC with 4 thread(s)

Heap Configuration:
   MinHeapFreeRatio         = 0
   MaxHeapFreeRatio         = 100
   MaxHeapSize              = 10737418240 (10240.0MB)
   NewSize                  = 1431306240 (1365.0MB)
   MaxNewSize               = 3578789888 (3413.0MB)
   OldSize                  = 2863661056 (2731.0MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
PS Young Generation
Eden Space:
   capacity = 1209532416 (1153.5MB)
   used     = 279076032 (266.14764404296875MB)
   free     = 930456384 (887.3523559570312MB)
   23.073051065710338% used
From Space:
   capacity = 112721920 (107.5MB)
   used     = 9442424 (9.004997253417969MB)
   free     = 103279496 (98.49500274658203MB)
   8.376741631086482% used
To Space:
   capacity = 109051904 (104.0MB)
   used     = 0 (0.0MB)
   free     = 109051904 (104.0MB)
   0.0% used
PS Old Generation
   capacity = 2976907264 (2839.0MB)
   used     = 609708576 (581.4634094238281MB)
   free     = 2367198688 (2257.536590576172MB)
   20.481275428806907% used

67050 interned Strings occupying 6996392 bytes.
```

获取到的 VM 参数，可关注如下几项参数的情况

```bash
-XX:+HeapDumpOnOutOfMemoryError  # 是否开启 OOM 时输出 dump 文件，+ 表示开启
-XX:HeapDumpPath=/data/logs      # 若配置该项，dump 文件输出到该路径下

-XX:+PrintGCDetails              # 是否打印 GC 详细信息
-XX:+PrintGCTimeStamps           # 是否打印 GC 时间戳(基准形式)
-XX:+PrintGCDateStamps           # 是否打印 GC 时间戳(日期形式)
-XX:+PrintTenuringDistribution   # 在每次新生代 GC 时，输出幸存区中对象的年龄分布
-Xloggc:/data/logs               # 若配置该项，GC 日志输出到该路径下
```

如果应用没有启用相应的参数输出日志，可以进行以下操作查看相关日志

```bash
# 1.查看应用 GC 次数及平均每次 GC 时间
jstat -gc <pid> 5000 # 每 5 秒显示一次 pid 的进程生成 GC 情况

# 2.查看 JVM 堆内存不同代的具体使用情况
jmap -heap <pid> # 参照上面使用
```



### 参考

[Java OOM 分析](https://jluncc.github.io/2019/06/10/oom-analysis-1/)

[OutOfMemoryError系列](https://blog.csdn.net/renfufei/article/details/76350794)

[深入探究 JVM | 探秘 Metaspace](https://www.sczyh30.com/posts/Java/jvm-metaspace/)