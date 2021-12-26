---

date: 2021-12-26 15:07:46
title: k8s 下 spring-cloud+eureka 架构下服务优雅停机
author: Joden_He
tags: 

  - k8s
categories: 
  - k8s
description: k8s 下  spring-cloud+eureka 架构下服务的优雅停机方案

---

## 概述

在实际生产环境中，由于 eureka 是 AP，导致每次发布程序时，k8s 的 pod 被终止了，但是 eureka 还认为服务存活（缓存），内部没有将流量拦截，部分请求仍然被分发到终止的容器，导致出现 500 的错误，因此优雅停机显得十分重要。

整个方案的关键在于以下几点：

1. 利用 k8s 容器 `PreStop` 钩子触发容器预结束事件
2. curl 方式将 eureka server 中的实例修改为 `DOWN ` 状态
3. 根据 eureka 客户端以及 ribbon 的相关设置，等待合适的时间，再结束

## 定义预结束事件操作

> k8s 在容器被终结之前，会发送一个 preStop 事件，它是阻塞的，所以它必须在删除容器的调用发出之前完成。如果钩子在执行期间挂起， pod 会一直保持 running 状态。

参考 k8s 官方的例子 [`pods/lifecycle-events.yaml`](https://raw.githubusercontent.com/kubernetes/website/main/content/zh/examples/pods/lifecycle-events.yaml)，我们可以根据自己的情况仿照写一下即可：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: lifecycle-demo
spec:
  containers:
  - name: lifecycle-demo-container
    image: nginx
    lifecycle:
      postStart:
        exec:
          command: ["/bin/sh", "-c", "echo Hello from the postStart handler > /usr/share/message"]
      preStop:
        exec:
          command: ["/bin/sh","-c","nginx -s quit; while killall -0 nginx; do sleep 1; done"]
```

为了简单点，我们在命令中直接执行预先写好的 shell 脚本（也可以将shell脚本做的事情写到这个 yaml 文件里）

```yaml
...
spec:
  containers:
  - name: xxx
    # 设置环境变量
    env:
    # 容器 ip
    - name: POD_IP
      valueFrom:
        fieldRef:
          fieldPath: status.podIP
    # eureka 地址
    - name: EUREKA_URL
      value: "http://xxx.xx.xx.xx:8000/eureka"
    # 服务名
    - name: SERVICE_NAME
      value: "xxx"
    lifecycle:
      preStop:
        exec:
          command:
            - "/bin/sh"
            - "-c"
            - "\
              curl https://gist.githubusercontent.com/JodenHe/8f1ce3770d25251ab5ea67ddbf15f27f/raw/56ee444cbf79599791642fb3b3d0915d1ef4282e/spring-cloud-with-eureka-graceful-down.sh \
              /bin/sh -s $EUREKA_URL $SERVICE_NAME $POD_IP 60
              "
```

## shell 脚本

[spring-cloud-with-eureka-graceful-down.sh](https://gist.githubusercontent.com/JodenHe/8f1ce3770d25251ab5ea67ddbf15f27f/raw/56ee444cbf79599791642fb3b3d0915d1ef4282e/spring-cloud-with-eureka-graceful-down.sh)

脚本执行需要四个入参：

1. EUREKA 地址
2. 需要停止的服务名
3. 服务的实例 ip
4. 服务停止缓冲时间

```shell
#!/bin/sh

# spring cloud + eureka 架构下服务优雅停机方案
# 在服务关闭前执行

# 第一个参数为 EUREKA 地址，第二个参数需要停止的服务名，第三个参数为需要停止服务的实例ip（用于获取实例id），第四个参数为等待时间（单位: s）
EUREKA_URL=$1
SERVICE_NAME=$2
INSTANCE_IP=$3
SLEEP_TIME=$4

echo EUREKA_URL: $EUREKA_URL
echo SERVICE_NAME: $SERVICE_NAME
echo INSTANCE_IP: $INSTANCE_IP
echo SLEEP_TIME: $SLEEP_TIME

# 1、将注册中心的服务标记为下线，使得流量不再调度进来
# 1.1 根据实例 ip 获取实例id
INSTANCE_NAME=$(
curl -v -X GET \
  $EUREKA_URL/apps/$SERVICE_NAME \
  -H 'accept: text/html, application/xhtml+xml, application/json;q=0.9, */*;q=0.8' \
  | jq -r ".application.instance[] | select(.ipAddr==\"$INSTANCE_IP\") | .instanceId"
)

echo INSTANCE_NAME: $INSTANCE_NAME

# 1.2 
$(
curl -v -X PUT \
   $EUREKA_URL/apps/$SERVICE_NAME/$INSTANCE_NAME/status?value=DOWN
)

# 2、等待足够长时间，使 eureka 客户端、ribbon 等拉取最新配置信息，比如 60s
sleep $SLEEP_TIME
```



## 参考

[为容器的生命周期事件设置处理函数](https://kubernetes.io/zh/docs/tasks/configure-pod-container/attach-handler-lifecycle-event/)

