---

date: 2021-02-27 15:49:44
title: centos7 搭建 k8s 集群
author: Joden_He
tags: 
  - k8s
  - cluster
categories: 
  - k8s
description: 介绍 centos7 下搭建 k8s 集群
---

## 说明

创建三个 centos 节点：

```text
192.168.56.101 k8s-master
192.168.56.102 k8s-nnode1
192.168.56.103 k8s-nnode2
```

> 虚拟机安装教程参考：[virtualbox 下 centos7 网络配置](/2021/02/27/virtualbox/centos7/centos7-network/)
>
> <font color="red">其中 master 节点需要至少 2 核 2G 配置</font>

## 所有节点

> 以下操作需要在所有节点运行，这里以 master 节点为例

### 修改主机名

> 相应节点叫相应的名字，这里以 master 节点为例

```sh
hostnamectl set-hostname k8s-master
```

修改后重启服务器后生效

![修改主机名](/images/k8s/20210227164745.png)

### 关闭防火墙

```sh
systemctl stop firewalld.service
```

![关闭防火墙](/images/k8s/20210227165345.png)

### 关闭防火墙自启动

```sh
systemctl disable firewalld.service
```

![关闭防火墙自启动](/images/k8s/20210227165552.png)

### 关闭 selinux

```sh
setenforce 0
```

![关闭selinux](/images/k8s/20210227165817.png)

### 关闭 swap

```sh
swapoff -a
```

![关闭 swap](/images/k8s/20210227170026.png)

vim /etc/fstab ，注释掉swap挂载这一行可以永久关闭swap分区

![永久关闭 swap](/images/k8s/20210227170139.png)

### 添加主机名与 IP 对应的关系

vim /etc/hosts 添加如下内容：

```text
192.168.56.101 k8s-master
192.168.56.102 k8s-node1
192.168.56.103 k8s-node2
```

### 将桥接的 IPV4 流量传递到 iptables 的链

```sh
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```

执行完再运行

```sh
sysctl --system
```

![将桥接的 IPV4 流量传递到 iptables 的链](/images/k8s/20210227170527.png)

### 安装 docker

- 卸载旧的 docker

  ```sh
  sudo yum remove docker \
  docker-client \
  docker-client-latest \
  docker-common \
  docker-latest \
  docker-latest-logrotate \
  docker-logrotate \
  docker-engine
  ```

  ![卸载旧的 docker](/images/k8s/20210227170848.png)

- 安装组件

  ```sh
  sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
  ```

  ![安装组件](/images/k8s/20210227171119.png)

- 添加 docker 安装源

  ```sh
  sudo yum-config-manager \
  --add-repo \
  https://download.docker.com/linux/centos/docker-ce.repo
  ```

  ![添加 docker 安装源](/images/k8s/20210227171215.png)

- 安装 docker

  ```sh
  sudo yum install docker-ce docker-ce-cli containerd.io
  ```

  ![安装 docker](/images/k8s/20210227171431.png)

- 安装成功后，查看 docker 版本

  ```sh
  docker --version
  ```

  ![docker 版本](/images/k8s/20210227171712.png)

- 修改 Cgroupfs 为 Systemd (docker 文件驱动默认由 cgroupfs 改成 systemd，与k8s保持一致避免 conflict)

  ```sh
  mkdir -p /etc/docker
  vim /etc/docker/daemon.json
  ```

  内容：

  ```json
  {
    "exec-opts": ["native.cgroupdriver=systemd"]
  }
  ```

- 设置开机启动

  ```sh
  systemctl enable docker
  systemctl start docker
  ```

  ![docker 自启动](/images/k8s/20210227172101.png)

- 查看文件驱动

  ```sh
  docker info | grep Driver
  ```

  ![docker 文件驱动](/images/k8s/20210227172215.png)


### Kubernetes yum源配置

`vim /etc/yum.repos.d/kubernetes.repo`，添加文件内容如下：

```text
[kubernetes]
name=Kubernetes Repo
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
gpgcheck=0
enabled=1
```

### 安装 k8s

```sh
yum install -y docker-ce kubelet kubeadm kubectl
```

- 设置 k8s 开机启动

  ```sh
  systemctl enable kubelet
  ```

  ![k8s 开机启动](/images/k8s/20210227172910.png)

- 启动 k8s 后台 daemon

  ```sh
  systemctl start kubelet
  ```

  


## master 节点运行

> 以下操作只在 master 节点运行，如果操作失败，可以通过先执行 `rm -rf $HOME/.kube` 和  `kubeadm reset` 命令来清理环境重新安装

### 部署 Kubernetes Master

```sh
kubeadm init --kubernetes-version=1.20.4  \
 --apiserver-advertise-address=192.168.56.101   \
 --image-repository registry.aliyuncs.com/google_containers  \
 --service-cidr=192.168.56.0/16 --pod-network-cidr=10.122.0.0/16
```

> apiserver-advertise-address 的地址为当前节点的 ip 地址
>
> service-cidr 设置成当前节点的网段即可

![部署 k8s master](/images/k8s/20210227204801.png)

![k8s master部署](/images/k8s/20210227204938.png)

记录生成的最后部分内容，此内容需要在其它节点加入 Kubernetes 集群之前就执行。根据 init 后的提示，创建kubectl 配置文件

```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
export KUBECONFIG=$HOME/.kube/config
```

![kubectl 配置](/images/k8s/20210227205227.png)

查看 docker 镜像：

```sh
sudo docker images
```

![docker 镜像](/images/k8s/20210227205420.png)

下面就可以直接使用 kubectl 命令了

```sh
kubectl get node
```

![kubectl get node](/images/k8s/20210227205905.png)

### 安装 calico 网络

```sh
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

![安装 calico 网络](/images/k8s/20210227210224.png)

查看 calico 网络是否创建成功：

```sh
kubectl get pods -n kube-system
```

![calico](/images/k8s/20210227210608.png)

再次查看 node，可以看到 master 节点状态为 ready

![master状态](/images/k8s/20210227210755.png)

至此，k8s master节点创建完毕

## node 节点运行

> Node 节点才需要进行的操作

### node 节点加入集群

向集群添加新节点，执行在 kubeadm init 输出的 kubeadm join 命令：复制上面命令，在 node 节点上执行在k8s-node1, k8s-node2 执行：

> token 和密钥在主节点 init 会生成

如果忘记记录 token 和密钥，或者 token 过期了，可以通过以下命令获取（在 master 节点运行）

- 先获取 token

  ```sh
  #如果过期可先执行此命令
  kubeadm token create    #重新生成token
  #列出token
  kubeadm token list  | awk -F" " '{print $1}' |tail -n 1
  ```

  ![token](/images/k8s/20210227211423.png)

- 获取 CA 公钥的哈希值

  ```sh
  openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed  's/^ .* //'
  ```

  ![CA 公钥哈希值](/images/k8s/20210227211528.png)

- 从节点加入集群 （k8s-node1 和 k8s-node2 执行）

  ```sh
  kubeadm join 192.168.56.101:6443 --token token填这里 --discovery-token-ca-cert-hash sha256:哈希值填这里
  ```

  > 192.168.56.101:6443 ---> k8s-master 节点的 ip 和端口

  最终运行命令

  ```sh
  kubeadm join 192.168.56.101:6443 --token ljd4iy.mk633re8cfq42ybu --discovery-token-ca-cert-hash sha256:344376e98d61c6fb52d35ab369e13dcc49c4222972d4b5f821aea4f589fe593f
  ```

  ![从节点加入集群](/images/k8s/20210227212205.png)


在 k8s-master 查看集群节点数：

```sh
kubectl get nodes
```

![节点数](/images/k8s/20210227213819.png)

### 测试 k8s

- 创建 nginx 实例，并暴露端口：

  ```sh
  kubectl create deployment nginx --image=nginx
  kubectl expose deployment nginx --port=80 --type=NodePort
  kubectl get pod,svc
  ```

  ![创建 nginx 实例](/images/k8s/20210227214156.png)

- 验证

  > 在 web 浏览器输入以下地址，会返回 nginx 欢迎界面 
  >
  > http://192.168.56.101:32409/  (该端口为上图所显示)
  >
  > 在 linux 节点用 curl 访问，会返回 nginx html 欢迎页面的内容


### node 节点配置 kubectl 配置

如果想 node 节点也可以使用 kubectl 需要进行 master 节点复制 config 文件到 node 节点

在 k8s-node1 和 k8s-node2 节点执行以下命令

```sh
sudo scp root@192.168.56.101:/etc/kubernetes/admin.conf /etc/kubernetes/admin.conf
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

![node 节点配置 kubectl](/images/k8s/20210227215036.png)

## 参考

[如何从零开始搭建一个完整的K8S集群-------基于CentOS 8系统](https://blog.csdn.net/jacky128256/article/details/106449237)

[kubeadm生成的token重新获取](https://blog.csdn.net/weixin_44208042/article/details/90676155)

[CentOS7 部署K8S集群成功后，重启就不能用了？？？k8s环境自启动](https://www.cnblogs.com/CoderLinkf/p/12410749.html)