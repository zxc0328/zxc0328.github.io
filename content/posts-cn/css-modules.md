---
title: "聊聊CSS Modules"
date: 2016-11-12 20:02:32
tags: 
- CSS
categories: 
- CSS
---


### CSS在工程化上的一些问题

关于React的CSS in JS，有一个[著名的talk](https://speakerdeck.com/vjeux/react-css-in-js)，由Facebook的工程师[vjeux](https://github.com/vjeux)带来。

里面最有名的一张slide是这样的：

![css in js](https://occc3ev3l.qnssl.com/zindex/Screen%20Shot%202016-11-13%20at%204.28.31%20PM.png)

里面列举了CSS的一些问题。其中，Dead Code Elimination，Minification，和Sharing Constants这些问题我们已经通过在我们的工作流中加入SASS和PostCSS这样的CSS预处理器解决了。

然而还有一些问题没有解决，比如全局命名空间。同一个document下的所有CSS的类名，都是在同一个“作用域”下的，因此我们常常要考虑如何避免命名冲突问题。现有的解决办法主要是靠BEM这样的命名惯例，或者是用多层CSS父子选择器来模拟命名空间。然而这样的办法对工程师有许多的限制。多级选择器有比较高的优先级，不容易维护。

<!-- more -->

### 解决全局作用域：Webpack css-loader

Webpack的css-loader首先做出了解决全局作用域的尝试。解决办法就是在写CSS类名时加入`:local(...)`这样的标记。

比如：

```
:local(.className) { background: red; }
:local .className { color: green; }
:local(.className .subClass) { color: green; }
:local .className .subClass :global(.global-class-name) { color: blue; }
```

会被转化为：

```
._23_aKvs-b8bW2Vg3fwHozO { background: red; }
._23_aKvs-b8bW2Vg3fwHozO { color: green; }
._23_aKvs-b8bW2Vg3fwHozO ._13LGdX8RMStbBE9w-t0gZ1 { color: green; }
._23_aKvs-b8bW2Vg3fwHozO ._13LGdX8RMStbBE9w-t0gZ1 .global-class-name { color: blue; }
```

这里的办法就是把CSS类名转化为hash字符串，这样就可以保证每个类名都是独一无二的，自然也就不用在意命名冲突的问题了。只要在类名在当前模块内不会相互冲突就可以了。

### CSS Modules

上述的办法，还是有一些不便。大多数情况下，比如在JavaScript中，变量都默认是局部变量。你想要声明一个全局变量，只能去全局作用域声明，或者把变量挂到local上（非严格模式下，不写var声明的是全局变量这种坑就不说了）。

Webpack的开发者之后将css-loader中的local变成了默认设定，于是CSS Modules这个规范就呼之欲出了。

[CSS Modules规范](https://github.com/css-modules/css-modules)。我们可以通过`css-loader?modules`这个参数来开启CSS Modules。

CSS Modules中的类名默认就是local的，如果你想要声明全局类名，可以加上`:global(...)`这个标记。


### Single Responsibility Principle

讲CSS Modules的下一个特性之前。我们先聊点其他的，我们知道设计模式中有一条叫做Single Responsibility Principle。

比如我们有一个button：

```
.button {
    display:inline-block;
    padding:2em;
    background-color:red;
}
```

与其把这些属性写在一个class里，我们可以把它拆分成多个单独的class：

```
.button {
    display:inline-block;
}
.button--large{
    padding:2em;
}
.button--warnning{
    background-color:red;
}
```

然后在HTML中组合使用就可以了。

```
<button class="button button--large button--warnning">
```

这样的好处是什么呢？我们的UI中，一个组件往往有很多不同的状态。如果我们将每一个class写成只专注于一个属性，做好一件事，那就可以用这些class组合成所有我们想要的不同状态的组件。相比给每个状态的组件写一个单独的class，代码要更优雅简洁一些。

比如我们想要一个small尺寸的普通button，只要加两个class:

```
.button {
    display:inline-block;
}
.button--small{
    padding:1em;
}
.button--large{
    padding:2em;
}
.button--normal{
    background-color:blue;
}
.button--warnning{
    background-color:red;
}
```

然后组合就可以了：

```
<button class="button button--small button--normal">
```

### CSS Classes Composing

要想实现上述的这种组合，可以使用SASS的Mixin，但Mixin主要是提供了源代码中的抽象，最后生成的代码，和手写不同状态class的代码量，是一样的。

CSS Modules提供的Classes Composing则刚好可以满足我们的需求。

比如我们想渲染一段文字：

```
.text{
  font-size: 20px;
  composes: red from "./common/color.css";
}
```

color.css里是这样的

```
.red{
	color: red;
}
```

最后渲染出的class是这样的

```
<div class="App-text-2AEnE_0 color-red-3ag3h_0"></div>
```

`composes`引入的类被作为一个单独的class引入，而不是和text类合在一起。



### CSS Modules和Vue工作流的整合

Vue-loader在v9.8.0之后加入了对CSS Modules的支持。

我们只要在`.vue`文件的`<style>`处加一个`module`就行

```
<style lang="sass" module>
.text{
  font-size: 20px;
  composes: red from "sass!./common/color.scss";
}
</style>
```

这里有一点要注意，就是`composes`引入的如果是需要预处理器处理的，要在前面加上预处理器的标记，比如SASS用户就加上`sass!`。

如果需要对CSS Modules进行一些配置（其实这个是对Webpack的css-loader的配置，所以配置时可以参考[css-loader的文档](https://github.com/webpack/css-loader)），写在vue-loader的配置的`cssModules`属性里即可

```
loader: 'vue',
options: {
	cssModules: {
		localIdentName: '[name]-[local]-[hash:base64:5]',
		camelCase: true
	}
}    
```

vue-loader会自动将一个`$style`属性注入到对应的Vue实例中。在模板中用class binding语法写就可以了。

```
<template>
  <div :class="$style.app">
    <div :class="$style.text">
      some text
    </div>
    <main-text></main-text>
  </div>
</template>
```

`$style`其实是一个原class名和处理之后class名的hash，像这样：

```
{
  app: "App-app-3cl75_0",
  text: "App-text-2AEnE_0 color-red-3ag3h_0"
}
```

我写一了一个简单的[DEMO仓库](https://github.com/zxc0328/css-modules-demo)，可以供参考。

### 结语

CSS Modules可以解决全局作用域和Class组合两个问题，加上SASS等预处理器，着实让我们在写CSS时的工程化程度大大提高了。

对于使用Vue的同学来说，vue-loader可以使CSS Modules可以轻松的整合到已有的工作流中。如果你正在使用Vue，可以试试使用CSS Modules。


### Links

+ [CSS Modules Welcome to the Future](http://glenmaddern.com/articles/css-modules)
+ [The case for CSS modules - Mark Dalgleish](https://www.youtube.com/watch?v=zR1lOuyQEt8&index=29&list=LLHdx8Qwo6uxw0fj3gQ5yeTg)
+ [React: CSS in JS by vjeux](https://speakerdeck.com/vjeux/react-css-in-js)
