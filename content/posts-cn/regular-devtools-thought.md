---
title: "Regular Devtools开发手记"
date: 2016-07-28 15:52:00
tags: 
- Tooling
categories: 
- Tooling
---

### Regular Devtools的前辈们

[Dan Abramov](https://github.com/gaearon)的[Redux Devtools](https://github.com/gaearon/redux-devtools)把前端框架开发者工具推向了主流视野，他在React Europe 2015上的这个[演讲](https://www.youtube.com/watch?v=xsSnOQynTHs)被疯转。Redux Devtools可以说是独领风骚，开启了新一代前端框架开发者工具的潮流。

[React Devtools](https://github.com/facebook/react-devtools)也可以算是前开发者工具的祖师爷之一了。React Devtools不但有常用的组件树查看功能，还能直接查看JSX代码，筛选组件，双向修改state，查看组件源码等等。

而Regular Devtools主要参考的则是[Vue Devtools](https://github.com/vuejs/vue-devtools)。Vue Devtools是一个轻量，简洁的Devtools，只实现了开发者工具中最核心的功能。而Vue与Regular在某些方面比较相似，所以Vue Devtools就成为了我开发过程中的一个重要参考。

<!--more-->

### Chrome Extension组成

其实前端开发者们熟悉的Chrome Devtools是用Web技术开发的。验证的方法很简单，如果你把Chrome Devtools设置为单独窗口，然而再按下打开Chrome Devtools的快捷键的话（`Control+Shift+I`），你可以开启一个'meta inspecting'，也就是打开第二个开发者工具来检查第一个开发者工具的DOM元素。当然了，如果你熟悉Chrome Devtools团队的一些知名开发者，比如[Addy Osmani](https://github.com/addyosmani)和[Paul Irish](https://github.com/paulirish)的话，你可能对Chrome Devtools是用Web技术开发的这个事实习以为常。正如V8并不是完全的C++实现，有一部分API是JavaScript实现，Chrome的界面也不都是原生的，Hybrid无处不在。

好了，下面我们来说说Chrome Extension。Chrome Extension其实就是使用Web技术来开发的，只不过和一般的页面不同的是，Chrome Extension的执行环境可以访问到一系列的`chrome.*`API。比如用`chrome.tabs`API来打开新tab，或者监听和tab有关的一系列事件。完整的API文档在[这里](https://developer.chrome.com/extensions/api_index)。

Chrome Extension一般由以下几个部分组成：

+ UI page
+ Content Script
+ Background Script
+ Devtools Panel page

所谓UI page，就是用户点击导航栏中Extension的图标之后出现的弹出框。这个框是一个HTML页面，包含了Extension的UI逻辑。

如果你的Extension想要访问当前用户页面的DOM结构，那你必须使用Content Script。Content Script的特点就是，它可以访问当前用户页面的DOM结构，但除此之外，当前用户页面上的`window`对象等全局变量以及JavaScript脚本运行后产生的变量都无法访问。Content Script是运行在沙盒中的，它只被允许访问页面的DOM结构。不过Content Script作为Extension的一部分，可以和Background Script通信。

如果我们把一个Chrome Extension看成一个Web应用，Content Script和UI page可以一起被当做Extension这个应用的View层。

Background Script，正如名字所示，是Extension运行在后台的一个脚本。这个脚本可以和Content Script通信。

如果你开发的是一款Chrome Devtools Extension，类似Regular Devtools，那你就需要Devtools Panel page了。这个页面负责在Chrome Devtools里新建tab，并且展示tab中的UI。

### Regular Devtools的组成

![pic](http://7oxh2b.com1.z0.glb.clouddn.com/Screen%20Shot%202016-07-29%20at%202.28.44%20PM.png)

Regular Devtools的组成和普通Extension不同之处在于没有UI page，取而代之的是Devtools Panel page。这其实没有太大的区别。只不过UI部分由Devtools Panel page来展现而不是UI page。

上图是Regular Devtools的大致结构。其中Content Script和Background Script两部分构成Model层，负责数据的搜集和处理。Devtools Panel page则是UI层，负责展现数据。下面我们来讲讲具体的实现。

### 获取Regular组件实例

首先我们需要获得Regular组件实例。这一步的实现思路大致就是在页面中注入一个Event Emitter（在页面所有脚本运行之前），并挂载在全局的`window`对象上。然后每一个Regular组件的初始化、更新、销毁等生命周期事件发生时都会在Event Emitter上emit相应的事件，并传入实例。这样我们就可以获取到页面中所有的Regualr组件实例以及相应的其他信息了。

这个Event Emitter的实现可以参考

### `inject`大法

说起来很简单，可以刚才讲到Content Script时我们说到过，它无法访问用户页面上的全局变量，因此Content Script无法访问到用户页面的`window`对象，那我们要如何获取挂载在`window`对象上的Regualr组件实例数组呢？

解决办法很简单，Content Script可以访问用户页面的DOM结构，那我们插入一个`script`标签就可以在用户页面的context下执行脚本了。

比如我们可以实现这样一个`inject`函数：

```
function inject(content) {
    var script = document.createElement('script')
	script.textContent = ';(' + content.toString() + ')(window)'
	document.documentElement.appendChild(script)
	script.parentNode.removeChild(script)
}
```


### 跨页面通信

好了，现在我们在页面中注入的脚本已经获取了当前页面下的Regular的实例，并且在Regular的组件发生更新和销毁事件时，Event Emitter都会收到消息。

大家可以再看一下上面的架构图，页面中的消息要通过Background Script，再由background script发送到devtools page，展现给用户。那我们要如何进行跨页面通信呢。

首先一个问题就是，如何将消息从用户页面传送到Content Script，刚才强调了，Content Script是运行在沙盒中的，无法直接访问用户页面中的变量。

不过，幸运的是，`window.postMessage()`方法可以解决这个问题。如果你将`.postMessage`方法的`targetOrigin`参数设置为`"*"`，那Content Script就会收到这个消息！

Content Script与Background Script之间的通信可以使用Chrome Extension提供的API`chrome.runtime.connect`和`chrome.runtime.sendMessage`进行。Background Script与Devtools Page的通信同理。

### 性能优化

Devtools Page中呈现的UI使用了Regular来打造，这些组件根据页面中传来的消息（数据）而呈现不同的状态。

要想改善UI组件的性能，首先要减少操作DOM节点的频率，也就是减少组件渲染的次数。

为此，在注入页面的脚本中，我加入了一个`debounce`函数，相信大家都不陌生。实现如下:

```
// debounce helper
    var debounce = function(func, wait, immediate) {
        var timeout; //Why is this set to nothing?
        return function() {
            var context = this,
                args = arguments;
            clearTimeout(timeout); // If timeout was just set to nothing, what can be cleared? 
            timeout = setTimeout(function() {
                timeout = null;
                if (!immediate) func.apply(context, args);
            }, wait);
            if (immediate && !timeout) func.apply(context, args); //This applies the original function to the context and to these arguments?
        };
    };
```

这个函数的作用就是将一段时间内发生的多个事件转化为最后一个，起一个缓冲的作用。

在用户操作Regular组件的过程中，会频繁的发生组件的初始化，销毁，状态更新等事件。一次操作往往能同时触发多个事件。如果我们不进行debounce，那么Devtools UI会频繁的收到消息，造成短时间内的反复渲染，造成页面的卡顿。

其实用户的一次操作之后，我们只要在Devtools中渲染其最后一次组件状态更新后的组件状态就可以了。所以加入debounce可以明显改善UI的性能。

Regular Devtools比较特殊的一点，在于页面间消息传递的是JSON数据，因此每一次UI的状态都是全量替换的。如果我直接全量替换所有UI组件的状态，所有组件都会被重新创建，那性能可想而知是很不理想的。

而且Regular Devtools的UI组件中除了从用户页面中传来的状态，还有UI组件的本地状态，如果直接全量替换一部分状态，那所有组件都会被重新创建，UI组件的本地状态就无法保留下来了。

对此我的做法是，对老状态对象和新状态对象进行diff，如果部分状态没有改变，则保留其引用。如果状态改变了，是原始值，则直接替换，如果是对象，则递归diff。这样的结果就是，只更新新老状态中不一样的部分，保留相同的部分。

这样一来，Devtools的UI中只更新状态变化部分对应的UI，不变的部分则保留原样（因为组件状态的引用没有变）。页面上组件再多也不卡了。

### 来自Chrome Devtools的黑魔法

Regular Devtools有一个功能，即DOM节点和Regular组件的双向查找。这个是通过Chrome Console CLI的一个API`inspect()`实现的。

原理很简单，向`inspect()`中传入DOM节点，Chrome就会检查那个节点。

至于检查DOM节点反查Regular组件，则也是通过Chrome Console CLI的`$0`变量实现的。Console中的`$0`变量指向的是最近检查过的DOM节点。拿到这个DOM节点之后，查找它属于哪个Regular组件便可以了。

### 结语

Devtools可以提高我们的开发效率，实在是一个很酷的工具。Regular Devtools目前还只有基本的功能。如果你有更好的想法，欢迎参与到Regular Devtools的开发中来！