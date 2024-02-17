---
title: "聊聊Chrome Devtools的Timeline"
date: 2016-11-29 20:15:49
tags: 
- Devtools
categories: 
- Devtools
---

最近在进行移动端Web页面的性能调优时用到了Chrome Devtools的Timeline。Timeline主要是针对浏览器渲染引擎的相关数据进行记录。关于Timeline的介绍可以看[官方文档](https://developers.google.com/web/tools/chrome-devtools/evaluate-performance/timeline-tool)。下面主要说说实际应用时如何来分析Timeline的数据，然后优化页面。

<!-- more -->

### Timeline的使用场景

Timeline的使用场景一个是在页面发生明显的卡顿时，比如CSS动画或者页面滚动时，用来分析卡顿的原因。还有一个场景便是进行页面渲染性能的评估，看页面渲染性能是否有优化的空间，或者通过截图观察页面渲染的过程，优化[Speed Index](https://sites.google.com/a/webpagetest.org/docs/using-webpagetest/metrics/speed-index)来提升用户体验。

### 关于渲染

我们知道，如果以每秒16-24帧的速度连续播放图片，那人就会产生动画的幻觉。在UI动画中，因为目前主流显示设备的刷新率达到了60Hz，我们追求的的帧率是60fps。那理想情况下，每一帧的时间应该是`1/60=16.67ms`。

又因为JavaScript是单线程事件驱动的模型。所以浏览器在一帧之内要完成JavaScript脚本的运行，然后计算，合成这帧画面，并渲染。

### Long Frame

所谓的Long Frame，也就是我们在Timeline的FPS中被标注为红色（如下图）的那些帧，便是用时超过正常标准的那些帧。

![longframe](https://occc3ev3l.qnssl.com/longframe.png)

*long frame*

Long Frame往往意味着卡顿，因为帧率低于60fps，甚至30fps。我们会通过查看这帧中的JS运行情况，浏览器渲染情况，以及浏览器用户触发事件的情况来分析卡顿的原因。

### Timeline中有价值的数据


#### JS调用栈


![jscallstack](https://occc3ev3l.qnssl.com/jsstack.png)
*JS Call Stack*

如果是JS运行时间过长，比如超过10ms，那我们就有必要检查这个JS的调用栈，可以通过查看Details一栏中的Bottom-up选项卡来看到自顶向下的调用信息。分析那个函数调用占用了过多的时间，并加以解决。



#### 浏览器渲染信息

浏览器渲染每一帧画面，需要经过这些阶段。

![render](https://occc3ev3l.qnssl.com/frame-full.jpg)
*render pipeline*

其中我们需要避免的主要就是Layout，如果触发了Layout的重新计算，那就意味着你的页面需要重绘，这是一个非常expensive的任务。如果我们在CSS动画时使用了GPU加速，那这些动画就不会造成重绘，而是作为一个单独的层在Compose阶段进行组合。

如果是浏览器渲染用时过多，我们就需要检查代码中是否触发了不必要的浏览器重绘。这方面的优化主要是针对CSS的优化，是一个专门的专题。一般的措施是采用CSS3 transform进行动画，开启硬件加速来避免浏览器重绘。

#### 事件输入

我们可以在Flame Chart的Interaction部分看到浏览器在这段时间触发的事件。

用户注册的事件，都有对应的回调函数。因此如果这个回调函数包含大量的逻辑，并且这个事件触发非常频繁的话，比如scroll或者touchmove，那我们就需要对事件触发的频率进行限制。在防止回调函数频繁触发层面，对应的办法有JS层面的节流函数。在限制事件注册的数量层面，我们可以使用事件代理，尽量避免在一个DOM元素上注册多个触发频繁的事件。

> 题外话：在调试移动端页面时，我发现Chrome在Timeline里标出了pinch这样的事件。说明浏览器的渲染层是支持这样的高级事件的。但在DOM的标准中没有找到这样的事件。需要大家用touch事件来合成。Chrome的这个做法不知道应该理解成浏览器面向未来的支持，还是浏览器私有的实现。当然，在移动端的操作系统层面，识别pinch应该不是一个难事。DOM的touch event中没有支持pinch，swipe等事件，这个问题值得思考。


#### 其他数据

Timeline中还有很多有价值的数据，比如CPU占有率，JS的Heap大小（可以简单的理解成JS的内存占用情况），DOM节点的数量，事件监听器的数量等等。