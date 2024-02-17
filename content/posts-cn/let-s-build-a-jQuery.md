---
title: "Let's build a jQuery!-Part 1(early release)"
date: 2016-03-11 17:24:00
tags: 
- jQuery 
- JavaScript
categories: 
---

这个系列博客的灵感来源于[这篇博客](https://limpet.net/mbrubeck/2014/08/08/toy-layout-engine-1.html)和一次失败的[造轮子](https://github.com/zxc0328/litejs)经历。出于对jQuery进行源码分析和让旧日作品发挥余热的目的，我开始写这一系列博客。主要内容就是从头开始打造一个jQuery-like的基础库，并且配上相应的源码，也就是我们的Litejs。

我们要做的工作有以下这些：

+ 选择器和元素封装
+ DOM操作
+ Ajax请求
+ 事件系统
+ 触摸事件

分别对应这系列文章的四个部分。

好，让我们开启这段神奇的重复造轮子之旅吧！

<!--more-->
### 开发环境的搭建

我们使用ES6的模块语法编写模块，并使用[rollup.js](http://rollupjs.org/)打包，rollup是基于ES6模块的JS打包工具。我对于rollup的思考可以参考。除此之外我们还是用Babel和Flow。Flow是Facebook出品的JavaScript静态类型检查工具，我们用Babel来去除Flow的类型标记，以及编译其他的ES6特性代码。

// todo 测试环境搭建

`git checkout 1-1`

### jQuery的组织架构

简单的说，jQueryAPI由`jQuery`构造函数以及`jQuery`函数上的一系列静态API组成。`jQuery`构造函数就是我们平常使用的`$`函数，这个函数接收选择器字符串作为参数，返回jQuery对象实例。jQuery对象封装了原生的DOM对象，为其添加了一系列的属性和方法，以及链式调用的能力。

与此同时，因为JavaScript的函数也是对象，因此我们可以在函数上添加属性和方法。函数是对象，因此也可以访问对象原型的方法。`jQuery`函数上的静态方法，包括了回调、AJAX、事件、动画、工具函数等jQuery API。另一些API，比如DOM相关的API，则是绑定在`jQuery`构造函数实例化出来的对象上。

#### `lite`构造函数

下面就让我们开始编写我们的自制`jQuery`—`litejs`。让我们来编写我们的第一个函数，`lite`构造函数。

```
// The lite constructor function
var lite = function( selector, context ) {
  return new lite.prototype.init( selector, context );
};
```
这个函数很简单，调用了`lite.fn.init`函数，传入了选择器和上下文，并返回实例化的对象。让我们来看看真正干活的`lite.prototype.init`函数。


#### `lite.prototype.init`

我们之前说过，`lite`构造函数也是个对象，我们把一些静态方法都放在这个对象的原型上。其中就有`init`方法。我们先不急着看这个方法做了什么，我们在这里要做一个特殊的处理：

```
lite.prototype.init.prototype = lite.prototype;
```
很简单，因为`lite.prototype.init`方法返回的实例无法访问到`lite`函数的原型链，所以我们设置这个对象构造函数的原型是`lite.prototype`。

#### `lite.extend()`

`lite.extend()`其实就是我们常说的`mixin`函数

