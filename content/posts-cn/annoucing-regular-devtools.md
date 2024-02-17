+++
title = "发布Regular Developer Tools"
date = 2016-07-26 18:20:33
tags = ['Tooling']
categories = ['Tooling']
hiddenInHomeList = true
+++
---

### 为什么我们需要开发者工具？

现在五花八门的开源前端工具里面，有一些致力于解决前端不够工程化的问题，比如Webpack、Gulp、PostCSS。有些则致力于加强编程语言的特性，比如Babel。而有一部分工具则是为了提高开发者的体验，比如React Devtools、React Hotloader和Redux Devtools。提高开发者体验的工具，从某种角度来说，其实和其他直接参与到前端开发工作流中的工具一样，最终都是为了提高生产力。

React Devtools已经足够惊艳，而Redux Devtools带来的Time-travel Debugging和React Hotloader的热更新则让人直呼这是黑魔法。

所以，被这些开发者工具惯坏的我们，怎么能接受Regular没有Devtools的现实呢？

<!--more -->
### Regular Devtools初印象

Regular Devtools是一款Chrome拓展。主要的功能是展示组件树，以及查看每个组件的数据。开发者对组件的操作，造成组件树和数据的变化，都会在Devtools中实时展现。

比如这样：

![rdt_demo](http://7oxh2b.com1.z0.glb.clouddn.com/rdt_demo_ss.gif)

更强大的是，你可以检查当前组件对应的DOM节点，这个节点会在Chrome Devtools中的Element Tab中出现，就像你平时习惯的右键inspect一样。

反之，如果你正在检查一个DOM元素，此时你可以切换到Regular Tab，Regular DevTools会自动选中那个DOM节点对应的Regular组件（如果那个DOM节点是Regular组件渲染出的）。

比如这样：

![rdt_demo](http://7oxh2b.com1.z0.glb.clouddn.com/rdt_demo_dom_ss.gif)

### 特性

对Regular Devtools有了初步印象之后，下面就详细介绍一下它的功能

+ 左侧Element View显示当前页面的Regular组件树，右侧State View显示被选中组件的状态。
+ Element View和State View都会随着页面上组件和状态的变化实时更新。
+ 提供DOM节点和Regular组件的双向查找：可以点击State View中的Inspect查找当前Regular组件对应的DOM节点。
+ 也可以在Element选项卡中选中DOM节点后切换到Regular选项卡，如果被选中的DOM节点由Regular组件渲染，那么这个Regular节点会被自动选中。
+ 页面刷新时Devtools会自动重新加载，你也可以通过顶栏右侧的按钮手动重载。
+ 通过`include`方式被引入的组件将会在组件树中注明，并且在组件树种作为视觉父节点`this.$outer`子节点展示。

### 安装

读到这里，想必你一定蠢蠢欲动了，那就快来体验一下Regular DevTools吧。

+ **Step 1** 克隆Regular Devtools[仓库](https://regularjs.github.io/regular-devtools/)
+ **Step 2** 打开Chorme的`chrome://extensions/`页面
+ **Step 3** 点击`Load unpacked extension···`
+ **Step 4** 选择你刚刚克隆的仓库文件夹
+ **Step 5** 大功告成!

打开一个使用了Regular组件的页面，打开Chrome Devtools，选择Regular选项卡，如果一切顺利，你将在左边的Element View看到当前页面的组件树，然后就可以开始愉快的开发了！

>注意，你的项目必须使用[Regular v0.4.5](https://regularjs.github.io/)或更高的版本   
>如果你手头上没有使用最新版Regular的项目，可以先拿这个[Regular Todo例子](https://regularjs.github.io/regular-devtools/)尝鲜


### 开发手记

对这款Devtools的技术实现有兴趣的同学可以浏览[Regular Devtools开发手记](http://zxc0328.github.io/2016/07/28/regular-devtools-thought/)这篇博客。

### 已知的问题

这些问题主要是由于某些特殊组件`$outer`属性的指向和正常组件不同造成的。

+ 有`isolate`属性的组件将不会出现在组件树中，其内嵌组件会正常显示。
+ Regular-dnd中的`Draggable`和`Droppable`组件的内嵌组件会和父组件显示在同一个层级。

### 关于我

网易有数前端实习生