---
title: 一个组件的诞生(下)
date: 2016-09-30 10:34:07
tags: 
- Component
categories: 
- Component
---

上一篇中我们实现了一个通用的组件模型。但这个模型在前端这种用户交互密集的应用场景下，会遇到更多的挑战。


### 数据驱动

你接到了这样一个需求，要在banner上加几个指示的圆点（indicator），当前页面是第几个页面，第几个圆点就会高亮。你想着这个需求很简单啊，刷刷刷就写好了。

<!-- more -->

**HTML**

```
<div class="banner">
	<li>
		<img src="{{ url1 }}"/>
		<img src="{{ url2 }}"/>
	</li>
	<div class="button-left"></div>
	<div class="button-right"></div>
	<div class="indicator">
		<li class="item"></li>
		<li class="item"></li>
		<li class="item"></li>
	</div>
</div>
```

**JS**

```
import * as _ from "utility"

var banner = new Banner({
	el: document.querySelector("template"),
	data: {
		index:0,
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
			data.index += direction
			var indicators = document.querySelector(".indicator");
			indicators.map( (item, index) => {
				_.removeClass(item, "hightLight");
				if (index === data.index) {
					_.addClass(item, "highLight");
				}
			})	
		}
	}
})

```

这的确可以实现我们想要的逻辑，但是看起来还是有点累赘。

![](https://pic1.zhimg.com/4f19d188d43babcf6362d2372307492c_b.jpg)

如果我们能把一部分逻辑写在模板里，然后在组件中，只需要修改数据，模板就会进行对应的渲染，是不是很酷呢？

这就是所谓的**数据驱动**。组件帮我们封装了底层的DOM操作细节。我们只需要声明组件的状态就可以了。用户的交互造成组件的状态改变，然后状态的改变又造成了DOM层面的重新渲染。组件就像是一个黑盒子，接收我们输入的数据，输出视图。计算机做的事情其实就是处理数据。因此，我们做这样的抽象，是非常**符合本能的一种抽象**。可以让我们集中精力来关注业务逻辑。同时也**降低了前端组件的复杂度**。

那就让我们来试试吧。

**HTML**

```
<div class="banner">
	<li>
		<img src="{{ url }}" for="url in urls"/>
	</li>
	<div class="button-left"></div>
	<div class="button-right"></div>
	<div class="indicator" >
		<li class="item" 
		class="{{ $index === index ? 'highLight' : 'normal' }}"
		  for="url in urls"></li>
	</div>
</div>
```

**JS**

```
import * as _ from "utility"

var banner = new Banner({
	el: document.querySelector("template"),
	data: {
		index:0,
		url: ["/somewhere/1.jpg", "/somewhere/2.jpg"]
	},
	events: {
		".left-button.click": this.onSwitch.bind(this, 1),
		".right-button.click": this.onSwitch.bind(this, -1)
	},
	methods: {
		onSwitch: function(event, direction) {
			// 切换banner逻辑	
			data.index += direction //更新index
			this.render() //重新渲染
		}
	}
})

```

> 这里有一个新的`for="url in urls"`，我们把这个叫做指令。指令可以理解为为模板增加分支、循环等逻辑的标记。模板引擎在编译这个指令时就会按数据进行对应的渲染。这里用到的是循环指令，作用就是按给定的数据循环遍历，每遍历到一个就讲数据填充到模板中进行渲染。`$index`是这个指令中一个特殊的变量，代表当前遍历到的元素的下标。

这样就清楚多啦。我们修改`data.index`，然后调用`this.render()`这个生命周期方法。不用操纵任何DOM节点。视图就更新了。是的，我们在某种程度上实现了数据驱动！

### Vue and beyond

大家可以对比一下Vue的初始化：

```
new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue.js!'
  },
  methods: {
    reverseMessage: function () {
      this.message = this.message.split('').reverse().join('')
    }
  }
})
```

和我们的组件的初始化：

```
new Banner({
	el: document.querySelector("template"),
	data: {
		message: 'Hello Vue.js!'
	},
	events: {
		".some-button.click": this.oClick,
	},
	methods: {
		onClick: function() {
			// 业务逻辑
		}
	}
})

```


是不是很像呢，除了`events`这个选项。因为Vue把这些逻辑都放到指令中了，比如：

`<button v-on:click="reverseMessage">Reverse Message</button>`

所以Vue的本质，就是一个用来构建用户界面的组件库。和我们一步步的设计API，调整模板，构建出来的这个组件，没有本质的区别。

但Vue为什么好用呢？首先Vue有完善的指令系统，比如`v-for``v-for`和`v-if`等。其次是Vue实现了数据的双向绑定。双向绑定就是用户在界面上输入的数据，可以被同步到组件的状态中。刚才，我们在自己的组件中实现了将组件状态同步到界面，而双向绑定就以为着这个同步可以是反向的。

> 双向绑定的实现是一个单独的话题。Angular使用的脏检查是一个流派。Vue使用`Object.defineProperty`API来实现。React倒是没有数据绑定的概念，不过React的Virtual DOM Diff从某种角度来说，其实也是脏检查。

我们所实现的其实是一种粗犷的同步方式。并没有实现所谓的数据“绑定”。数据绑定就以为着为每个模板中被绑定的数据创建一个`Watcher`，这个`Watcher`决定数据变化时做什么操作。比如对于DOM中的表达式，`Watcher`就会对这个表达式重新求值，然后更新这个表达式所在的DOM节点的对应属性。

按之前我们的做法，如果一个数据变动，就会造成整个组件的重新渲染。这样明显是低效的。数据绑定可以做到特定DOM节点的更新，这是目前前端组件的主流。

### 从MVC到MVVM

前端的MVC或者MVVM，View一般就指模板（模板可以看成是抽象的View，DOM则是这个抽象View的implementation detail）。我们的第一个组件构造器构造出来的对象其实是MVC中的Controller。里面的`data`属性应该被单独拿出作为一个Model对象，这个对象可以通过观察者模式和Controller进行通信。

MVVM中的Model就是一个plain object，比如我们打造的组件中的`data`。Vue这样的MVVM框架，Vue实例其实是MVVM中的VM，即ViewModel。Model，如上文说的，是一个挂载在`data`属性上的普通的对象。我们修改这个对象，就会驱动View的更新。MVVM的独特处之一就在于此。MVVM的独特处之二就是，Model和View直接数据的双向绑定。VM和MVC中Controller的不同之处在于通信方式的不同。MVVM中各个部分的联动比较复杂，我们叫reactive system。MVC中的事件广播模式，主要是在组件之间的通信这个层面比较明显。




