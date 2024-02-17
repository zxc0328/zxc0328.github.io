---
title: "Table组件中slot内容的跨级传递"
date: 2017-09-19 15:00:09
tags: 
- Vue
categories: 
- Vue
---

在开发MUI的[Table组件](https://github.com/Muxi-Studio/MUI/tree/dev/src/components/table)时，我们遇到了一个问题。用户在顶层组件中嵌套的内容，需要被保存到组件的数据中，并且在表格内部渲染出来。

通常，Vue内嵌内容是使用slot进行渲染的。在父组件的模板中，在子组件的标签中嵌入模板，然后在子组件的内部，使用`<slot/>`标签进行渲染。但现在我们的需求是非父子组件的slot渲染，这就要求我们换一种思路去保存和调用slot。

要理解以下的内容，**请确保你阅读了[Render Functions & JSX](https://vuejs.org/v2/guide/render-function.html)，理解了Vue的VNode、render function、模板等概念以及这些概念之间的关系**。

<!-- more -->

### slot方案

首先要明确一个点，所谓的slot就指的是一个组件在声明时候的内嵌内容。Vue的模板都会被编译成VNode节点树，slot指的是一个VNode的`children`属性这个数组里包含的VNode节点集合，这些内容由组件声明时的内嵌内容编译而来。

我们可以通过`this.$slots.default`拿到默认的子VNode列表。如果内嵌内容上没有声明`name`属性，那这些内容都归属于`default`这个属性。

所以Slot其实就是一个VNode数组，我们可以把这个数组作为`prop`传入子节点进行渲染。

> `{{ vnode }}`这种语法会把vnode作为一个对象去序列化，这不是我们所期望的。所以我们需要用`v-bind`去传递VNodes的引用。

想要渲染slot，可以使用`render function`。之前讲过，slot其实就是VNode的children，所以我们在`render function`中`createElement`的时候把slot的引用作为children传入就可以了。

```
render(createElement) {
        return createElement('div', this.content)
    }
```

> 在组件初始化时给this.$slots赋值，然后在模板中使用slot渲染或许也是一种办法，但不一定行的通，也比较hacky。

但我们发现这样不能达到目的。VNode是Vue中对一个DOM节点的内部表示，VNode是有状态的，一个VNode同时只能渲染出一个DOM节点实例。也就是说一个VNode在渲染之后不能再次渲染，除非先把这个VNode从文档中移除，然后才可以再次渲染。

所以，因为我们的表格中的VNodes是会被每一个row复用的，现在这种用法只能渲染第一行的slot内容。

解决方案就是，用一个`deepClone`函数clone VNode，在每次渲染时初始化新的VNodes实例。

```
render(createElement) {
        return createElement('div', deepClone(this.content, createElement))
    }
```

### scopedSlots方案


这样似乎就可以解决问题了，但我们发现Table的自定义内容常常是一个按钮这样的可以交互的组件，会有事件绑定，如果我们要在子组件中给slot动态传入属性，这是办不到的。

所以slot就不能满足我们的需求了，更好的解决方案就是scopedSlots。

要了解什么是scopedSlots，我们首先将scopedSlots的模板：

```
<template scoped="prop">
  <div></div>
</template>
```

进行编译，结果是：

```
function anonymous() {
  with(this){return _c('div',{scopedSlots:_u([{key:"default",fn:function(prop){return [_c('div')]}}])})}
}
```

这种形式是我们之前没有遇到过的，scopedSlots被编译后，生成了一个函数，而且scopedSlot是被存放在VNode的`data`属性中，而不是在`children`中。

仔细观察这个函数，这个函数接收一个参数，然后返回一个VNode，这个VNode的属性是从这个参数中获取的。那scopedSlots的原理就很清楚了，**scopedSlots就是一个lazy evaluation的函数，在需要渲染的时候，接收scope对象，然后渲染。这样就可以达到一个类似动态作用域的效果**。

既然scopedSlots是一个函数，我们在render function里面只要调用这个函数，并且传入对应的scope对象作为参数就可以了：

```
render(createElement) {
        const prop = {
            index: this.id
        }
        return createElement('div', [
            this.content.call(this, prop)
        ])
    }
```

这种形式顺便解决了之前slot无法重复利用VNode的问题，**因为scopedSlots函数每次返回的都是一个新的VNode节点**。