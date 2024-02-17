---
title:  "React 16 Fiber源码速览"
date: 2017-09-28 22:50:36
tags: 
- React
- Source code reading
categories: 
- React
- Source code reading
---

> 本文的写作有一部分没有完成，打算针对React 16.3再对本文进行修改，请大家留意

React 16在近期发布了。除了将备受争议的BSD+Patents协议改为MIT协议之外，React 16还带来了许多新特性，比如：


+ 允许在render函数中返回节点数组和字符串。

```
render() {
  // 再也不用在外面套一个父节点了
  return [
    // 别忘了加上key
    <li key="A">First item</li>,
    <li key="B">Second item</li>,
    <li key="C">Third item</li>,
  ];
}
```
+ 提供更好的错误处理。
+ 支持自定义DOM属性。

<!-- more -->

但最关键的一点还是：

![twitter](http://wx2.sinaimg.cn/large/64c45edcgy1fjzoreufb4j20xm0kcjwa.jpg)



没错，React 16是一次**重写**，在保持API不变的情况下，将核心架构改为了代号为Fiber的异步渲染架构。新架构带来了的变化有：

+ 体积减小

![file size](http://wx1.sinaimg.cn/large/64c45edcgy1fjzoretq6dj210q0bu776.jpg)

+ [服务端渲染速度大幅提升](https://medium.com/@aickin/whats-new-with-server-side-rendering-in-react-16-9b0d78585d67)

+ 更responsive的界面

### 一次预谋已久的重写

Fiber这个架构并不是突然冒出来的。Facebook的工程师在设计React之初就设想未来的UI渲染会是异步的。从`setState()`的设计和React内部的事务机制可以看出这点。

在去年，React的开发者[Andrew Clark](https://github.com/acdlite)在社区中放出了Fiber架构的一个[文档](https://github.com/acdlite/react-fiber-architecture)。描述了Fiber架构的基本信息。同时表示Facebook的工程师正在实现这个新架构。今年3月的React Conf 2017上，Lin Clark做了[A Cartoon Intro to Fiber](https://www.youtube.com/watch?v=ZCuYPiUIONs)这个分享，介绍了Fiber架构的工作原理。今年9月，Fiber架构随着React 16正式发布。Fiber架构的代码放在原来的React仓库之中，并且可以通过运行时的判断来切换新老架构，方便测试和部署。因此Fiber的开发是一个渐进的过程。[这个网站](http://isfiberreadyyet.com)实时展示了Fiber通过的测试用例，随着所有用例的通过，Fiber也正式发布了。有趣的是，在React 16发布之前，Fiber架构的React就已经运行在Facebook的产品中了。FB的工程师表示看到新架构在线上产品运行起来，是很激动人心的。具体的情况可以看这篇博客：[React 16: A look inside an API-compatible rewrite of our frontend UI library](https://code.facebook.com/posts/1716776591680069/react-16-a-look-inside-an-api-compatible-rewrite-of-our-frontend-ui-library/)。

### Fiber概念简介

本文的题目是React 16 Fiber源码速览，所以关注的主要是Fiber相关的代码。在分析源码之前，首先介绍一些基本概念。

> 推荐看上文中提到的[A Cartoon Intro to Fiber](https://www.youtube.com/watch?v=ZCuYPiUIONs)。这个分享比较系统和形象解释了Fiber架构的工程流程，并且使用了React源码中的术语。有助于理解Fiber的概念和源码。下文中的配图也来自这个分享。

#### reconciler VS renderer

![Reconciler VS Renderer](http://wx1.sinaimg.cn/large/64c45edcgy1fk0jck2eo4j20gk09a76l.jpg)

Reconciler就是我们所说的Virtul DOM，用于计算新老View的差异。React 16之前的reconciler叫Stack reconciler。Fiber是React的新reconciler。Renderer则是和平台相关的代码，负责将View的变化渲染到不同的平台上，DOM、Canvas、Native、VR、WebGL等等平台都有自己的renderer。我们可以看出reconciler是React的核心代码，是各个平台共用的。因此这次React的reconciler更新到Fiber架构是一次重量级的核心架构的更换。

由reconciler和renderer两个概念引出的是phase的概念。Phase指的是React组件渲染时的阶段。第一阶段是reconciliation，这一阶段做的是Fiber的update，然后产出的是effect list（可以想象成将老的View更新到新的状态所需要做的DOM操作的列表）。这一个阶段是没有副作用的，因此这个过程可以被打断，然后恢复执行。第二阶段是commit阶段。Reconciliation产生的effect list只有在commit之后才会生效，也就是真正应用到DOM中。这一阶段往往不会执行太长时间，因此是同步的，这样也避免了组件内视图层结构和DOM不一致。

#### Fiber是什么

React源码中的注释说：

> A Fiber is work on a Component that needs to be done or was done. There can be more than one per component.

简单的说，一个Fiber就是一个POJO对象，代表了组件上需要做的工作。一个React Element可以对应一个或多个Fiber节点。

在render函数中创建的React Element树在第一次渲染的时候会创建一颗结构一模一样的Fiber节点树。不同的React Element类型对应不同的Fiber节点类型。一个React Element的工作就由它对应的Fiber节点来负责。我们如果在console中打印React 16的组件实例，会发现有一个`_reactInternalFiber`属性指向它对应的Fiber实例。

虽然React的代码中其实没有明确的Virtul DOM概念，但Fiber和我们概念中的Virtul DOM树是等价的。

Fiber带来了一个给React的渲染带来了重要的变化。React内部有事务的概念。之前React渲染相关的事务是连续的，一旦开始就会run to completion。现在React的事务则是由一系列Fiber的更新组成的，因此React可以在多个帧中断断续续的更新Fiber，最后commit变化。

那为什么说一个React Element可以对应不止一个Fiber呢？因为Fiber在update的时候，会从原来的Fiber（我们称为current）clone出一个新的Fiber（我们称为alternate）。两个Fiber diff出的变化（side effect）记录在alternate上。所以一个组件在更新时最多会有两个Fiber与其对应，在更新结束后alternate会取代之前的current的成为新的current节点。

#### Fiber节点的数据结构

下面介绍Fiber类型的重要属性：

```
{
	tag: TypeOfWork, // fiber的类型，下一节会介绍
	alternate: Fiber|null, // 在fiber更新时克隆出的镜像fiber，对fiber的修改会标记在这个fiber上
	
	return: Fiber|null, // 指向fiber树中的父节点
	
	child: Fiber|null, // 指向第一个子节点
	sibling: Fiber|null, // 指向兄弟节点
	
	effectTag: TypeOfSideEffect, // side effect类型，下文会介绍
	nextEffect: Fiber | null, // 单链表结构，方便遍历fiber树上有副作用的节点
	pendingWorkPriority: PriorityLevel, // 标记子树上待更新任务的优先级

}
```

在实际的渲染过程中，Fiber节点构成了一颗树。这棵树在数据结构上是通过单链表的形式构成的，Fiber节点上的`chlid`和`sibling`属性分别指向了这个节点的第一个子节点和相邻的兄弟节点。这样就可以遍历整个Fiber树了。

Fiber树的图示如下：

![fiber tree](http://wx3.sinaimg.cn/large/64c45edcgy1fkc8x8n8x2j20fa0co428.jpg)

#### TypeOfWork

这是源码中的typeOfWork，代表React中不同类型的fiber节点。

```
{
  IndeterminateComponent: 0, // Before we know whether it is functional or class
  FunctionalComponent: 1,
  ClassComponent: 2,
  HostRoot: 3, // Root of a host tree. Could be nested inside another node.
  HostPortal: 4, // A subtree. Could be an entry point to a different renderer.
  HostComponent: 5,
  HostText: 6,
  CoroutineComponent: 7,
  CoroutineHandlerPhase: 8,
  YieldComponent: 9,
  Fragment: 10,
}s
```

对几个常用的类型作一下解释：

**ClassComponent**

就是应用层面的React组件。ClassComponent是一个继承自React.Component的类的实例。

**HostRoot**

ReactDOM.render()时的根节点。

**HostComponent**

React中最常见的抽象节点，是ClassComponent的组成部分。具体的实现取决于React运行的平台。在浏览器环境下就代表DOM节点，可以理解为所谓的虚拟DOM节点。HostComponent中的Host就代码这种组件的具体操作逻辑是由Host环境注入的。

#### TypeOfSideEffect

> 说一下这是以二进制位表示的。可以多个叠加。

```
{
  NoEffect: 0,          
  PerformedWork: 1,   
  Placement: 2, // 插入         
  Update: 4, // 更新           
  PlacementAndUpdate: 6, 
  Deletion: 8, // 删除   
  ContentReset: 16,  
  Callback: 32,      
  Err: 64,         
  Ref: 128,          
};
```

#### Priority

Priority指的是Fiber中一个work的优先级。这是React源码中的对Priority类型的定义：

```
{
  NoWork: 0, // No work is pending.
  SynchronousPriority: 1, // For controlled text inputs. Synchronous side-effects.
  TaskPriority: 2, // Completes at the end of the current tick.
  HighPriority: 3, // Interaction that needs to complete pretty soon to feel responsive.
  LowPriority: 4, // Data fetching, or result from updating stores.
  OffscreenPriority: 5, // Won't be visible but do the work in case it becomes visible.
}
```

我们可以把Priority分为同步和异步两个类别，同步优先级的任务会在当前帧完成，包括SynchronousPriority和TaskPriority。异步优先级的任务则可能在接下来的几个帧中被完成，包括HighPriority、LowPriority以及OffscreenPriority。

### React 16 Fiber源码目录结构

React库的入口、组件的基类`ReactComponent`、`ReactElement.createElement`函数等等所有平台公用的代码位于`src/isomorphic`下。

我们关注的Fiber代码位于`src/renderers/shared/fiber`下。我们先来看看`src/renderers`下面有什么：

![renderers](http://wx2.sinaimg.cn/large/64c45edcgy1fk1ka90rh0j20s109xacl.jpg)

可以看到`src/renderers`下的代码就是上文介绍的renderer，分dom、native、art等等平台。那我们再看看`src/renderers/shared`目录下有什么：

![renderers/shared](http://wx2.sinaimg.cn/large/64c45edcgy1fk1kj1kekxj20s10ezq6q.jpg)

`src/renderers/shared`其实就是reconciler相关的代码了。可以看到里面有fiber和stack新老两大reconciler（在笔者发文时，Stack reconciler已经完成了它的使命，相关的代码已经被移除了）。

最后让我们来看看`src/renderers/shared/fiber`下的代码：

![fiber](http://wx2.sinaimg.cn/large/64c45edcgy1fk1m7cifluj20sh0pk10v.jpg)

这些就是React fiber的核心代码了。Fiber节点的定义在`ReactFiber.js`中，Fiber的reconciler构造函数在`ReactFiberReconciler.js`中，Fiber节点的工作流程由`ReactFiberBeginWork.js`、`ReactFiberCommitWork.js`和`ReactFiberCompleteWork.js`组成。Fiber的子节点reconcile逻辑在`ReactChildFiber.js`中，`ReactFiberScheduler.js`则是调度相关的逻辑。接下来就让我们通过具体的场景，来分析React 16的源码吧！

### 阅读React源码须知

下面简单介绍一下在React源码中，起辅助作用的代码。以免大家在看源码时被这些代码所迷惑。

#### flow type

React使用了flow作为静态类型检查工具。所以React源码中都是带有类型声明的。这对熟悉Java或者C++这些静态类型语言的同学应该不陌生。**类型声明对于快速理解源码也是有很大帮助的**。

#### `if (__DEV__)`

React源码中常常有`if (__DEV__)`这样的代码，比如：

```
if (__DEV__) {
    warning(
      shouldUpdate !== undefined,
      '%s.shouldComponentUpdate(): Returned undefined instead of a ' +
        'boolean value. Make sure to return true or false.',
      getComponentName(workInProgress) || 'Unknown',
    );
  }
```

这些代码是为了更好的开发者体验而编写的。React中的友好的报错，render性能测试等等代码都是写在`if (__DEV__)`中的。在production build的时候，这些代码不会被打包。因此我们可以毫无顾虑的提供专为开发者服务的代码。React的最佳实践之一就是在开发时使用development build，在生产环境使用production build。

大家在刚开始接触源码时可以跳过`if (__DEV__)`中的代码，专注于理解核心的部分。

#### 源码阅读小技巧

如果读者想在阅读文本之后打算自己深入探索React源码，我可以给出一些阅读源码的小技巧。如果对于React中某个方法的调用过程感兴趣，可以在本地用create-react-app新建一下小demo项目，然后直接在node_modules中的react-dom.development.js和react.development.js两个文件里的对应方法**打断点**。这样在中断的时候就可以看到整个**调用栈**了，Chrome种可以通过点击调用栈切换到其中任何一帧的状态。如果发现调用过程中有自己感兴趣的函数，可以clone React的整个仓库，用编辑器对想要查看的函数进行**全局搜索**，找到那个函数的源码进行阅读。此外还有一个小tip，如果对某个特性的实现感兴趣，可以去**搜索React的pull request和issue列表**，说不定可以找到当初实现这个特性时候提的PR，PR中一般会写实现时的一些考虑。另外React源码的注释也是非常详尽的，有些已经等于简单的文档了，所以**仔细的阅读注释**也是理解源码的捷径之一。

#### 本文源码的时效性

React 16.0发布后，新架构的很多特性还没有完全开放，因此React这段时间还在一个积极的开发过程中，源码变动会比较大。本文是分析的源码是React v16.0的源码。大家在阅读时Github上的React源码时要注意，目前的master分支的React源码和本文中的源码会有一些差异。比如在本文发布时，React的目录结构就进行了调整，源码从src中转移到了packages目录下，按react、react-reconciler、react-dom等等NPM模块的方式划分。还有一些fiber的实现也在进行一些小的重构。比如在performWork相关的代码中加入performWorkOnRoot和renderRoot这几个函数，通过准确的命名让函数的作用更清晰。又比如Priority的概念直接被expirationTime取代了，workLoop中直接根据expirationTime来判断任务的执行时机。所以**推荐大家阅读master分支下的最新代码**，因为React 16在代码质量上的确还处于一个未完成的状态，随着进一步的开发，源码的可读性会更高。


### 确定源码分析的入口

### React 16组件源码分析：用户触发的`setState`开启的一次渲染

我们知道，React的渲染是由`setState`触发的，所以就让我们从`setState`入手，来分析React 16的组件渲染流程。

#### setState

`setState`方法是React基类上的一个方法。因此位于`src/isomorphic`下的`modern/class/ReactBaseClasses.js`：

![setState](http://wx2.sinaimg.cn/large/64c45edcgy1fk1m7bwt6vj20fd05pdgr.jpg)

我们看到`setState`调用了`this.updater.enqueueSetState`。updater是renderer在渲染的时候注入的对象，这个对象由reconciler提供。具体的逻辑可以看`ReactDOM.render`相关的代码，这里就不展开了。

#### enqueueSetState

既然updater是reconciler提供的，那我们就可以在fiber的代码中找到它。updater就位于`src/renderers/shared/fiberReactFiberCompleteWork.js`中。

![enqueueSetState](http://wx3.sinaimg.cn/large/64c45edcgy1fk1m7by675j20ch06tjsg.jpg)

这里只截取了一部分的updater代码，可以看到updater提供了`enqueueSetState`方法，这个方法首先从全局拿到React组件实例对应的fiber，然后拿到了fiber的优先级。最后调用了`addUpdate`向队列中推入需要更新的fiber，并调用`scheduleUpdate`触发调度器调度一次新的更新。

熟悉React源码的朋友应该知道，`setState`的流程到这里为止，和React 15的流程基本是一样的。从下面开始，我们就可以看到Fiber架构的不同之处了。

#### addUpdate

我们首先来看`addUpdate`函数，这个函数向Fiber的更新队列里加入一次更新：

```
function addUpdate(
  fiber: Fiber,
  partialState: PartialState<any, any> | null,
  callback: mixed,
  priorityLevel: PriorityLevel,
): void {
  const update = {
    priorityLevel,
    partialState,
    callback,
    isReplace: false,
    isForced: false,
    isTopLevelUnmount: false,
    next: null,
  };
  insertUpdate(fiber, update);
}
```

`addUpdate`函数组装了一个update，然后将fiber和update传入了insertUpdate函数中。我们先来看一下这里用到的两个类型，Update和UpdateQueue：

```
type UpdateQueue = {
  first: Update | null,
  last: Update | null,
  hasForceUpdate: boolean,
  callbackList: null | Array<Callback>,

  // Dev only
  isProcessing?: boolean,
};
```

```
type Update = {
  priorityLevel: PriorityLevel,
  partialState: PartialState<any, any>,
  callback: Callback | null,
  isReplace: boolean,
  isForced: boolean,
  isTopLevelUnmount: boolean,
  next: Update | null,
};
```

我们可以看到，UpdateQueue是一个单向链表，有first和last指针指向链表的头部和尾部。其中的每一个Update都有一个next属性指向下一个Update。这样的数据结构在React 16中是很常见的。

之前说到，在更新时，一个React element会有一个current fiber和一个alternate fiber。我们又把alternate fiber叫working in progress fiber。这两个fiber都有一个Update Queue。这两个Queue里面的item的引用是相同的，也就是所谓的persistent structure。区别在于，working in progress fiber会在更新完一个队列项之后将其从队列中移除。所以working in progress update queue永远是current queue的一个子集。在更新完成之后，working in progress fiber取代current fiber成为新的current fiber。如果更新中断（有更高优先级的更新插入），current fiber的update queue就可以作为备份，使得之前中断的更新可以重新开始。

再看`insertUpdate`，这个函数处理了将一个update插入到current queue和work-in-progress queue两个队列中的逻辑：

![insertUpdate](http://wx4.sinaimg.cn/large/64c45edcgy1fkbtqzlproj21eq210nik.jpg)


#### scheduleUpdate

看完了`addUpdate`相关的逻辑，我们再来看`scheduleUpdate`：

![scheduleUpdate](http://wx4.sinaimg.cn/large/64c45edcgy1fkbz4oej2tj21do3c91kx.jpg)


#### performWork

performWork的作用就是“刷新”待更新队列，执行待更新的事务：

![performWork](http://wx2.sinaimg.cn/large/64c45edcgy1fkc077e5vgj20g2244gun.jpg)

performWork的代码很长，其中很大一部分是错误处理代码，这些代码和React16中的新特性有关，官方博客的介绍如下：

> Previously, runtime errors during rendering could put React in a broken state, producing cryptic error messages and requiring a page refresh to recover. To address this problem, React 16 uses a more resilient error-handling strategy. By default, if an error is thrown inside a component’s render or lifecycle methods, the whole component tree is unmounted from the root. This prevents the display of corrupted data. However, it’s probably not the ideal user experience.
Instead of unmounting the whole app every time there’s an error, you can use error boundaries. Error boundaries are special components that capture errors inside their subtree and display a fallback UI in its place. Think of error boundaries like try-catch statements, but for React components.

我们需要关注的函数，一个是`workLoop`，这个函数是React更新pendingWork队列的主循环。一个是`scheduleDeferredCallback`，这个函数会在未来安排一次更新，来处理`workLoop`中没有做完的事务。


#### workLoop

*图片注释还需要打磨*

我们来看`workLoop`的代码：

![workLoop](http://wx4.sinaimg.cn/large/64c45edcgy1fkc0p3sx5fj21e93z97wh.jpg)

除了图中所注释的，`workLoop`中有一个值得注意的细节。我们看到，loop中首先判断nextUnitOfWork的优先级是不是高于或等于TaskPriority。如果不是，则进入另一个分支，这个分支和前一个在对nextUnitOfWork的处理上有着微妙的区别。之前在介绍Priority的时候我们说到过，TaskPriority以及更高的优先级属于同步优先级，这些更新会在nextTick之前完成。所以loop中的两个分支其实就是对同步和异步的任务做了不同的处理。两个分支的区别主要是第二个分支使用了deadline.timeRemaining()来判断是否还有时间继续处理任务。

在之前的分析中，我们没有关注deadline这个参数，workLoop中的这个参数是从performWork中传入的，而performWork中的deadline参数是由scheduleUpdateImpl传入的。scheduleUpdateImpl给同步优先级的任务的deadline参数传入的是null。这是符合常理的，因为同步优先级的任务会一定会在一次workLoop中执行完毕。scheduleUpdateImpl中的异步优先级的任务在scheduleDeferredCallback中处理，我们看这个函数的类型：

```
scheduleDeferredCallback(
    callback: (deadline: Deadline) => void,
  ): number | void,
```
deadline出现了！所以异步任务的deadline是在被scheduleDeferredCallback调用时传入的。

#### scheduleDeferredCallback

让我们来看看`scheduleDeferredCallback`这个函数。全局搜索一番，我们发现这个函数是在renderer初始化时被注入的。

React 16抽象出了一个叫`ReactFiberReconciler`的工厂函数。这个函数接收一个`HostConfig`类型的参数，返回一个Reconciler。每个renderer初始化时需要传入当前平台相关的配置，也就是一个`HostConfig`实例，才能拿到一个自定义的Reconciler。

> 这里说一点题外话，React抽象出这个工厂函数意味着React标准化了自定义Renderer的接口。Renderer通过`ReactFiberReconciler`这个API就可以将自定义Renderer接入FiberReconciler。[Making-a-custom-React-renderer](https://github.com/nitin42/Making-a-custom-React-renderer)就利用了这个函数来打造自定义Renderer。

`HostConfig`的类型签名是这样的：

```
export type HostConfig<T, P, I, TI, PI, C, CX, PL> = {
  getRootHostContext(rootContainerInstance: C): CX,
  getChildHostContext(parentHostContext: CX, type: T, instance: C): CX,
  getPublicInstance(instance: I | TI): PI,

  createInstance(
    type: T,
    props: P,
    rootContainerInstance: C,
    hostContext: CX,
    internalInstanceHandle: OpaqueHandle,
  ): I,
  appendInitialChild(parentInstance: I, child: I | TI): void,
  finalizeInitialChildren(
    parentInstance: I,
    type: T,
    props: P,
    rootContainerInstance: C,
  ): boolean,

  prepareUpdate(
    instance: I,
    type: T,
    oldProps: P,
    newProps: P,
    rootContainerInstance: C,
    hostContext: CX,
  ): null | PL,
  commitUpdate(
    instance: I,
    updatePayload: PL,
    type: T,
    oldProps: P,
    newProps: P,
    internalInstanceHandle: OpaqueHandle,
  ): void,
  commitMount(
    instance: I,
    type: T,
    newProps: P,
    internalInstanceHandle: OpaqueHandle,
  ): void,

  shouldSetTextContent(type: T, props: P): boolean,
  resetTextContent(instance: I): void,
  shouldDeprioritizeSubtree(type: T, props: P): boolean,

  createTextInstance(
    text: string,
    rootContainerInstance: C,
    hostContext: CX,
    internalInstanceHandle: OpaqueHandle,
  ): TI,
  commitTextUpdate(textInstance: TI, oldText: string, newText: string): void,

  appendChild(parentInstance: I, child: I | TI): void,
  appendChildToContainer(container: C, child: I | TI): void,
  insertBefore(parentInstance: I, child: I | TI, beforeChild: I | TI): void,
  insertInContainerBefore(
    container: C,
    child: I | TI,
    beforeChild: I | TI,
  ): void,
  removeChild(parentInstance: I, child: I | TI): void,
  removeChildFromContainer(container: C, child: I | TI): void,

  scheduleDeferredCallback(
    callback: (deadline: Deadline) => void,
  ): number | void,

  prepareForCommit(): void,
  resetAfterCommit(): void,

  // Optional hydration
  canHydrateInstance?: (instance: I | TI, type: T, props: P) => boolean,
  canHydrateTextInstance?: (instance: I | TI, text: string) => boolean,
  getNextHydratableSibling?: (instance: I | TI) => null | I | TI,
  getFirstHydratableChild?: (parentInstance: I | C) => null | I | TI,
  hydrateInstance?: (
    instance: I,
    type: T,
    props: P,
    rootContainerInstance: C,
    hostContext: CX,
    internalInstanceHandle: OpaqueHandle,
  ) => null | PL,
  hydrateTextInstance?: (
    textInstance: TI,
    text: string,
    internalInstanceHandle: OpaqueHandle,
  ) => boolean,
  didNotHydrateInstance?: (parentInstance: I | C, instance: I | TI) => void,
  didNotFindHydratableInstance?: (
    parentInstance: I | C,
    type: T,
    props: P,
  ) => void,
  didNotFindHydratableTextInstance?: (
    parentInstance: I | C,
    text: string,
  ) => void,

  useSyncScheduling?: boolean,
};
```

这里主要包括一些平台相关的代码，比如节点的操作（`insertBefore`和`appendChild`等等），还有一些配置项，比如`useSyncScheduling`。我们看到`scheduleDeferredCallback`就在其中。我们来看看renderer初始化的代码：

在React DOM的入口中：

```
scheduleDeferredCallback: ReactDOMFrameScheduling.rIC,
```

在React Native的入口中：

```
scheduleDeferredCallback: global.requestIdleCallback,
```
我们可以看到`scheduleDeferredCallback`的实现和平台相关。在Native环境下，它是React Native的js runtime提供的`global.requestIdleCallback`，在浏览器环境下，它是`ReactDOMFrameScheduling.rIC`。


#### Cooperative Scheduling && requestIdleCallback

`window.requestIdleCallback`的函数签名和`scheduleDeferredCallback`是一模一样的。requestIdleCallback的callback接收一个`IdleDeadline`类型的参数。这个`IdleDeadline`和React中的`deadline`都有一个`timeRemaining`方法。

requestIdleCallback的W3C规范叫Cooperative Scheduling of Background Tasks。React官方在介绍fiber时也提到了Cooperative Scheduling这种技术。从源码来看，React主要利用了浏览器提供的requestIdleCallback API来实现这一特性。

相比于利用setTimout这样的API实现task scheduling，requestIdleCallback带来的Cooperative Scheduling让开发者让浏览器在空闲时间调用callback，并且在callback中可以获取到当前帧剩余的时间。利用这个信息我们可以合理的安排当前帧需要做的工作，如果工作太多而时间不够，就再调用requestIdleCallback来做剩余的工作。

requestIdleCallback的回调具体执行的时间点是在一帧开始，JavaScript执行完，浏览器执行渲染流程之后，到这帧结束之前。图示如下：

![idleCallback](http://wx1.sinaimg.cn/large/64c45edcgy1fkc7kiv42qj20kh03vq3a.jpg)

`deadline`中的`timeRemaining`的最大值是50ms，以免浏览器长期空闲时，callback的任务一直执行，使得UI不能及时响应用户输入。


#### ReactDOMFrameScheduling.rIC

`ReactDOMFrameScheduling.rIC`的逻辑是，如果浏览器实现了requestIdleCallback，就返回原生API。如果没有实现，就返回一个polyfill。这个polyfill的实现非常有趣，可以学到很多有意思的黑科技。

我们来看看`ReactDOMFrameScheduling.rIC`的实现：



虽然Chrome和Firefox都已经实现了requestIdleCallback，但某些浏览器还是需要polyfill，所以我们重点关注一下requestIdleCallback的polyfill的实现。

预估一个比较低的frame rate。requestAnimationFrame获取一帧开始，时间戳，触发一个message事件，postMessage在layout paint和composite之后被调用。deadline通过frame rate - rafTime可以得到

```
| frame start time                                      deadline |
[requestAnimationFrame] [layout] [paint] [composite] [postMessage]
```

通过requestAnimationFrame直接的时间差获取过去两帧的准确frame rate，动态调整当前帧的frame rate。

#### 默认优先级

既然一次更新是同步还是异步是由优先级决定的，那我们在用户代码中通过`setState`来schedule的一次update的优先级是多少呢？

我们回顾一下`enqueueSetState`的代码：

![enqueueSetState](http://wx3.sinaimg.cn/large/64c45edcgy1fk1m7by675j20ch06tjsg.jpg)

`addUpdate`和`scheduleUpdate`的`priorityLevel`是通过`getPriorityContext(fiber, false)`获取的。

我们来看看`getPriorityContext`的实现：

![getPriorityContext](http://wx3.sinaimg.cn/large/64c45edcgy1fkd4vrvbycj20ox0ian05.jpg)

所以我们得出了一个重要的结论。在React 16中，**异步渲染默认是关闭的**。用户代码的优先级是同步的。

#### performUnitOfWork


讲完了deadline对象的由来，我们回到workLoop，看看React是的reconcilation是如何进行的。我们可以看到首先被调用的是performUnitOfWork，这个函数做的就是所谓的reconcilation阶段的工作了。然后React将调用commitAllWork进入commit阶段，将reconcilation结果真正应用到DOM中。

![performUnitOfWork](http://wx4.sinaimg.cn/large/64c45edcgy1fkc84mx1vij20fa0gq7af.jpg)

React 16保持了之前版本的事务风格，一个“work”会被分解为begin和complete两个阶段来完成。我们先关注beginWork

#### beginWork

![beginWOrk](http://wx2.sinaimg.cn/large/64c45edcgy1fk6jdy25scj20wq1kvqdw.jpg)

`beginWork`函数根据fiber节点不同的tag，调用对应的update方法。可以说是一个入口函数。真正的逻辑要看update开头的这一些了函数。

#### updateClassComponent && updateHostComponent

上一节中讲到，`beginWork`中不同tag的元素有不同的update系列方法，我们重点关注的是对ClassComponent和HostComponent两种component的更新方法。ClassComponent对应的是React组件实例，HostComponent对应的是一个视图层节点，在浏览器环境中就等于DOM节点。

我们先关注`updateClassComponent`函数：

![updateClassComponent](http://wx2.sinaimg.cn/large/64c45edcgy1fkq4dx9ljkj20ox0jyn2j.jpg)

`updateHostComponent`这里就不再详细分析了。因此HostComponent没有生命周期钩子需要处理，这个函数主要做的就是调用`reconcileChildren`对子节点进行diff。


#### reconcileChildren

`reconcileChildren`实现的就是江湖上广为流传的Virtul DOM diff。这年头人人都看过一两个Virtul DOM diff的实现，那React 16的diff是如何实现的呢？

![reconcileChildren](http://wx4.sinaimg.cn/large/64c45edcgy1fkurhf1dgwj20p60sy44v.jpg)

`reconcileChildren`这个函数里调用了三个功能相似的函数：`mountChildFibersInPlace`、`reconcileChildFibers`和`reconcileChildFibersInPlace`。在源码中我们发现，这三个函数其实是同一个函数，通过传入不同的参数“重载”而来的。

```
exports.reconcileChildFibers = ChildReconciler(true, true);

exports.reconcileChildFibersInPlace = ChildReconciler(false, true);

exports.mountChildFibersInPlace = ChildReconciler(false, false);
```

ChildReconciler是一个工厂函数，它接收shouldClone, shouldTrackSideEffects两个参数。reconcileChildFibers函数的目的是产出effect list，所以shouldClone, shouldTrackSideEffects两个参数都是true。mountChildFibersInPlace是组件初始化时用的，所以不用clone fiber来diff，也不用产出effect list。reconcileChildFibersInPlace是在之前reconcile被中断的fiber树上继续工作，因此shouldClone参数为false。

ChildReconciler内部有很多helper函数，最终返回的函数叫reconcileChildFibers，这个函数实现了对子fiber节点的reconciliation。下面我们关注reconcileChildFibers函数的实现。


#### reconcileChildFibers

图的注释：

+ 总的，这个函数根据newChild的类型调用不同的方法。newChild可能是一个元素，也可能是一个数组（React16新特性）
+ 如果是reconcile单个元素，以reconcileSingleElement为例比较key和type，如果相同，复用fiber，删除多余的元素（currentFirstChild的sibling），如果不同，调用createFiberFromElement，返回新创建的。
+ 如果是string，reconcileSingleTextNode
+ 如果是array，reconcileChildrenArray
+ 如果是空，deleteRemainingChildren删除老的子元素

React的reconcile算法采用的是层次遍历，这种算法是建立在一个节点的插入、删除、移动等操作都是在**节点树的同一层级中进行**这个假设下的。所以reconcile算法的核心就是如何diff两个子节点数组。


#### reconcileChildrenArray

React16的diff算法采用和来自社区的两端同时比较法同样结构的算法。

> 关于diff算法演化历史可以看司徒正美的[这篇博客](https://segmentfault.com/a/1190000011235844)

因为fiber树是单链表结构，没有子节点数组这样的数据结构。也就没有可以供两端同时比较的尾部游标。所以React的这个算法是一个简化的两端比较法，只从头部开始比较。

下面我们来看一下代码：

图片

从头部遍历。第一次遍历新数组，对上了，新老index都++，比较新老数组哪些元素是一样的，（通过updateSlot，比较key），如果是同样的就update。第一次遍历玩了，如果新数组遍历完了，那就可以把老数组中剩余的fiber删除了。

如果老数组完了新数组还没完，那就把新数组剩下的都插入。

如果这些情况都不是，就把所有老数组元素按key放map里，然后遍历新数组，插入老数组的元素，这是移动的情况。

最后再删除没有被上述情况涉及的元素（也就是老数组中有新数组中无的元素，上面的删除只是fast path，特殊情况）

#### completeUnitOfWork

> 注：这里effect list链表插入的想法只是猜测，需要进一步确认。
 
`completeUnitOfWork`是complete阶段的入口。complete阶段的作用就是在一个节点diff完成之后，对它进行一些收尾工作，主要是更新props和调用生命周期方法等等。`completeUnitOfWork`主要的逻辑是调用`completeWork`完成收尾，然后将当前子树的effect list插入到HostRoot的effect list中。具体的让我们来看代码：

![completeUnitOfWork](http://wx4.sinaimg.cn/large/64c45edcgy1fks860npkqj20p41foalg.jpg)

#### completeWork

complete阶段主要工作都是在`completeWork`中完成的。这个函数很长，需要仔细梳理。

![completeWork](http://wx3.sinaimg.cn/large/64c45edcgy1fkurhf2r9jj20ou3c9x1u.jpg)

可见completeWork主要是完成reconciliation阶段的扫尾工作，重点是对HostComponent的props进行diff，并标记更新。

到这里，我们就讲完了reconciliation阶段。这个阶段主要负责产出effect list。所以可以说reconcile的过程相当于是一个纯函数，输入是fiber节点，输出一个effect list。side-effects是在commit阶段被应用到UI中的，这样就将side-effects从reconciliation中隔离开了。因为纯函数的可预测性，让我们可以随时中断reconciliation阶段的执行，而不用担心side-effects给让组件状态和实际UI产生不一致。

commit这个阶段有点像Git的commit概念。在缓冲区中的代码改动只有在commit之后才会被添加到Git的Object store中。

下面我们就来关注commit阶段的实现。看看effect list是如何被“提交”到UI中的。


#### commitAllWork

reconciliation阶段结束之后，我们需要将effect list更新到UI中。这就是commit节点的工作。commit阶段的入口是`commitAllWork`函数，我们来看看它的实现：

![commitAllWork](http://wx1.sinaimg.cn/large/64c45edcgy1fkorwi0836j20oz2f71b8.jpg)

这里需要注意的是，React 16中的生命周期方法是在reconciliation和commit两个阶段中被调用的，commit阶段的`commitAllLifeCycles`函数中的生命周期方法包括`componentDidMount`、`componentDidUpdate`和`componentWillUnmount`三个。

![lifeCycle](http://wx2.sinaimg.cn/large/64c45edcgy1fknn7fnubmj20eb08wjt7.jpg)

#### reconciliation+commit流程总结


经过上述对reconciliation和commit两个阶段的源码分析，是不是觉得有些混乱？我总结了一张reconciliation+commit过程中的函数调用图，希望可以帮助你理清这两个阶段的函数调用流程。从图中我们可以看出，`workLoop`中调用了`performUnitOfWork`和`commitAllWork`，分别作为reconciliation和commit两个阶段的入口。`performUnitOfWork`中又分为begin和complete两个阶段来处理。

![callStack](http://wx4.sinaimg.cn/large/64c45edcgy1fkota00kyhj20o402ydg3.jpg)

### 展望&&结语

#### 潜伏的大招——异步渲染

在上文中，我们知道，React 16中默认没有开启异步渲染。用户的setState都是和React 15一样，在一个tick内完成的。fiber可以解决的问题，比如将优先级低的任务分散在多个帧中完成，在每一帧中留足够的时间给响应用户输入和渲染这样优先级高的任务。在默认不开启异步渲染的情况下，是不能做到的。因此我们期待未来版本的React可以开启这个杀手特性。

我们在阅读源码的过程中，看到了一些没有被文档记录的组件类型，比如CoroutineComponent和YieldComponent。这也许意味着未来React会把渲染的时机掌控权交给用户。我们可以定义一个CoroutineComponent，在reconcile完成后交出控制权给用户。由用户主动调用commit来让组件继续渲染。因为React将组件的渲染分为reconcile和commit两个阶段，reconcile又是没有副作用的，由多个院子操作组成。因此这样的设想是完全可行的。以上只是笔者的推测，丢一个A Clark的链接。

#### React 16的设计给前端框架带来的思考

这次React更新核心架构，让我们看到Facebook的工程师再次用技术推进了用户体验的极限。淘宝FED的口号是用技术为体验提供无限可能，笔者觉得这句话用来形容React也是很合适的。在React上，我们看到了一些借鉴自操作系统中的设计。Fiber可以被比作是一个轻量级线程。有自己的数据，也有优先级的分别。React的作用就是调度fiber，使得优先级高的任务优先执行，同时也保证低优先级的任务会在未来一段时间执行完毕。在diff算法的设计上，React借鉴了社区的经验，这是对社区的一种认可。
