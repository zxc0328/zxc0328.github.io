---
title: "如何阅读大型前端开源项目的源码"
date: 2018-05-01 15:01:50
tags: 
- Source code reading
categories: 
- Source code reading
---

目前网上有很多「XX源码分析」这样的文章，不过这些文章分析源码的范围有限，有时候讲的内容不是读者最关心的。同时我也注意到，源码是在不断更新的，文章里写的源码往往已经过时了。因为这些问题，很多同学都喜欢自己看源码，自己动手，丰衣足食。

这篇文章主要讲的是阅读大型的前端开源项目比如 React、Vue、Webpack、Babel 的源码时的一些技巧。目的是让大家在遇到需要阅读源码才能解决的问题时，可以更快的定位到自己想看的代码。授人以鱼不如授人以渔，希望大家可以通过这篇博客，了解到阅读大型前端项目源码时的切入点。在之后遇到好奇的问题时，可以自己去探索。

<!-- more -->

### 问题驱动——不要为了看源码而看源码

首先我们要明确一点，看源码的目的是什么？

我个人的意见是，看源码是为了解决问题。开源项目的源代码并没有什么非常特殊的地方，也都是普通的代码。这些代码的数量级一般都挺大，如果想是从源码中学到东西，直接浏览整个 Codebase 无疑是大海捞针。

但如果是带着问题去看源码，比如想了解一下 React 的合成事件系统的原理，想了解 React 的 setState 前后发生了什么，或者想了解 Webpack 插件系统的原理。也有可能是遇到了一个 bug，怀疑是框架/工具的问题。在这样的情况下，带着一个具体的目标去看源码，就会有的放矢。

### 看最新版的源码

之前看到一种说法，看源码要从项目的第一个 commit 开始看。如果是为了解决前文中对框架/工具产生的困惑，那自然要看当前项目中用到的框架/工具的版本。

如果是为了学习源码，我也建议看最新的源码。因为一个项目是在不断迭代和重构的。不同版本之间可能是一次完全的重写。比如 Vue 2.x 和 React 16。重构导致了代码架构上的一些变化，Vue 2.x 引入了 Vritual DOM，Pull + Push 的数据变化检测方式让整个代码的结构变的更清晰了，所以 2.x 的代码其实比 1.x 的更容易阅读。React 16 重写了 Reconciler，引入了 fiber 这个概念，整个代码仓库结构也更清晰，所以更推荐阅读。

### 前置条件

看源码怎么看，当然不能一把梭了。

![](https://raw.githubusercontent.com/zxc0328/for-picgo/master/64c45edcgy1fqw0buj08wj20go0aztie.jpg)

看源码之前需要对项目的**原理**有一个基本的了解。所谓原理就是，这个项目有哪些组成部分，为了达到最终的产出，要经过哪几步流程。这些流程里，业界主流的方案有哪几种。

比如前端 View 层框架，要渲染出 UI，组件要经过 mount、 render 等等步骤。数据驱动的前端框架，在 mounted 之后，就会进入一个循环，当用户交互触发组件数据变化时，会更新 UI。其中数据的检测方式又有分 Push 和 Pull 两种方案。渲染 UI 可以是全量的字符串模板替换，也可以是基于 Virtual DOM 的差量 DOM 更新。


又比如前端的一些工具，Webpack 和 Babel 这些工具都是基于插件的。基本的工作流程就是读取文件，解析代码成 AST，调用插件去转换 AST，最后生成代码。要了解 Webpack 的原理，就要知道 Webpack 基于一个叫 [tapable](https://github.com/webpack/tapable) 的模块系统。

那我们要如何了解这些呢？要了解这些，可以去各大网站和博客上的《XXX源码解析》系列。通过这些文章，我们可以对我们要看的框架/工具的原理有一个大致的了解。

### 本地build

不过最终我们还是要直接看源码。笔者真正看源码的第一步就是把项目的代码仓库 clone 到本地。然后按项目 README 上的构建指南，在本地 build 一下。

如果是前端框架，我们可以在 HTML 中里直接引入本地 build 出的 umd bundle（记得用 development build，不然会把代码压缩，可读性差），然后写一个简单的 demo，demo 里引入本地的 build。如果是基于 Nodejs 的工具，我们可以用 npm link 把这个工具的命令 link 到本地。也可以直接看项目的 package.json 的入口文件，直接用 node 运行那个文件。 

这里要强调一下，大型的开源项目一般都会有一个 **Contribution Guide**，目的是让想贡献代码的开发者更快上手。里面就有讲怎么在本地构建代码。

以 React 为例，React 的 [Contributing Guide](https://reactjs.org/docs/how-to-contribute.html) 里就 Development Workflow 这一节。里面有这么一段话：

> The easiest way to try your changes is to run yarn build core,dom --type=UMD and then open fixtures/packaging/babel-standalone/dev.html. This file already uses react.development.js from the build folder so it will pick up your changes.

所以 React 仓库中的 fixtures/packaging/babel-standalone/dev.html 就是一个方便的 demo 页。我们可以在这个页面快速查看我们在本地对代码的改动。

你可以尝试着在项目的入口文件中加入一句 log，看看是不是可以在控制台/终端看到这句 log。如果可以的话，恭喜你，你现在可以随便把玩这个项目了！

### 理清目录结构

在看具体的代码之前，我们需要理清项目的目录结构，这样我们才能更快的知道在哪里地方找相关功能的代码。

我们看看 React 的目录结构。React 是一个 monorepo。也就是一个仓库里包含了多个子仓库。我们在 packages 目录下可以看到很多单独的 package：

![](https://raw.githubusercontent.com/zxc0328/for-picgo/master/64c45edcgy1fqsdkkvpjtj21k00wmaka.jpg)

在 React 16 之后，React 的代码分为 React Core，Renderer 和 Reconciler 三部分。这是因为 React 的设计让我们可以把负责映射数据到 UI 的 Reconciler 以及负责渲染 Vritual DOM 到各个终端的 Renderer 和 React Core 分开。React Core 包含了 React 的类定义和一些顶级 API。大部分的渲染和 View 层 diff 的逻辑都在 Reconciler 和 Renderer 中。

Babel 也是一个 monorepo。Babel 的核心代码是 babel-core 这个 package，Babel 开放了接口，让我们可以自定义 Visitor，在AST转换时被调用。所以 Babel 的仓库中还包括了很多插件，真正实现语法转换的其实是这些插件，而不是 babel-core 本身。

![](https://raw.githubusercontent.com/zxc0328/for-picgo/master/64c45edcgy1fqv1g5muerj21jm14kqe3.jpg)

Vuejs 的代码比较典型，核心代码在 src 目录下，按功能模块划分。因为 Vue 也支持多平台渲染，所以把平台相关的代码都放到了 platform 文件夹下，core 文件夹中是 Vue 的核心代码，compiler 是 Vue 的模板编译器，把 HTML 风格的模板编译为 render function。

![](https://raw.githubusercontent.com/zxc0328/for-picgo/master/64c45edcgy1fqshma4atzj21js0ietdc.jpg)


Webpack 和 Babel 一样，可以说都是基于插件的系统。Webpack 的主要源码在 lib 目录下，里面的 webpack.js 就是入口文件。

上面说了四个项目的目录结构，那我们遇到一个新的开源项目，应该怎么了解它的目录结构呢？

如果这个项目是一个 monorepo，首先我们要找到核心的那个 package，然后看里面的代码。

不是 monorepo 的话，一般来说，如果这个项目是一个 CLI 的工具，那 bin 目录下放的就是命令行界面相关的入口文件，lib 或者 src 下面就是工具的核心代码。如果这个项目是一个前端 View 层框架，那目录结构就和 Vue 类似。

作为验证，大家可以看一下打包工具 [parcel](https://github.com/parcel-bundler/parcel) 和前端 View 层库 [moon](https://github.com/kbrsh/moon) 的目录结构。目录结构这个东西往往是大同小异，多看几个项目就熟悉了。

### debugger && 全局搜索大法

运行了本地的 build，了解了目录结构，接下来我们就可以开始看源码了！之前说了，我们要以问题驱动，下面我就以 React 调用 setState 前后发生了什么这个问题作为例子。

我们可以在 setState 的地方打一个断点。首先我们要找到 setState 在什么地方。这个时候之前的准备工作就派上用处了。我们知道 React 的共有 API 在 react 这个 package 下面。我们就在那个 package 里面全局搜索。我们发现这个 API 定义在 src/ReactBaseClasses.js 这个文件里。

于是我们就在这里打一个断点：

```
Component.prototype.setState = function(partialState, callback) {
  invariant(
    typeof partialState === 'object' ||
      typeof partialState === 'function' ||
      partialState == null,
    'setState(...): takes an object of state variables to update or a ' +
      'function which returns an object of state variables.',
  );
  debugger;
  this.updater.enqueueSetState(this, partialState, callback, 'setState');
};
```

然后运行本地 React build 的 demo 页面，让组件触发 setState，我们就可以在 Devtool 里看到断点了。

我们走进 this.updater.enqueueSetState 这个调用，就来到了 ReactFiberClassComponent 这个函数中的 enqueueSetState，这里调用了 enqueueUpdate 和 scheduleWork 两个函数，如果要深入 setState 之后的流程，我们只需要再点击 ![](https://raw.githubusercontent.com/zxc0328/for-picgo/master/64c45edcgy1fqx8qcid0aj200i00q053.jpg) 走进这两个函数里看具体的代码就可以了。


如果想看 setState 之前发生了什么，我们只需要看 Devtool 右边的调用栈：

![](https://raw.githubusercontent.com/zxc0328/for-picgo/master/64c45edcgy1fqvtw441ytj20c60ukae9.jpg)

点击每一个 frame 就可以跳到对应的函数中，并且恢复当时的上下文。

结合一步一步的代码调试，我们可以看到框架的函数调用栈。对于每个重要的函数，我们可以在仓库里搜索到源码，进一步研究。

Node 工具的调试方法也是相似的，我们可以在运行 node 命令时加上 --inspect 参数。具体可以看[ *Debugging Node.js with Chrome DevTools*](https://medium.com/@paul_irish/debugging-node-js-nightlies-with-chrome-devtools-7c4a1b95ae27) 这篇博客。

其实大家都知道单步调试这种办法，但在哪里打断点才是最关键的。我们在熟悉框架的原理之后，就可以在框架的**关键链路**上打断点，比如前端 View 层框架的声明周期钩子和 render 方法，Node 工具的插件函数，这些代码都是框架运行的必经之地，是不错的切入点。

如果是为了了解一个特定的问题，大家可以直接在自己觉得有问题的地方打断点。然后把源码运行起来，想办法让代码运行到那个地方。我们在断点可以看到局部变量等等信息，有助于定位问题。

### 来自开发团队的资源

其实开源项目的开发团队也都致力于让更多的人参与到项目中来，降低项目的门槛。所以我们在线上其实可以找到很多来自开发团队的资源。这些资源可以帮助我们去理解项目的原理。

#### 关注核心开发者

每个项目都有一些核心开发者，比如 React 的 [Dan Abramov](https://twitter.com/dan_abramov), [Andrew Clark](https://twitter.com/acdlite) 和 [Sebastian Markbåge](https://twitter.com/sebmarkbage)。Webpack 的 [Tobias Koppers](https://twitter.com/wSokra) 和 [Sean Larkin](https://twitter.com/TheLarkInn)。Vue 的 [Evan You]()。我们可以在 Twitter 上关注他们，了解项目的动态。


#### 关注官方博客和演讲视频

如果我们关注了上面的核心开发者，就会发现他们时常会发布一些和源码/项目原理有关的博客或者视频。

React 的官方博客最近就有很多和项目开发有关的博客。

+ [Behind the Scenes: Improving the Repository Infrastructure](https://reactjs.org/blog/2017/12/15/improving-the-repository-infrastructure.html) 这篇介绍的是 React 项目仓库的基础设施。
+ [Sneak Peek: Beyond React 16](https://reactjs.org/blog/2018/03/01/sneak-peek-beyond-react-16.html)

Andrew Clark 一开始就写了一篇介绍 fiber 架构的[文档](https://github.com/acdlite/react-fiber-architecture)。
Dan Abramov 最近在 JSConf 上对 React 未来的一些新特性的介绍 - [Beyond React 16](https://www.youtube.com/watch?v=v6iR3Zk4oDY)。React 博客中的 *Sneak Peek: Beyond React 16* 也是对这次 Talk 的介绍。

Evan You [介绍前端框架数据变化侦测原理的 Talk](https://www.youtube.com/watch?v=r4pNEdIt_l4)。Vue 文档中也有 [Reactivity in Depth](https://vuejs.org/v2/guide/reactivity.html) 这样的介绍原理的章节。

Sean Larkin 的 [Everything is a plugin! Mastering webpack from the inside out](https://www.youtube.com/watch?v=4tQiJaFzuJ8) 介绍了 Webpack 的核心组件 Tapable。

James Kyle 的 [How to Build a Compiler](https://www.youtube.com/watch?v=Tar4WgAfMr4) 可以让我们了解 Babel 转译代码的基本流程。


### 写在最后


本文最核心的观点就是，看源码的目的是为了解决问题。我们鼓励大家在本地把大型项目的源码跑起来，自己随意把玩，研究。因为源码也是普通的代码，并没有太多门槛。唯一的门槛可能就来源于开源项目作者和普通开发者之间的信息不对称，普通开发者对项目的原理和目录结构不够了解。

我们可以从开发者那里获取资源，同时也可以多阅读社区里的源码分析文章，这些都有助于我们理解项目的原理，为后续的源码分析打下基础。
