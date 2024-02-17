---
title: "一个组件的诞生(上)"
date: 2016-09-29 10:24:27
tags: 
- Component
categories: 
- Component
---


你是一个新手前端工程师，今天你接到了一个需求，写一个banner。banner嘛，大家都知道，就是一个用来展示图片的东西，不同的图片可以滚动切换。

你心想着这很简单啊，于是刷刷刷写了一个。

<!-- more -->

**JS**

```
var banner = {
	init: function() {
		this.addEvent();
	},
	addEvent: function() {
		var buttonLeft = document.querySelector(".button-left");
		var buttonRight = document.querySelector(".button-right");
		buttonLeft.addEventListener("click", function() {
			this.switchClassName(1)
		})
		buttonRight.addEventListener("click", function() {
			this.switchClassName(-1)
		})

	},
	switchClassName: function(direction) {
		// 切换Class类名
	}
}

banner.init();
```

**HTML**

```
<div class="banner">
	<li>
		<img src="1.jpg"></img>
	</li>
	<li>
		<img src="2.jpg"></img>
	</li>
	<li>
		<img src="3.jpg"></img>
	</li>
	<div class="button-left"></div>
	<div class="button-right"></div>
</div>

```


![old chap](http://www.troll.me/images/monocle-guy/jolly-god-old-chap-thumb.jpg)

正当你觉得自己可以刷知乎的时候，主管把你叫到一边，告诉你：

+ 如果你这个页面上需要有两个banner，怎么办呢？复制粘贴吗?
+ 如果在需要在某个时间点从页面上移除banner，怎么办呢？手动remove节点吗，销毁对象吗？
+ 如果banner中图片的URL是从服务端API请求到的动态的数据，难道要用一个个用修改图片的src的方法来显示这些数据吗？


听到这里，你的内心是崩溃的。

![y no ](http://www.troll.me/images/y-u-no/css-floats-y-u-no-clear-yourself-thumb.jpg)


### 面向对象大法

这时，你想起了大学时学过的面向对象。你把你的组件改成了这样：

**JS**

```

var Banner = function(options) {
}

Banner.prototype.init = function() {
	this.addEvent();
}

Banner.prototype.addEvent = function() {
	var buttonLeft = document.querySelector(".button-left");
	var buttonRight = document.querySelector(".button-right");
	buttonLeft.addEventListener("click", function() {
		this.switchClassName(1);
	})
	buttonRight.addEventListener("click", function() {
		this.switchClassName(-1);
	})
}

Banner.prototype.switchClassName = function(direction) {
	// 切换Class类名
}

Banner.prototype.destroy = function() {
	//todo:解绑事件
}

//初始化
var banner1 = new Banner();
var banner2 = new Banner();

//销毁
banner1.destroy();
banner2.destroy();
```

棒，这样就可以轻松的使用多个banner，并通过调用`init`和`destroy`等生命周期方法来初始化和销毁组件了。

> 生命周期  
> 这是组件开发中的一个术语。如果把组件比作一个生物，我们设计的API，比如`init`和`destroy`就相当于这个生物的出生和死亡。一个组件从`init`被调用时被初始化，渲染到页面上，然后和用户发生交互，这时调用的方法也属于生命周期的一部分。用户界面随着用户的交互而发生变化，所以组件也不是一成不变的。生命周期方法其实就是在组件的某个阶段会被调用的方法。


### 模板和渲染

还有一个问题没有解决，那就是如何轻松的将数据填充到DOM节点中呢？

#### 字符串拼接大法

你首先想出了一个比较笨的办法：

```
var url = "/images/1.png"
var template = funtion(url) {
	return "<img src='" + url + "'/>";
}

template(url); // "<img src='/images/1.png'/>"
```

这就是传说中的字符串拼接大法。一种非常原始的“模板”。简单的说就是把HTML中不变的部分原样写成字符串，中间的变化的部分，我们叫模板的变量，会被作为参数输入到模板函数中。然后这些变量和字符串拼接起来，就生成了目标HTML字符串。

上例中的`template`函数做的事情，我们叫做“编译”模板。当然这个函数非常简单，只是拼接了一下字符串。一个成熟的模板引擎的编译函数做的事情，要更接近于传统意义上的“编译”。


#### DOM based模板

字符串拼接大法的问题，相信大家都看到了，就是写起来非常的麻烦，各种单引号和双引号。如果要换行还必须用`+`号拼接。

于是我们就想到了，如果我们把模板作为普通的HTML，那就会方便很多。我们只要像写HTML那样把模板写在HTML文件中，比如:

**HTML**

```

<body>
	<div id="template">
		<img src="{{ url1 }}">
		<img src="{{ url2 }}">
	</div>
</body>
```

我们要向获取模板的内容，只需要这样：

`var templateString = document.querySelector("#template").innerHTML`

返回的就是这个模板的字符串了。

假设我们写了一个模板引擎，它的作用就是将`{{`和`}}`花括号内的表达式，在模板的context下面解析，并返回值。

> context，就是指一个特定的作用域。里面有当前作用域下面的变量（标识符）和对应的值之间的映射。模板的context就是指一个有模板中相关变量的作用域。

这个模板引擎是这样使用的:

```
import 'complie' from 'templateEngine'

var templateString = document.querySelector("#template").innerHTML
var data = {
	url1: "/images/1/png",
	url2: "/images/1/png"
}
var HTMLString = compile(templateString, data)

```
当`compile`函数遇到变量时，就会从`data`中去找。`data`在这个例子就是context。

> 插一句，被大家吐槽的`eval`函数可以用来做在特定context下求值的事情，这正是这种模板引擎所需要做的。所以你可以用`eval`来实现一个简单的模板引擎。

那这段模板

```
<div id="template">
	<img src="{{ url1 }}"/>
	<img src="{{ url2 }}"/>
</div>
```

会被解析为

```
<div id="template">
	<img src="/images/1.png"/>
	<img src="/images/2.png"/>
</div>

```

因为这个花括号中可以是任意合法的JavaScript表达式，所以你也可以这样玩

**JS**

```
//其他省略
var data = {
	displayFlag: false,
	url1: "/images/1/png",
	url2: "/images/1/png"
}

```

那这段模板

```
<div id="template" style="{{ displayFlag ? 'display:block':'display:none'}}">
	<img src="{{ url1 }}"/>
	<img src="{{ url2 }}"/>
</div>
```

会被解析为

```
<div id="template" style="display:none">
	<img src="/images/1.png"/>
	<img src="/images/2.png"/>
</div>

```


很神奇，不是吗?

![magic](https://www.ricardomartins.com.br/wp-content/uploads/2015/05/Magic_meme.gif)

#### `render`生命周期方法

因为现在我们的组件不再使用静态的HTML字符串了，我们需要一个函数来输出HTML字符串。所以我们需要在组件上增加一个叫`render`的生命周期函数。

```
Banner.prototype.render = function() {
	this.el.innerHTML = compile(this.el.innerHTML, this.data);
}

```
这里的`el`是这个组件的一个配置项，代表这个组件的根节点。组件的模板来自根节点的内容。模板编译之后输出的HTML字符串又会替换根节点中的模板，从而渲染这个组件。

> 当然，模板所在的节点和渲染组件的节点可以不是同一个，这个是和具体的组件框架实现相关的。只要模板可以以字符串的形式被引入，就可以达到目的了。是否放在HTML的DOM节点中并不是问题的关键。我们可以写单独的模板文件并用webpack等打包工具引入。


### 初始化配置

一个组件中，比如模板根节点，组件某个节点的事件和handler的Map，事件的handler，以及组件的`data`这样的数据，应该在组件实例化的时候，作为配置被传入构造函数中。

所以，让我们改造一下Banner构造函数和`init`方法：

```

var Banner = function(options) {
	this.data = options.data || {};
	this.el = options.el || document.querySelector("body");
	this.events = options.events;
}

Banner.prototype.init = function(options) {
	this.render();//渲染模板
	this.addEvent(this.events);//根据events这个Map来绑定事件
}

```

此时，如果你这样再初始化一个组件：


```
var banner = new Banner({
	el: document.querySelector("template"),
	data: {
		url1: "/somewhere/1.jpg",
		url2: "/somewhere/2.jpg"
	},
	events: {
		".left-button.click": this.onSwitch.bind(this, 1),
		".right-button.click": this.onSwitch.bind(this, -1)
	},
	methods: {
		onSwitch: function(event, direction) {
			// 切换banner逻辑		
		}
	}
})

```

> 这里我们新增了events和methods两个对象，这其实就是将之前写死在组件内部逻辑里的事件绑定和回调函数拿出来，在初始化时传入，由组件内部根据这个配置自动绑定。


并调用`init`方法：

`banner.init()`

这时你的页面上就出现了一个banner！大功告成！


![pp](https://occc3ev3l.qnssl.com/zindex/pp.png)


### API设计和抽象

到这里，我们的组件之旅就达到了一个阶段。我们对比一下就可以发现，传统的Backbone和jQuery UI的组件就是类似这种模式的。

具体的说，首先你的组件是一个构造函数，你通过传入不同的配置来实例化不同的组件。不知道大家有没有发现，我们的组件以及**不仅仅是一个Banner**了，而是一个**通用**的组件。你通过传入不同的模板、数据、事件和回调来实例化不同的组件。可以是一个Banner，也可以是一个评论框等等。

传统前端组件有着相似的生命周期API设计。比如初始化、渲染、销毁等。这个和前端组件的特点有关，就算是React或者Vue这样最新的技术，它的生命周期API也不外乎这几种，再加上一些和自己这个库运行流程相关的特有的API而已。

模板是一种对DOM的封装。模板把我们从手动操纵DOM中解放出来。我们在传统的HTML基础上加入表达式和分支逻辑，让模板可以根据输入的数据渲染出相应的HTML字符串。这实际上就是将HTML作为底层的细节封装起来。模板作为一种**中间形态**存在，是**一种抽象**。抽象，是计算机最**本质**的一种属性。我们在平时的API设计和编码时应该牢记这一点，培养自己的抽象思维。

[下一篇](http://zxc0328.github.io/2016/09/30/let-s-build-a-component-2/)会讲到如何将我们的组件改造成数据驱动的模式，并将这个组件和Vue进行对比。










