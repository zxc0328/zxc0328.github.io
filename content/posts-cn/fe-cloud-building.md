---
title: "基于Travis CI和Github的前端云构建"
date: 2017-05-22 16:04:00
tags:
categories:
---

最近在思考团队里前端代码部署的问题，之前采用的方案是在本地构建，推到Github上一个专门放build后前端代码的仓库，然后Github的Webhook去触发后端的部署逻辑。代码就从这个仓库里拉取。

这种方案看起来没什么大问题，但总觉得比较awkward。首先这套方案不够自动化，需要大量的人工操作。然后Github的Webhook其实并不是特别好用，如果后期要和我们内部的私有云平台对接起来，还要经过一些桥接才可以。

本来呢，因为最近学了docker的缘故，我想写一个简单的Node服务，用来自动构建代码，然后通知服务端部署。每个应用就是一个单独的容器，这样环境就可以隔离。这个方案想来也不错。直到我仔细研究了一下Travis CI，才发现这个CI真是不简单。云端构建的任务用Travis CI就可以完美的实现。

<!-- more -->

### 关于CI

CI是持续集成的意思，持续集成里主要包括构建和测试代码。之前对Travis CI的印象是可以跑测试，仔细看了之后才发现Travis CI其实是一个云服务，提供了一个虚拟的Linux环境。你可以运行自定义的脚本。这个Linux环境的自由度还是非常大的。对于前端构建来说，Travis CI的网络环境可以快速安装npm包，这是一个非常大的优势。

### `.travis.yml`文件

Travis CI的配置文件其实就是让你写几个生命周期hook，内容一般是shell命令。比如`install`这个hook里主要写一些安装依赖的逻辑，`script`这个hook里主要是写测试和构建的逻辑，`deploy`这个hook里是写部署的逻辑。另外这几个hook都有各自的`before`和`after`版本。总而言之自由度是很大的。

一个示例`.travis.yml`文件。虽然我们不能直接`.travis.yml`中写逻辑，但我们可以运行任意的脚本，所以可以看出`.travis.yml`的能力基本等价于shell脚本。

```
language: node_js
node_js:
  - "7"
install:
  - npm install
script:
  - npm run build
after_script:
  - tar -cvf bundle.tar ./dist
  - node deploy.js
```

### 云端构建

在看过了上节的`.travis.yml`文件之后，云端构建的大致逻辑应该已经非常清楚了。我们在Travis CI的虚拟机中安装node依赖，build代码，压缩代码，然后运行一个js脚本。这个脚本的内容就是将代码上传到CDN。

`deploy.js`中还可以向后端的平台发送部署的请求，以达到自动部署的目的。如果后端是分布式的架构，向管理的节点发送请求即可。


### 一些展望

Travis CI的能力取决于这个虚拟机里提供了怎样的环境。Travis CI支持docker，因此我们可以用Travis CI进行docker镜像的构建和上传。Travis CI支持Nodejs，因此我们可以在虚拟机中安装hexo，进行博客的云端构建和自动部署。云端的构建，由于保证环境的隔离，因此稳定性会比本地高。以上都是Travis CI可能的用途。Travis CI作为一个云服务，在运维方面，还有无限的可能性等我们去探索

