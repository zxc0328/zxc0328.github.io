---
title: "k3s 安装小记"
date: 2019-06-04 14:22:24
tags: 
- k3s 
- k8s
- cloud
categories:
- k3s 
- k8s
- cloud
---

[k3s](https://k3s.io/) 刚出来的时候，我刚好看到这个项目，然后了解到这是一个轻量级的 k8s 发行版。之前刚好遇到在阿里云[学生机](https://promotion.aliyun.com/ntms/act/campus2018.html)（1C2G）上安装 k8s 后内存占用太多的问题，因此就决定尝试。最后的效果超出了预期，k3s 可以帮助我们在低配置机器上运行 k8s 集群，缓解了 k8s 对于资源占用的压力，降低了服务器的成本。

<!-- more -->

### k3s 简介

k3s 是 [Rancher](https://rancher.com/) 推出的轻量级 k8s。k3s 本身包含了 k8s 的源码，所以本质上和 k8s 没有区别。但为了降低资源占用，k3s 和 k8s 还是有一些区别的，主要是：

+ 使用了相比 Docker 更轻量的 [containerd](https://containerd.io/) 作为容器运行时（Docker 并不是唯一的容器选择）
+ 去掉了 k8s 的 Legacy, alpha, non-default features
+ 用 sqlite3 作为默认的存储，而不是 etcd
+ 其他的一些优化，最终 k3s 只是一个 binary 文件，非常易于部署

所以 k3s 适用于边缘计算，IoT 等资源紧张的场景。同时 k3s 也是非常容易部署的，官网上提供了[一键部署的脚本](https://raw.githubusercontent.com/rancher/k3s/master/install.sh)。

### 安装环境

 本文的安装环境：
 
 + 阿里云 1C2G 机器若干，运行 CentOS 7.6 64位
 + k3s [v0.5.0](https://github.com/rancher/k3s/releases/tag/v0.5.0)


### 安装脚本

`https://get.k3s.io` 这是 k3s 的安装脚本。我们直接运行这个脚本就可以安装 k3s。因为我们需要在 k3s 运行之前做一些事情，所以运行脚本的时候我们选择只安装，不启动 k3s

```
// 下载脚本
curl -sfL https://get.k3s.io > install.sh
// 运行脚本
INSTALL_K3S_SKIP_START=true ./install.sh
```

如果速度太慢，可以把 binary 手动下载然后传到国内的对象存储，然后去脚本里面把 binary 地址改成国内的地址。


### 镜像获取 && 启动 k3s

安装玩之后我们还要做一个事情，就是把之后 k3s 要用到的一些镜像下载到本地。因为 k3s 用的 pause 镜像地址是 gcr.io 的，所以国内是访问不了的。

> k3s 提供了 air-gap support 这个特性来支持镜像的本地预加载。这个特性本身是为了无法访问外网的环境准备的。国内的环境其实也是等于是没法访问外网，所以刚好可以用这个特性解决问题。一开始 k3s 刚出来的时候没有这个特性，所以只能重新编译 k3s binary 然后改掉硬编码的镜像地址了。


我们去 k3s 的 release 里面获取 `k3s-airgap-images-$ARCH.tar` $ARCH 是我们服务器的 CPU 架构。想办法把这个文件传到服务器上。


然后运行：

```
sudo mkdir -p /var/lib/rancher/k3s/agent/images/
sudo cp ./k3s-airgap-images-$ARCH.tar /var/lib/rancher/k3s/agent/images/
```

最后运行：

```
systemctl start k3s
```

然后 k3s 就启动了。


可以运行 `kubectl` 和 `netstat -nplt` 查看 k3s 是否正常启动。值得注意的是要看看 `ipconfig` 里是否出现了 flannel 的网络设备。之后的操作就和 k8s 一样了，用 `kubectl` 命令进行操作。

这个时候我们启动的是一个 k3s server（master节点），当然这个节点本身也是一个 worker Node。我们如果需要更多的节点，就需要手动加入更多。

### 加入节点

和 master 节点安装一样，首先获取安装脚本，然后运行：

```
K3S_TOKEN=xxx K3S_URL=https://server-url:6443 INSTALL_K3S_SKIP_START=true ./install.sh 
```

> token 是从 master 节点的 `/var/lib/rancher/k3s/server/node-token` 文件里获取的。

就可以安装节点了，然后按上一节里的复制镜像到本地目录。

最后启动节点：

```
systemctl start k3s-agent
```



### 一点黑魔法


如果我们用的是多台学生机，就肯定会遇到一个问题，这些机器不在同一个局域网里面，用内网 IP 是没法相互访问的（因为一个账号只能有一台机器，专有网络是无法跨账号的）。


如果机器之前内网 IP 不通，那 k3s 就算搭建起来了，节点之间的通信也是有问题的。

因此我们需要一点黑魔法，这个魔法就是阿里云[云企业网](https://help.aliyun.com/product/59006.html)。云企业网可以将多个 VPC 连接起来，可以跨账号，跨区域。这就是我们想要的东西！阿里云真香~

把我们的所有节点都加入云企业网之后，所有的节点之间就是内网互通的了。这之后再安装 k3s，整个集群就可以正常的工作了。


### 其他


k3s 默认安装了 Traefik，如果不想用这个，可以在启动的时候加入参数：

```
INSTALL_K3S_SKIP_START=true INSTALL_K3S_EXEC="--no-deploy traefik" ./install.sh
```

官方的安装脚本写的很好，如果需要一些定制化的安装需求，多看看安装脚本，往往可以解决问题。