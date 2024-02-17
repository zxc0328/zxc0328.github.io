---
title: "Kubernete安装不完全指南"
date: 2017-10-26 19:21:26
tags: 
- Kubernetes
- Cloud
categories: 
- Kubernetes
- Cloud
---

安装Kubernetes向来不是一件容易的事情。之前在集群上部署好的K8s突然出了问题，于是需要重新安装。这次没有之前那次那么顺利了，出现了很多奇怪的问题。但经过一番实践和总结，总算是得出了Kubernetes国内安装的一个不完全指南。这个指南理论上适用于任意版本的K8s。

<!-- more -->

Master节点的安装过程主要可以参考[**《使用kubeadm安装kubernetes1.7》**](https://blog.jsjs.org/?p=414)。主要的流程是：

+ 安装Docker
+ 安装Kubernetes相关组件
+ Kubeadm init Master节点
+ 安装Overlay Network
+ Join Node节点

下面主要说一些关于如何确定软件版本、如何制作软件镜像源等等细微但是很要命的问题。

### 关于Docker

目前使用**Docker 1.12**版本是比较稳定的。Docker这种基础设施软件没有必要追最新的。在阿里云的CentOS只要直接`yum install docker`就可以安装这个版本了。

### 关于操作系统

本文讲的是在**CentOS 7**上安装Kubernetes。理由是Kubernetes对CentOS支持比较好，网上资料比较多。当然Ubuntu的支持应该也是没问题的。

### 关于K8s源

因为国内的网络环境，我们在服务上明显是不能直接访问国外的镜像源的。所以我们找掌握寻找镜像源，以及在必要的时候自己制作镜像源的技能（因为云服务商的镜像源不一定同步了最新的版本）。

所以我们可以直接用阿里云的源，比如：

```
cat >> /etc/yum.repos.d/kubernetes.repo <<EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
EOF
```

目前阿里云的源是1.7.5的，Kubernetes目前的稳定版本是1.8.x，1.9.x马上要发布了。可见过国内的源虽然可以用，但版本上不会特别新。

第二种办法是在国外服务器上下载镜像然后上传到国内服务器。**这样的好处是可以任意选择自己想要的版本**。具体可以参考使用上文中的《kubeadm安装kubernetes1.7》。

### 关于镜像

Kubernetes安装过程中一个很大的问题，在于相关组件的镜像都是托管在Google Container Registry上的。国内的镜像加速一般针对的是Dockerhub上的镜像。所以国内的服务器是没法直接安装GCR上的镜像的。

这个问题其实很好解决，首先我们可以**自己在本地翻墙拉到镜像，并把镜像push到阿里云的镜像仓库**。拉镜像上传的脚本如下：

```
#!/bin/bash
set -o errexit
set -o nounset
set -o pipefail

KUBE_VERSION=v1.7.5
KUBE_PAUSE_VERSION=3.0
ETCD_VERSION=3.0.17
DNS_VERSION=1.14.4

GCR_URL=gcr.io/google_containers
ALIYUN_URL=registry.cn-hangzhou.aliyuncs.com/muxi

images=(kube-proxy-amd64:${KUBE_VERSION}
kube-scheduler-amd64:${KUBE_VERSION}
kube-controller-manager-amd64:${KUBE_VERSION}
kube-apiserver-amd64:${KUBE_VERSION}
pause-amd64:${KUBE_PAUSE_VERSION}
etcd-amd64:${ETCD_VERSION}
k8s-dns-sidecar-amd64:${DNS_VERSION}
k8s-dns-kube-dns-amd64:${DNS_VERSION}
k8s-dns-dnsmasq-nanny-amd64:${DNS_VERSION})


for imageName in ${images[@]} ; do
  docker pull $GCR_URL/$imageName
  docker tag $GCR_URL/$imageName $ALIYUN_URL/$imageName
  docker login
  docker push $ALIYUN_URL/$imageName
  docker rmi $ALIYUN_URL/$imageName
done
```

K8s每个版本需要的镜像版本号在[kubeadm Setup Tool Reference Guide](https://kubernetes.io/docs/admin/kubeadm/#custom-images)这个文档的的Running kubeadm without an internet connection一节里有写。所以可以根据安装的实际版本来跳帧这个脚本的参数。**注意把上面的镜像地址换成自己的。muxi是你创建的一个namespace，而不是仓库名**。

而Kubernetes也提供了镜像地址相关的配置项，一共有三个：

一个配置文件：

```
cat > /etc/systemd/system/kubelet.service.d/20-pod-infra-image.conf <<EOF
[Service]
Environment="KUBELET_EXTRA_ARGS=--pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/muxi/pause-amd64:3.0"
EOF
```

两个环境变量：

```
export KUBE_REPO_PREFIX="registry.cn-hangzhou.aliyuncs.com/muxi"
export KUBE_ETCD_IMAGE="registry.cn-hangzhou.aliyuncs.com/muxi/etcd-amd64:3.0.17"
```

### Trouble Shooting

解决了获取镜像的问题之后，K8s集群的搭建应该就问题不大了。我们可以通过：

```
kubectl get pods --all-namepaces

kubectl get nodes 

kubectl get cs 
```

这几个命令来看集群的运行情况。

但有一些特殊的问题还是会存在，所以接下来我们就看看如何解决这些问题：

#### Kubeadm init卡住

```
systemctl status kubelet
```

首先中断初始化，查看kubelet的状态。如果出现cgroupfs相关问题，那就需要同步Docker和kubelet的cgroupfs设置。将两者设置为一样的。具体可以看[这里](https://github.com/kubernetes/kubernetes/issues/43805#issuecomment-320965626)。笔者用的Docker 1.12和Kubernetes 1.7的cgroupfs默认都是systemd，所以没有问题。

#### Flannel下Pod不能访问网络问题

之前遇到过k8s+flannel这个组合下Pod不能访问外网的问题。解决方案如下：

在Master节点创建一个busybox Pod：


*busybox.yaml*

```
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: default
spec:
  containers:
  - image: busybox
    command:
      - sleep
      - "3600"
    imagePullPolicy: IfNotPresent
    name: busybox
```


然后运行`kubectl create -f busybox.yaml`创建这个Pod。

在Master节点运行`ping 8.8.8.8`，或者ping另外的公网IP。如果没有成功，则用`traceroute`查看请求的路由列表。一般来说路由列表到容器的网关之后就断了。我们记下这个网关的IP。在镜像中运行`kubectl exec busybox -- ifconfig`，查看eth0设备的IP，这个IP应该就是之前traceroute得到的IP。所以问题出在这个网卡上。

在Master节点运行`ifconfig`，我们看到cni0网卡的IP和之前Pod里的默认网卡的网段是重叠的。所以Pod中的请求就会走这个设备。

Pod访问公网应该走的是节点上的Docker0设备。cni0是Flannel的虚拟网卡，这个网络自然是不通外网的。为了解决这个问题，我们运行：


```
/sbin/iptables -t nat -I POSTROUTING -s 10.24.1.0/24 -j MASQUERADE
```

其中10.24.1.0/24就是cni0设备的IP。

### Links

几个不错的社区和博客。大家遇到问题的时候可以去浏览一下看看。说不定会有解决方案。

+ [Tonybai的博客](http://tonybai.com/articles/)
+ [K8s中文社区](https://www.kubernetes.org.cn/)
+ [Docker.io](http://dockone.io/)





