---
title: "Ninja:dev server made easy"
date: 2016-07-05 20:32:43
tags: 
- Tooling
categories: 
- Tooling
---

[Ninja](https://github.com/Muxi-Studio/ninja)是一款用于前后端分离开发的本地开发服务器，也被称为是前端容器。它有以下一些特性：

+ HTML、CSS的热重载
+ 根据配置文件自动生成路由
+ 根据配置文件自动读取在线/本地的虚拟数据
+ 支持`Jinja2`语法（以及其他Nodejs模板）
+ 支持路由代理
+ 支持`webpack`（通过`webpack-dev-middleware`）

<!-- more -->

### Ninja的前世今生

在木犀，之前我们的前后端分离开发方案是这样的：在本地运行一个简单的`Flask`应用，只包含路由，在路由中读取虚拟数据并渲染`Jinja2`模板，然后运行这个应用，进行开发。前端模板开发完成后和后端直接对接调试。

这种方案有一些明显的问题，首先是这个`Flask`应用没法规范化为一个命令或是一个包，这就需要我们每次在构建前端仓库时都手动写路由等逻辑，这显然是不现实的。还有一个问题就是，`Flask`的Python环境与目前前端工具生态的主流Nodejs格格不入，想要接入`webpack`等前端工具，我们必须使用Nodejs来构建我们的dev server。

于是我们需要一款能支持`Jinja2`语法，同时又能满足前端开发各种需要的dev server。Ninja就在这个节点上横空出世了！

### 基本用法

Ninja的上手非常简单，首先安装Ninja

```
npm install -g ninja_cli
```
下面以Ninja和Webpack一起使用为例，这个例子的源代码在Ninja仓库的[`example文件夹`](https://github.com/Muxi-Studio/ninja/tree/master/example/ninja_webpack_example)下可以找到。

首先我们需要一个`ninja.conf.js`配置文件，当然你也可以选择用命令行参数配置。在这个文件里我们要配置一些必要的参数：

```
module.exports = {
	template: "swig", // 模板引擎名
	mock: "/mock/mock.json", // 虚拟数据位置
	webpack: true, // 启用webpack支持
	templateDir: "/template", // 模板文件目录
}
```

然后在webpack的配置里要注意`entry`和`output`的配置：

```
  entry: [
    path.resolve(__dirname, './index.js')
  ],
  output: {
    path: '/',
    publicPath: 'http://localhost:3000/',
    filename: 'bundle.js'
  },
```

webpack的中间件提供静态文件，我们可以在Ninja的3000端口访问到，所以在模板中的静态文件路径可以写`/`+文件名。如`<script src="/bundle.js"></script>`。

配置好Webpack之后，我们写两个简单的模板

```
// base.html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Ninja Test</title>
</head>
<body>
	<h2>I'm the head</h2>
	{% block content %}{% endblock %}
	<a href="/">index page</a>
	<a href="/second">second page</a>
	<script src="/bundle.js"></script>
</body>
</html>

// second.html
{% extends 'base.html' %}
{% block content %}
	<p>{{ title }}</p>
{% endblock %}
```

然后写一个mock文件`/mock/mock.json`，配置路由和虚拟数据：

```
{
	"routes": [
		{
			"name": "home",
			"endpoint": "/",
			"template": "home",
			"async": false // 同步路由
		},
		{
			"name": "second",
			"endpoint": "/second",
			"template": "home",
			"async": false
		},
		{
			"name": "api.courses",
			"endpoint": "/api/v1.0/courses",
			"async": true // 异步路由（API）
		}
	],
	"data":{
		"home":{
			"title": "hello"
		},
		"second":{
			"title": "second"
		},
		"api.courses":{
			"title": "second"
		}
	}
}
```

然后启动Ninja

```
$ ninja
```
之后Ninja会自动用Chrome打开`http://localhost:3000/`。并显示我们首页模板。大功告成！

![ninja_start](http://7xqk8r.com1.z0.glb.clouddn.com/ninja_start.gif)


### 开发手记

Ninja的主体就是一个Express server，读取配置文件然后生成路由，填入虚拟数据和模板。Livereload使用了`connect-inject`做模板的JS注入。用`socket.io`做浏览器与Ninja的实时通信。至于Webpack的集成，直接使用官方的`webpack-dev-middleware`和`webpack-hot-middleware`就可以轻松达成。

总体来说没有遇到太大的困难，当然也可能是因为使用了大量的三方库。`@朱承浩`同学建议Ninja的Web server部分可以自己实现，没必要使用`Express`。我认为JS注入应该也可以自己实现。所以后期可以将这些部分替换为自己的实现。

Ninja是我第一次尝试些CLI应用，也是第一次尝试写前端工具。在功能上借鉴了[`puer`](https://github.com/leeluolee/puer)（但是没看代码，看不懂Coffee Script），在前后端分离的理念上借鉴了[`NEI`](https://github.com/NEYouFan)。在这个过程中我最大的发现是Node生态圈的繁荣，基本上想要的功能都有相应的NPM包封装好了。这次用了[`commander`](https://github.com/tj/commander)做命令行参数的文档生成，[`chalk`](https://github.com/chalk/chalk)给`console.log`加颜色，[`consolidate`](https://github.com/tj/consolidate.js)来统一Node模板调用，[`opn`](https://github.com/sindresorhus/opn)用来打开浏览器。随着我Node水平的提升，在今后的开发中，会尽量少用不必要的三方库。

### 展望

Ninja目前的版本号是`0.1`，等稳定之后会放出`1.0`版。后续版本会加入subcommand，加入脚手架功能，一键搭建前端目录建构和配置文件。云端Mock数据平台也要做相应的跟进。

总之，Ninja是木犀前后端分离开发实践一个进步。这仅仅只是一个开始，希望大家一起努力，提高开发的效率和开发者的体验。



