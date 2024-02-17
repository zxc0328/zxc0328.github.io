---
title: "最佳实践之多个高度不定水平实体垂直居中"
date: 2015-07-23 22:35:05
tags: 
- CSS
categories: 
- CSS
---
如何快速并且健壮地实现高度不一定的实体的居中呢？  

vertical-align显然是一种办法。这个元素从语义上来看就是用来实现垂直对齐的。要理解vertical-align，首先我们得了解几个概念，inline boxes, line box, 以及baseline。  

inline boxes是行内元素构成的行内box，而line box则是由一行内所有的inline boxes构成的box，比如：

    <div>
    	<span>我是inline box</span>
    	我是匿名inline box
    </div>

除了display为inline或者inline-block的标签会形成inline box，行内文本也会形成匿名的inline box。

一行所有的inline boxes构成匿名的line box，其中inline box中高度最高的那个的高度便是line box的高度，由此撑起了整行的高度。（这涉及到line-height方面的知识）

而关于baseline，则是文字在排版时顶部与底部中间的基线，如果是行内文本，具体位置随字体不同而不同，一般都在中线以下的位置。具体的需要参考西文字体学（Typography）(Jobs在大学旁听的课程之一，你值得拥有)  

而line—height其实就是两条基线之间的距离。

![baseline](http://7oxh2b.com1.z0.glb.clouddn.com/blog_7_24_1.png)

<!--more-->

对于inline-block级元素来说，基线的定义是这样的：


    The baseline of an 'inline-block' is the baseline of its last line
    box in the normal flow, unless it has either no in-flow line boxes
    or if its 'overflow' property has a computed value other than
    'visible', in which case the baseline is the bottom margin edge.
    （CSS2.1 W3C）
    
 一般是正常文档流中的最后一个line box的基线。
 
我们面对的场景是这样的，一个高度固定的div，中间有数个高度不固定的div需要被垂直居中。让我们来看看vertical-align:middle的定义：

    Align the vertical midpoint of the box with the baseline of the
    parent box plus half the x-height of the parent.（CSS2.1 W3C）
    
 （元素）的垂直中点与父元素的基线加上x轴高度的一半对齐。
 
 听起来有点奇怪，什么是x-height呢，x-height其实就是上图中小写字母x占据的高度，西文字体中小写字母x占据了mean line和baseline直接的这段空间，也就是所谓的字面所在。
 
 x这个字母恰好是镜面对称的，所以我们可以清楚的看到，vertical-align:middle就是将元素的中点对准x字母的中线，所以就形成了垂直居中的效果。
 
 好，现在我们来看实际的情况，我们有一个高度固定的div，然后内部有两个inline-block级的div，需要水平居中，这两个div的高度随机。
 
 代码：
 
    <div style="height:300px;">
    	<div style="display:inline-block">我是一号div，啊啊啊啊啊啊啊啊啊啊<br>啊啊啊啊啊啊啊啊</div>
    	<div style="display:inline-block">我是二号div<</div>
    </div>
 
 
<div style="height:100px;background-color:blue"><div style="display:inline-block;background-color:red">我是一号div，啊啊啊啊啊啊啊啊啊啊<br>啊啊啊啊啊啊啊啊</div><div style="display:inline-block;background-color:white">我是二号div</div></div>

这是默认情况，两个div的vertical-align被设置为baseline,可以看出两个div的baseline在一条线上。第一个div的baseline是最后一个line box的baseline。

那么父级元素div，一个块级元素，是否存在baseline呢？答案是否定的，在inline-block元素处于块极元素内部的情况下，vertical-align更多与同级inline-block元素的相互对齐有关，而和父级元素无关。

可以实验一下，我们把父级div设为inline-block，看看会发生什么事情。

    <div style="display:inline-block;line-height:100px;background-
    color:blue">
    	<div style="display:inline-block;background-color:red">
    	我是一号div，啊啊啊啊啊啊啊啊啊啊<br>啊啊啊啊啊啊啊啊
    	</div>
    	<div style="display:inline-block;background-color:white">
    	我是二号div
    	</div>
	</div>
	
<div style="display:inline-block;line-height:100px;background-color:blue;font-size:150px"><div style="display:inline-block;background-color:red;font-size:15px">我是一号div，啊啊啊啊啊啊啊啊啊啊<br>啊啊啊啊啊啊啊啊</div><div style="display:inline-block;background-color:white;font-size:15px">我是二号div</div>X</div>

这里我将父级div设为inline-block并且把line-height设为100px，因此其子元素的基线就和父元素的基线对齐了。我特意放了一个特大号的字，X的底部便是baseline了。

了解到这一点之后，我们便清楚了，父元素是块级元素的情况下，vertical-align只和同级元素有关。vertical-align:middle便是同级元素的中线相互对齐。

我们只需要一个空的div，它只起占位的作用，高度是100%，宽度是0，于是，其他高度小于父元素的子元素便会与这个占位元素的中线对齐，也就达到了垂直对齐。

    <div style="height:100px;background-color:blue;font-size:0">
		<div style="height:100%;width:0;display:inline-block;vertical-
		align:middle"></div>
    	<div style="display:inline-block;background-color:red;font-size:
    	15px;vertical-align:middle">我是一号div，啊啊啊啊啊啊啊啊啊啊<br>啊啊
    	啊啊啊啊啊啊</div>
    	<div style="display:inline-block;background-color:white;font-
    	size:15px;vertical-align:middle">我是二号div</div>
   	</div>


<div style="height:100px;background-color:blue;font-size:0"><div style="height:100%;width:0;display:inline-block;vertical-align:middle"></div><div style="display:inline-block;background-color:red;font-size:15px;vertical-align:middle">我是一号div，啊啊啊啊啊啊啊啊啊啊<br>啊啊啊啊啊啊啊啊</div><div style="display:inline-block;background-color:white;font-size:15px;vertical-align:middle">我是二号div</div></div>

垂直居中的效果就达成了！

ok，用:before伪元素来实现也是一个很不错的做法，相比空div要更简洁一些。

 
 