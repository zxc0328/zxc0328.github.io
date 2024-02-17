---
title: "前端微服务实践-以木犀通行证为例"
date: 2017-06-05 15:01:58
tags:
- Mircoservices
categories:
- Mircoservices
---

在前端，长期以来困扰我们的一个问题就是，如何分发部署我们的前端代码？之前我们的前端代码是放在后端容器中部署的，因此如果需要更新就需要后端工程师去重启容器，更新代码。现在我们采取的方法是，将前端作为一个单独的服务，使用容器来部署。这样前端代码的部署和其他的代码就没有区别了。前端工程师可以自由的控制前端代码的部署，整个部署流程也变的非常标准化。前端微服务让前端部署变成了一件让人享受的事情。下面就以[木犀通行证](https://github.com/Muxi-Studio/MuxiAuth-fe)为例来讲讲具体的实现。


<!-- more -->


### Nodejs服务

如果只是在容器中放一些前端的静态文件，那不能叫前端微服务。前端微服务是指用Nodejs实现的View层。包括了同步路由以及前端模板。静态文件可以放在前端容器中分发，也可以上传到CDN分发。

Nodejs实现的这个View层（传统后端MVC中的View），主要负责渲染同步路由的模板。API服务则是由其他的服务提供，大家各司其职。前端接管View层有很多好处，前端渲染以及各种网络应用层的优化，都可以由前端自己来控制。目前Nodejs在大公司中早已频繁被用在服务最前端的那一层中了（后端一般是Java）。

具体实现来说，我们选用koa2作为Web框架，大致实现是这样的：

```
const send = require('koa-send');
const Koa = require('koa');
const Router = require('koa-router');
const userAgent = require('koa-useragent');
const path = require('path')
const swig = require('swig');
const router = new Router();
const app = new Koa();

const templateRoot = path.join(__dirname, "../dist/template/main")

app.use(userAgent);

router.get('/', function(ctx, next){
    if (!ctx.userAgent.isMobile) {
        let template = swig.compileFile(path.resolve(templateRoot, "auth.html"));
        ctx.body = template({})
    } else {
        let template = swig.compileFile(path.resolve(templateRoot, "auth_phone.html"));
        ctx.body = template({})
    }
});

router.get(/^\/static(?:\/|$)/, async (ctx) => {
     await send(ctx, ctx.path, {
         root: path.join(__dirname, "../dist")
     });
})

app
    .use(router.routes())
    .use(router.allowedMethods());

app.listen(3000);
console.log('listening on port 3000');
```

使用`koa-router`来写同步路由，路由中返回对应模板（这里的路由还对移动端或者桌面端流量进行了区分）。然后这里还对静态文件路由做了处理，如果用的是CDN分发静态文件，这里就不需要处理了。

### Dockerfile

`Dockerfile`是Docker的配置文件。我们要把这个Nodejs服务build成Docker镜像进行部署，因此要写一个`Dockerfile`，大致是这样的：


```
FROM node:latest

# Create app directory
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app
COPY . /usr/src/app

# Build static file
RUN npm install
RUN npm run build

WORKDIR /usr/src/app/server

# Bundle app source
EXPOSE 3000
CMD [ "npm", "start" ]
```

Dockerfile具体的写法可以看官方文档和这篇博客。木犀通行证中的Dockerfile主要是，构建了静态文件，然后启动了node服务进程。这样只要在任何有Docker的环境下`docker run`这个镜像，就可以运行这个服务了。

### 在阿里云镜像仓库Build镜像

镜像仓库类似是Github，是分发镜像的一个工具。阿里云的镜像仓库可以从Github仓库进行镜像构建，然后我们就可以拿到一个仓库的URL。在构建时我们可以指定版本号（和Git里的tag对应）。这样在部署时就可以部署这个镜像的某个版本。

阿里云的镜像仓库的用法这里就不详细说了。可以自己去阿里云上尝试一下。

### 在Kubernetes部署

*MAE发布后本节需要更新*

目前这一步是由专门的系统管理员来负责的。开发者只要将镜像仓库的地址和tag告诉系统管理员就可以了。管理员会在集群上更新并部署新版本。

### 总结

这个新的部署流程和之前相比，不同的地方主要是现在需要写Nodejs，一个View层服务。需要写Dockerfile。最终交付部署的是Docker镜像。希望大家在实践的过程中能有自己的思考。同时也对云计算时代的软件分发和部署有更多的了解。

