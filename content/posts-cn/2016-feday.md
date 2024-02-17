+++
title = "2016 FEDay见闻录"
date = 2016-03-19 22:27:24
tags = ['FEDay']
categories = ['FEDay']
hiddenInHomeList = true
+++
---

这次FEDay是我第一次参加开发者大会，也是第一次去广州。当初主要是冲着两个Facebook的工程师去的，因为我对React好感度很高。总的来说这次的收获还是很多的，也算是值回票价了。

<!--more-->

### Universal Applications by Stephan
首先是FB的[Stephan](https://twitter.com/stopachka)分享Universal Applications。Stephan会说中文，应该是他在台湾待过一段时间的原因。这里有个小插曲就是会场安排了一名翻译，Stephan说一句翻译就翻译一下，这样真的非常的多余。本来是想来听一下原汁原味的国外工程师分享的，结果还是打了折扣。

Stephan说的Universal Applications我之前在开发Readme的时候使用过一部分。就是在服务器端用react-dom的`renderToString`方法生成字符串。同时也运行了Redux的代码，根据客户端传来的state生成了相应的UI。

不过Universal Applications并没有这么简单。这种开发模式总的来说就是写一套代码，在客户端和服务端同时可用。服务端UI的渲染我们已经通过`renderToString`解决了。服务端的路由可以通过react-router的`match`函数来运行相应的UI组件渲染逻辑。而数据获取API的差异可以通过`isomorphic-fetch`这个库来抹平。


### 微信WebApp最佳实践 by 江剑锋

说实话我之前对微信前端的工作不是很了解。看了这个分享之后我清楚多了，微信的前端做的也就是普通前端的事情，并没有涉及前端之外的部分。这个工作有点类似于淘宝无线。不过淘宝这个App的混合程度更高一些，而微信里面的Web和原生的内容之间的分离比较明显，这和微信的特点有关系。

微信主要是为Web内容搭建了一个平台，也就是微信公众号。和原生的交互主要通过JS-SDK来实现。说实话这种开发模式是在世界上很少有的，算是微信的独创。目前Web内容一般来说会以原生的App为平台，而对于众多的普通开发者来说，微信是最好的开放平台。WebApp借微信这个超级App得以迅速的传播。打造了一个生态圈。所以微信Web开发现在已经成为了小公司前端的一个日常开发内容，现场有大量做过微信Web开发的前端同学，这个平台的体量可见一斑。

首先我们看了微信的一些用户数据，前十位的机型里面除了三星之外就是小米。安卓系统版本有不少。因此微信内嵌的X5内核的目的就是抹平这些系统里自带的WebView的差异。

我们在开发微信WebApp的过程中会遇到一些问题，这些多数和微信内嵌的X5内核有关。比如强制缓存，不仅会缓存CSS等资源，还会缓存HTML。要避免这个问题我们可以在URL后加参数。

另外还讲了一些普遍的问题，比如动画卡顿，flex部分支持。Video的`autoplay`无效和`control`条无法隐藏，`autoplay`无法使用是因为产品策略，让用户有选择权。如果一定要使用则可以监听`WXJSBridgeReady`事件。Cookie和localStorage偶尔失效这个问题我还没有遇到过，江剑锋表示客户端存储本来就是不可靠的存储，所以我们要做相应的准备。

接着介绍了WeUI，以及相应的jQuery,react,vue的binding。还有微信的一些调试工具。

最后放了一个消息出来，那就是微信的内核将改用Blink。这样上述的问题都将得到解决，除了`autoplay`依然是按照原来的策略。

### React Tips by 黄士旗

[黄士旗](https://twitter.com/huang47)是台湾人，目前在Facebook工作。他的React Tips的内容主要包括了Container Component和UI Component，HOC高阶函数，Array的函数式API等。

第一次了解Container Component和UI Component这个概念是在一些React boilerplate里以及Dan Abramov的Redux教程里。Dan的[这篇博客](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0#.kv5gzrgje)讲了这个问题。于是我在自己的Redux项目里也把组件分为UI组件和Container组件。简单的说，UI组件是Stateless function，只负责根据传入的数据返回UI组件，具有很高的可复用性。而Container组件则是负责处理数据，具有一些业务逻辑，基本是不可复用的（也不用考虑去复用）。

UI组件的类型很多，如何达到进一步的复用呢？这里黄士旗引入了HOC的概念。我们用HOC作为一个装饰器，传入的UI组件被附加一些UI元素后返回，从而实现了更好程度的复用。我们可以用一个Base UI组件加上各色装饰器来渲染不同类型的UI组件。一个装饰器大概就是这样

````
function makeEditale(baseCompoennt){
	return <div>
				<input />
				<baseCompoennt />
			</div>
}
````

之前我接触过ES7的装饰器，那就是一个class的语法糖，原理和这个是一样的。总之这里我们的原则就是Composition Over Inheritance。

最后黄士旗介绍了Array的`map`，`reduce`，`filter`API的使用。这些函数式特性的API和React就是天作之合。在各种场合都很好用。


### 下一代Web技术运用 by 陈子舜

陈子舜是腾讯云的技术总监，他分享的内容主要是提前运用下一代Web技术来提升Web性能。

在过往，前端面试中提到Web性能，我们总会提到雅虎的35条前端性能优化定律这些。而这些规则在移动端Web开发占主流和新技术出现的情况下是否还适用呢？

前端性能的要求就是我们的首屏必须在1s内渲染，这点我们可以通过Offline Caching来助力。

子舜举了QQ红包的例子。红包相关的代码是用Web技术写的，但是这些代码不能在需要时拉取，而是应该提前拉取并缓存在本地。这一点和Google在推的Progressive App异曲同工。Google IO的IOWA是可以离线使用的。

我们看到现在的离线存储技术有很多种，比如Application Cache和Service Worker。QQ红包中的离线存储是通过Native代码实现的。这些以后可以通过Service Worker来在Web端实现。Service Worker在目前还没有被主流浏览器实现，但这会是未来的方向。

### 前端能力的培养 by Winter

Winter的那套理论我之前在看他画“前端校招重点“的时候就看到过。主要就是20%的知识加上80%的能力。

在说20%的知识的时候，Winter说他看Python语法的文档来学语法。然后通过查论文来了解闭包的概念。总之就是，学知识需要看最权威的东西，追根溯源，搞清问题的本质。

80%的能力分为工程能力，架构能力，编程能力。这些就是靠大量的练习来达到的。一万小时理论看起来有些恐怖，那就从每周20小时开始努力吧。就算是这样，也可以达到一个很不错的水平。

### HTTP/2时代的Web性能 by Holger Bartel

Holger的分享还是很给力的，主要讲了HTTP/2和HTTP/1.1之间的区别，以及我们应该如何在HTTP/2中进行性能优化。以及如何在今天就开始使用HTTP/2。

其中性能优化方面，在HTTP/2时代，我们不需要合并文件了，也不需要inline CSS/image以及分离资源的域名。至于雪碧图，压缩这些工序，还是需要的。

























