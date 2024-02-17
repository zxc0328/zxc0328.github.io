---
title: "使用Kubeadm 1.6部署Kubernetes"
date: 2017-05-24 19:52:27
tags: 
- Kubernetes
categories: 
- Kubernetes
---


本文介绍了如何用Kubeadm 1.6版在Ubuntu 16.04系统上快速部署一个Kubernetes集群。

<!-- more -->

### 环境

阿里云ECS 华南1 可用区A Ubuntu 16.04.2 LTS (GNU/Linux 4.4.0-63-generic x86_64) 专有网络

| 节点类型      | 配置        |内网IP  |
| ------------- |:-------------:| -----:|
|MASTER     | 1 CPU 1GB RAM | 172.18.214.46|
| Node    | 1 CPU 2GB RAM      |    172.18.214.47 |

### 依赖安装&&代理设置

首先要在两个节点都安装Docker和Kubernetes相关的组件。因为相关的镜像都在墙外，所以这里需要挂代理或者自行寻找墙内的源。笔者选择的是挂代理的方案，给Ubuntu配置HTTP代理可以参考[这篇博客](http://dearmadman.com/2015/08/30/use-shadowsocks-in-ubuntu/)。给Docker配置代理可以参考[官方文档](https://docs.docker.com/engine/admin/systemd/#httphttps-proxy)

安装的步骤是按照官网的文档[Installing Kubernetes on Linux with kubeadm](https://kubernetes.io/docs/getting-started-guides/kubeadm/)来的：

```
# 升级包管理的镜像列表
apt-get update && apt-get install -y apt-transport-https
# 将docker和kubernetes相关的镜像源加入列表
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
# 安装docker和kubernetes相关组件
apt-get install -y docker-engine
apt-get install -y kubelet kubeadm kubectl kubernetes-cni
```

### 初始化

在MASTER节点运行：

`kubeadm init`

如果一切正常，最后会有如下的输出：

```
Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run (as a regular user):

  sudo cp /etc/kubernetes/admin.conf $HOME/
  sudo chown $(id -u):$(id -g) $HOME/admin.conf
  export KUBECONFIG=$HOME/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  http://kubernetes.io/docs/admin/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join --token 67e1ac.eac65cabb7d2801c 172.18.214.46:6443
```


### 设置环境变量


上一步中kubeadm会生成配置文件，输出的消息中要求我们设置环境变量`KUBECONFIG`为配置文件的路径。以便后续的使用。

```
sudo cp /etc/kubernetes/admin.conf $HOME/
sudo chown $(id -u):$(id -g) $HOME/admin.conf
export KUBECONFIG=$HOME/admin.conf
```

### Pod网络设置：weaver

到这里，我们已经初始化了一个单节点的Kubernetes集群。要想在集群中加入真正负载应用的Node，我们需要初始化一个Overlay Network。

Overlay Network的选择有很多，比如Flannel和Calico。但经过我个人的踩坑和[参考博客](http://tonybai.com/2016/12/30/install-kubernetes-on-ubuntu-with-kubeadm/)后，最终选择了weaver。

在Master节点运行：

```
kubectl apply -f https://git.io/weave-kube-1.6
```

稍候片刻，运行

```
kubectl get pods -o wide --all-namespaces
```

查看pods的运行情况：

```
NAMESPACE     NAME                                              READY     STATUS    RESTARTS   AGE       IP              NODE
kube-system   etcd-izwz9ap4sedl64wboiyh6cz                      1/1       Running   0          55m       172.18.214.46   izwz9ap4sedl64wboiyh6cz
kube-system   kube-apiserver-izwz9ap4sedl64wboiyh6cz            1/1       Running   0          54m       172.18.214.46   izwz9ap4sedl64wboiyh6cz
kube-system   kube-controller-manager-izwz9ap4sedl64wboiyh6cz   1/1       Running   0          55m       172.18.214.46   izwz9ap4sedl64wboiyh6cz
kube-system   kube-dns-3913472980-l8ghd                         3/3       Running   0          55m       10.32.0.2       izwz9ap4sedl64wboiyh6cz
kube-system   kube-proxy-n5332                                  1/1       Running   0          55m       172.18.214.46   izwz9ap4sedl64wboiyh6cz
kube-system   kube-scheduler-izwz9ap4sedl64wboiyh6cz            1/1       Running   0          54m       172.18.214.46   izwz9ap4sedl64wboiyh6cz
kube-system   weave-net-l86wx                                   2/2       Running   0          48m       172.18.214.46   izwz9ap4sedl64wboiyh6cz
```

如果weave-net和kube-dns这两个pod都处于Running的状态。说明网络初始化成功。

> weave-net如果出现CrashLoopBackOff的错误，可以参考[这里](http://tonybai.com/2016/12/30/install-kubernetes-on-ubuntu-with-kubeadm-2/)的解决方案

### 加入Node

在Node上运行之前Master上输出的：

```
kubeadm join --token 67e1ac.eac65cabb7d2801c 172.18.214.46:6443
```

设置配置文件路径的环境变量：

```
export KUBECONFIG=/etc/kubernetes/kubelet.conf
```

稍后，查看Node的运行情况：

```
kubectl get nodes

NAME                      STATUS    AGE       VERSION
izwz9972b5w4h8a4f1h9z7z   Ready     2h        v1.6.4
izwz9ap4sedl64wboiyh6cz   Ready     4h        v1.6.4
```

两个节点都显示Ready，说明加入成功。

> 默认情况Master是不承担负载的，如果要Master节点也参与Pod调度，可以运行`kubectl taint nodes --all node-role.kubernetes.io/master-`


### 示例应用

节点部署就绪，我们来试着部署一个应用吧：

```
kubectl create namespace sock-shop
kubectl apply -n sock-shop -f "https://github.com/microservices-demo/microservices-demo/blob/master/deploy/kubernetes/complete-demo.yaml?raw=true"
```

这两行命令会部署sock-shop相关的deployment和service。这些service共同组成了一个逻辑上的袜子商店网站。

等所有Pods都是Running状态了，我们可以运行：

```
kubectl -n sock-shop get svc front-end
```

查看front-end服务的端口：

```
NAME        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
front-end   10.103.179.228   <nodes>       80:30001/TCP   1h
```

端口是30001，然后我们就可以用`http://<MASTER_IP>:30001`来访问服务了:

![socks-shop](http://wx1.sinaimg.cn/large/64c45edcly1ffysdi2jjfj21kw0yhnpe.jpg)

### 查看和升级部署

在任意节点运行：

```
kubectl get depolyment --all-namespaces
```

可以看到当前的depolyment：

```
NAMESPACE     NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kube-system   kube-dns       1         1         1            1           3h
sock-shop     carts          1         1         1            1           1h
sock-shop     carts-db       1         1         1            1           1h
sock-shop     catalogue      1         1         1            1           1h
sock-shop     catalogue-db   1         1         1            1           1h
sock-shop     front-end      1         1         1            1           1h
sock-shop     orders         1         1         1            1           1h
sock-shop     orders-db      1         1         1            1           1h
sock-shop     payment        1         1         1            1           1h
sock-shop     queue-master   1         1         1            1           1h
sock-shop     rabbitmq       1         1         1            1           1h
sock-shop     shipping       1         1         1            1           1h
sock-shop     user           1         1         1            1           1h
sock-shop     user-db        1         1         1            1           1h
```

如果我们想将其中一个服务横向拓展，比如payment服务，我们只需要：

```
kubectl --namespace=sock-shop scale deployment payment --replicas 2
```

一个新的payment pod就被初始化，并被分配到合适的节点上运行。

关于更多deployment相关的更新、回滚的信息请参考[官方文档](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)。


### 一些思考

Kubernetes相对来说还是很容易上手的一个容器集群管理方案。只要我们开发的时候是按cloud native的思路去写，部署就是一件非常简单的事情。可以说几个配置文件就搞定了。Kubernetes接管了部署的更新和回滚，让运维变的轻松、可靠。比如部署的时候不会在新容器没有启动之前就终止旧容器。如果部署出了问题需要回滚，也可以进行一键式的回滚。部署也可以暂定，继续。这样的一套方案相比于手动管理容器，简直就是鸟枪换炮式的升级。

不仅仅是后端服务，我们的前端代码，也应该融入这套体系之中。前端作为一个单独的服务部署。这样可以更好的解耦。
