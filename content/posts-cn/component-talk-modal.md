---
title: "聊聊UI组件设计-Modal"
date: 2017-08-14 19:23:20
hiddenInHomeList : true
tags: 
- Component
categories: 
- Component
---

在人机交互中，有一个概念叫做[Modality](https://en.wikipedia.org/wiki/Modality_(human–computer_interaction))，中文叫模态。模态，顾名思义，就是模拟。计算机可以模拟人通过各种通道接收的信息，比如视觉、听觉、触觉等等通道。视觉就通过显示器输出，听觉通过音响、触觉通过振动。同理，人也可以模拟计算机接收到的电信号，人可以通过键盘、触摸板等待设备来模拟0/1信号。

模态可以是但通道的，也可以是多通道的（比如玩游戏时有声音、视觉、和振动反馈）。今天我们要将的计算机软件中的Modal组件，就是计算机向人建立的单通道信息交互方式。

<!-- more -->

### Modal在交互设计中的作用

具体到产品的交互设计中（交互设计和人机交互不是一回事，准确的说两者有交叉），Modal这个组件是很常用的，那么Modal在交互上的意义是什么呢？

Modal的弹出，其实就是计算机和人之间建立了一个信息传递的通道，这个通道是独占的，在关闭这个通道之前，不能进行其他的交互。计算机程序建立这个通道，为的是传递一些信息。但传递信息有很多的方式，为什么要使用独占通道的Modal来传达呢？

因为Modal传递的信息，是为了让用户**提供关键信息**，这个信息的回复（Modal的输入）可以是是一个true or false的选择，也可以是较为复杂的数据结构。Modal是一个浮层，所以用户在Modal弹出时，不能再点击应用的其他部分。用户必须要做出决定，是输入信息，或者取消（关闭Modal）。因此，Modal会中断用户当前的工作流。

### 实现：Modal组件的特点


+ 全局一般只能显示一个Modal，因为Modal显示时会需要用户做出决定后再消失（点击取消也是一种决定，即为对操作的否定）。
+ Modal一般提供确定和取消两个按钮。
+ Modal中标题和底部按钮直接的内容，一般是可以自由组合的。一种特殊情况是Modal中组合了input，那这种Modal类型被称为prompt。
+ Modal如果涉及异步的操作，则需要有一个confirm loading状态。这个状态下用户不能再次点击confirm。

这里推荐一篇很好的总结文章[覆盖层设计(上)-对话框&浮层](http://www.ui.cn/detail/224467.html)来自网易UEDC。系统的总结了交互设计中浮层相关的设计。

### 不同UI库的Modal设计

下面讲讲正题，Modal作为一个Web前端组件的设计方式。

根据Modal在前端代码中的调用方式，可以分为声明式和命令式两种。Modal的声明式使用是指在前端模板中声明Modal。命令式则是在前端代码中调用一个函数，来显式的调用Modal。

#### 声明式

Ant Design中的[Modal](https://ant.design/components/modal/)是典型的声明式组件。Modal被声明在模板中，在父组件初始化之时便存在了。

```
render() {
    return (
      <div>
        <Modal
          title="Basic Modal"
          visible={this.state.visible}
          onOk={this.handleOk}
          onCancel={this.handleCancel}
        >
          <p>Some contents...</p>
          <p>Some contents...</p>
          <p>Some contents...</p>
        </Modal>
      </div>
    );
  }
```


Modal出现时只是visible这个flag被设置为`true`。而不是在那个时候初始化Modal组件。要在Modal中组合内嵌内容，只要在模板中的Modal标签中组合内容即可。在用户点击Modal的取消时，只要将visible设为`false`。所以这里不涉及Modal的销毁问题。Modal的回收是和父组件一起的。

#### 命令式

Ant Design中的Modal组件也提供了几个静态的方法，用于在组件中手动初始化一个Modal，并且提供了`destroy`方法来手动销毁。

```
 const modal = Modal.success({
    title: 'This is a notification message',
    content: 'This modal will be destroyed after 1 second',
 });
```

Element UI中的Modal组件的API则是标准的命令式。


```
this.$prompt('请输入邮箱', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          inputPattern: /[\w!#$%&'*+/=?^_`{|}~-]+(?:\.[\w!#$%&'*+/=?^_`{|}~-]+)*@(?:[\w](?:[\w-]*[\w])?\.)+[\w](?:[\w-]*[\w])?/,
          inputErrorMessage: '邮箱格式不正确'
        }).then(({ value }) => {
          this.$message({
            type: 'success',
            message: '你的邮箱是: ' + value
          });
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '取消输入'
          });       
        });
```

Element UI的Modal API设计很有特色，首先调用创建Modal的方法后返回的是一个Promise。如果用户点击确定，那Promise就resolve。如果用户点击取消，那Promise就reject。我们可以在创建Modal的API调用之后链式的编写逻辑。

#### 自定义属性

Modal的属性中比较重要的就是内嵌的内容。在声明式定义的Modal的内嵌内容可以声明式的写在模板中。命令式的Modal则需要将内嵌内容作为初始化的属性传入。相比之下声明式的要更自然一些。

至于Modal另一组重要的属性，确认回调和取消回调。命令式的API在这方面更自然一些。特别是Element UI的基于Promise的调用。声明式的API则是在模板中声明属性，在View Controller中声明方法。