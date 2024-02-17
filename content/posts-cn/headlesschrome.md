---
title: "Headless Chrome截图服务实战"
date: 2017-08-03 09:26:25
tags: 
- Headless Chrome
categories: 
- Headless Chrome
---

### TL;DR

给太长不看同学的内容速览：

+ Headless是Chrome 59中加入的一种新的运行模式
+ Headless Chrome可以替代PhantomJS，并且更加强大
+ 可以通过Chrome DevTools Protocol这个协议对远程的Chrome浏览器进行调试
+ chrome-remote-interface是Nodejs下Chrome DevTools Protocol的封装
+ 可以使用`Emulation.setVisibleSize`对**整个页面**进行截屏

<!-- more -->

### PhantomJS的问题

之前有数报表的导出图片功能是用PhantomJS做的。PhantomJS有两个很大的问题：第一是，它的渲染引擎和JavaScript引擎基于Qt5，版本不是很高，所以渲染的时候会有一些兼容问题，而且JavaScript引擎也相对比较古老（最新的PhantomJS release是2.1版，这个版本基于Qt5.5。Qt5.5使用的Chromium内核版本是40，Chromium现在最新版本是62）。第二，PhantomJS现在已经处于一种维护不多的状态（Github上有1901个open issues）。

作为一个个人项目，PhantomJS在各种自动化测试以及页面自动化操作中被广泛使用，达到了很高的高度。但因为以上两个缺点，使用PhantomJS将不会是长久之计。

### Chrome的Headless模式

Headless Chrome其实不是一个全新的工具，而是普通的Chrome浏览器的headless模式。headless就是指Chrome的UI部分是不运行的。

所以只要你的机器上安装了Chrome 59+，你就可以使用Headless Chrome。相比之前`npm install`时经常要从bitbucket下载PhantomJS binary的麻烦事，Headless Chrome要方便不少，毕竟Web开发者一般都安装了Chrome。

你可以在命令行中用headless模式启动Chrome：

```
chrome \
  --headless \                   # Runs Chrome in headless mode.
  --disable-gpu \                # Temporarily needed for now.
  --remote-debugging-port=9222 \
  https://www.chromestatus.com   # URL to open. Defaults to about:blank.
```

我们可以直接在Chrome的CLI中进行一些操作，比如截屏：

```
chrome --headless --disable-gpu --screenshot --window-size=1280,1696 https://www.chromestatus.com/
```
但一般我们很少会这样直接使用Headless Chrome。对这部分有兴趣的同学可以看[官方文档](https://developers.google.com/web/updates/2017/04/headless-chrome)，这里就不多说了。

Headless Chrome的最大的优点就是，它就是Chrome，所以可以保持Evergreen，也就是持续的更新。并且我们可以在Headless模式使用Chrome带来的所有的现代Web平台的特性。所以Headless Chrome就成为了PhantomJS的完美升级版替代品。

### 强大的Chrome DevTools Protocol

要在脚本中和Chrome进行交互，需要用Chrome DevTools Protocol这个协议。所以这里首先介绍一下这个协议。

简单的说，我们可以在启动Chrome的时候开启一个用于远程调试的端口。然后我们可以在浏览器或者其他客户端中和Chrome建立socket连接，并使用Chrome DevTools Protocol进行通信。

Chrome DevTools Protocol通信的格式是JSON。比如我们想截屏，就可以发一个消息：

```
{
	id:30,
	method:"Page.captureScreenshot",
	params: {
		format:"png",
		quality:100
	}
}
```

返回的消息：

```
{
	id:30,
	data:"data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIA..."	
}
```

具体的API可以参考[文档](https://chromedevtools.github.io/devtools-protocol/)，我们从文档里可以看到，Chrome DevTools Protocol包含的范围非常广。简单的说，我们平时在Chrome DevTool里面可以做到的事情，能获取的数据，我们使用Chrome DevTools Protocol都可以做到。因为Chrome DevTool其实就是基于这个协议进行开发的一个B/S架构的工具。当然这个协议也应该是随着Chrome DevTool的开发，被标准化。现在不仅仅是Chrome，其他浏览器也支持部分Chrome DevTools Protocol。

我们在Chrome的拓展里也可以调用这一套API。所以Chrome拓展的潜力是很大的。可以通过合理使用Chrome DevTools Protocol获得更接近自带DevTool的debug体验。也可以对内存、DOM、渲染等数据进行二次的分析和利用。

### Nodejs服务

直接使用Chrome DevTools Protocol还是比较麻烦的。社区已经有了封装好的Nodejs包[chrome-remote-interface](https://github.com/cyrus-and/chrome-remote-interface)可以直接使用。我们可以直接像调用JavaScript API那样来和Chrome进行通信。

下面就演示一下如何在Node中进行Chrome的截屏：

```
const CDP = require('chrome-remote-interface');
const Koa = require('koa');
const app = new Koa();

const viewportWidth = 1440;
const viewportHeight = 900;
const delay = 500;

app.use(async ctx => {
  ctx.body = await capture(ctx.request.query.url);
});

app.listen(3000);

const capture = function (url) {
  return new Promise((resolve, reject) => {
    CDP.New().then((target) => {
      return CDP({ target });
    }).then(async (client) => {

      const { Page } = client;
      await Page.enable();
      await Page.navigate({ url: url });
  
      Page.loadEventFired(() => {
        setTimeout(async () => {
          const { data } = await Page.captureScreenshot();
          resolve(Buffer.from(data, 'base64'));
          const id = client.target.id;
          client.close();
          CDP.Close({ id });
        }, delay);
      });
    }).catch((err) => {
      console.error(err);
    });
  })
}
```

chrome-remote-interface将一个socket通信的来回封装成了一个异步的函数调用，返回一个Promise。在Node7.8+的环境，我们可以用async/await来轻松的进行流程控制。这里是具体的[Demo](https://github.com/cyrus-and/chrome-remote-interface/wiki)和[API文档](https://github.com/cyrus-and/chrome-remote-interface#api)，基本和Chrome DevTools Protocol里的接口是一一对应的。

这个服务监听3000端口，在请求中拿到url参数，调用`capture`函数截屏。我们这里默认Chrome的调试端口是127.0.0.1:9222。如果需要调整，可以在初始化CDP实例的时候传入参数。

我们调用`CDP.New()`初始化一个新的Tab，等待Page加载完成，打开url，然后等待页面的加载事件触发之后执行回调，在回调里调用截屏API并且获取数据。最后我们关闭这个Tab。整个截图的流程就是这样。


### Tip：截取整个页面

我遇到的一个很大的问题就是，页面比较长，我希望截取的图片的高度就是页面的高度，也就是说我希望给整个页面截屏。

一开始我截屏的结果是这样的：

![](https://raw.githubusercontent.com/zxc0328/for-picgo/master/64c45edcgy1fi6ilca424j20ct08y0t3.jpg)

这和想象的不一样啊，下面的大半截都没有了 ![xiaohuangji](https://raw.githubusercontent.com/zxc0328/for-picgo/master/64c45edcgy1fi6if8s30uj203d023q2q.jpg)

所以要如何截取整个页面的呢？我在chrome-remote-interface的wiki里看到了一篇[文章](https://medium.com/@dschnr/using-headless-chrome-as-an-automated-screenshot-tool-4b07dffba79a?1)。然而截屏的时候报错，说`Emulation.forceViewport`不存在.原来这个API已经在新版的Chrome中被废弃了。

最终我找到了一篇[神奇的博客](https://jonathanmh.com/taking-full-page-screenshots-headless-chrome/)，完美的解决了我的问题，核心代码是这样的：

```
Page.loadEventFired(async() => {
    if (fullPage) {
      const {root: {nodeId: documentNodeId}} = await DOM.getDocument();
      const {nodeId: bodyNodeId} = await DOM.querySelector({
        selector: 'body',
        nodeId: documentNodeId,
      });

      const {model: {height}} = await DOM.getBoxModel({nodeId: bodyNodeId});
      await Emulation.setVisibleSize({width: device.width, height: height});
      await Emulation.setDeviceMetricsOverride({width: device.width, height:height, screenWidth: device.width, screenHeight: height, deviceScaleFactor: 1, fitWindow: false, mobile: false});
      await Emulation.setPageScaleFactor({pageScaleFactor:1});
    }
  });

  setTimeout(async function() {
    const screenshot = await Page.captureScreenshot({format: "png", fromSurface: true});
    const buffer = new Buffer(screenshot.data, 'base64');
    fs.writeFile('desktop.png', buffer, 'base64', function(err) {
      if (err) {
        console.error(err);
      } else {
        console.log('Screenshot saved');
      }
    });
      client.close();
  }, screenshotDelay);

}).on('error', err => {
  console.error('Cannot connect to browser:', err);
});
```
原理就是用Emulation这个API，去调整页面的一些属性。其中核心的方法是`Emulation.setVisibleSize`，文档里对这个方法的说明是：

> Resizes the frame/viewport of the page. Note that this does not affect the frame's container (e.g. browser window). Can be used to produce screenshots of the specified size.

然后我们就可以愉快的给整个页面截屏了，比如这样：

![full image](https://raw.githubusercontent.com/zxc0328/for-picgo/master/64c45edcgy1fi6i22r84kj21402hdjux.jpg)


### Docker镜像

最后我们需要部署这两个服务，社区里有自制的Headless Chrome的Docker镜像，比如[yukinying/chrome-headless-browser-docker](https://github.com/yukinying/chrome-headless-browser-docker)。注意如果我们需要调整Chrome的默认window大小，可以修改[Dockerfile](https://github.com/yukinying/chrome-headless-browser-docker/blob/master/chrome/Dockerfile)然后自行build镜像。

```
ENTRYPOINT ["/usr/bin/dumb-init", "--", \
            "/usr/bin/google-chrome-unstable", \
            "--disable-gpu", \
            "--window-size=1440,900"  # 在ENTRYPOINT命令参数中加上window-size
            "--headless", \
            "--remote-debugging-address=0.0.0.0", \
            "--remote-debugging-port=9222", \
            "--user-data-dir=/data"]
```

Node服务的镜像就比较简单了：

```
FROM node:latest
# Create app directory
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app
COPY . /usr/src/app

RUN npm install koa chrome-remote-interface
EXPOSE 3000
CMD [ "node", "index.js" ]
```
生产中应该用pm2这样的Process Manager来保持进程的运行。依赖也应该写在package.json里。

服务之间的关系如下：

![service](https://raw.githubusercontent.com/zxc0328/for-picgo/master/64c45edcgy1fi6j3jqs0cj20fj07saaf.jpg)

### Links

如果大家想使用Headless Chrome的话，最好还是去浏览一下相关的文档，因为这些内容都属于比较新的东西，变化也比较快。本文主要是简单的介绍一下这方面实现的可行性。下面是相关的链接：

+ [Getting Started with Headless Chrome](https://developers.google.com/web/updates/2017/04/headless-chrome)
+ [Chrome DevTools Protocol Viewer](https://chromedevtools.github.io/devtools-protocol/)
+ [chrome-remote-interface](https://github.com/cyrus-and/chrome-remote-interface)
+ [Taking Full Page Screenshots with Headless Chrome](https://jonathanmh.com/taking-full-page-screenshots-headless-chrome/)


