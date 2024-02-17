---
title: "居中一切(centering all the directions)by KYLE SIMPSON"
date: 2015-05-21 22:53:26
hiddenInHomeList : true
tags: 
- HTML5
- 翻译
categories: 
- HTML5
- 翻译
---
![css centering](http://muxistudio.qiniudn.com/ccs_center_660_380px.jpg)

自从文明的曙光降临以来，人类一直在斗争,对抗一切不可能之事，来达到人类进化的下一个阶段；去在这个伟大的星球上，以及环绕我们的宇宙中圈出自己的地盘；勇敢的进军任何地点...噢，原谅我这么夸张。

<!--more-->

![twitter](http://muxistudio.qiniudn.com/e594bf4bef370534e5b4470b8fd2c4d5.png)


我想说，“如果你曾与用css居中内容(尤其是垂直居中)苦战，请举手。”其实说“如果你从没有挣扎过...请举手。”可能更快些。CSS是一种强大到让人震惊的工具，但是还是有一些事情，比如垂直居中，甚至在今天仍处于“太TM难了”的状态。我将研究居中的多种方法，不过我认为我们仍将需要更好的东西。


###&lt;center&gt;###
首先，如果你还在使用`<center>`标签来居中，那么1999年喊你回家吃饭。正经地说，你把你的“现代web开发者”会员卡交出来吧。直到你改掉你如此丑陋的代码为止。

*译者注：谷歌首页的logo就使用了`<center>`标签😰*
###text-align:center###
每当说到居中文本的时候，你显然无法放弃<code>text-align:center</code>。它的功能正如它的字面意义。它将父容器中的行内文本水平居中。

例：

    <style>
    #ex1_container { text-align:center; width:200px; background-      
    color:yellow; }
    </style>
    <div id="ex1_container">Hello World</div>


<div id="ex1_container1" style="text-align: center; width: 200px; background-color: yellow;">Hello World</div>

这很容易。这里的技巧在于构造一个我们希望其子元素居中的父元素。我们不直接对内容添加样式，而是它的父容器。

除此之外，这个方法只对行内元素起作用（而且大概只该被使用在文本上，不过这点不被严格限制）。而且它只作用于水平方向。

###vertical-align:middle###
 “不过，不过···”你说，“不过<code>vertical-align:middle</code>如何？”对，听起来不错。可是结果，没那么理想。
 
 不像text-align：center，设置在一个父元素上来形容水平如何居中*它的内容*，<code>vertical-align:middle</code>作用在内容本身，并且不在它的父元素处描述它的位置，只是描述相对于周围别的行内元素的位置。
 
 对，只对让行内文本和行内图像整齐地排排坐有用，但这不是我们想要的神一样的垂直居中。
 
**注**：你可以用<code>display:table</code>这个玩意在父元素上，把<code>display:table-cell</code>用在内容元素上。坏处：这就回到表格时代了，而且在低版本IE上兼容性很差。

###line-height###
一个有用的垂直居中文本的技巧是设置它容器的<code>line-height</code>属性为你想要的高度，然后文本会（大概的）自动垂直居中。这是一个常用的技巧。有时<code>line-height</code>会有一些副作用，并且它并不是十分完美的（由于字体单位不同，等等）。不过，当你所有的全部只是一根火柴棒，每样事物看起来都像是可以生火的木柴，对吧？

So，我们该怎么居中块级元素呢?

###margin:auto###

好吧，理论上说，我们大概会使用<code>margin: .. auto</code>，此次“..”表示我们的垂直margin。（我们一会后再说垂直居中的问题）

那么这是怎么工作的（水平居中）？例子如下：


    <style>
    #ex2_container { width:200px; background-color:yellow; }
    #ex2_content { margin:0px auto; background-color:gray; color:white;       
    display:table; }
    </style>
 
    <div id="ex2_container"><div id="ex2_content">Hello World</div>
    </div>



<div id="ex2_container" style="width: 200px; background-color: yellow;"><div id="ex2_content" style="margin: 0px auto; background-color: gray; color: white; display: table;">Hello World</div></div>

**注：**因为在这个例子中我没有确切定义<code>\#ex2_content</code>元素的宽度（我只想让它和文本需要的那么大），为了让它收缩在文字周围但依然可以被居中，我使用<code>display:table</code>。你大概以为<code>display:inline-block</code>会有用，但是它不起作用。不过如果你手动设定<code>\#ex2\_content</code>的<code>width</code>或者<code>max-width</code>，你就完全不需要设定<code>display:table</code>了。

很酷吧?这里，我们只是直接告诉内容我们需要相对父元素居中它本身，当然，是水平的。这相比
<code>text-align:center</code>来说不够语义化，因为你差不多正在描述内容周围的空间而不是控制内容本身。但我们在这点上会放它一马，因为这还不赖。

所以，你会说“那这肯定适用于垂直居中喽!?”当然不是，你是有多傻多天真才希望会有这样一个对称而简单的方案!?正经地说，你是在火星学的CSS吗!？

不幸的是，出于某些凡人无法理解的原因，这个“小把戏”不适用于垂直居中。

###hacks，hacks###

[这里](http://blog.themeforest.net/tutorials/vertical-centering-with-css/)有大量的hacks，比如像负边距那样的奇异的东西，还有[幽灵元素](http://css-tricks.com/centering-in-the-unknown/)<code>::before</code>等等其他的什么鬼。他们中的大多数都很脆弱，除非你的内容是固定宽度。

如果没有其他解决方案了，这将是我们在被压迫时会变成一种充满创造力的物种的证明——我们想办法制造它。

###translate(-50%,-50%)###

对Chris Coyier在[CSS-TRICKS.com](http://css-tricks.com/)上发表的一个技巧：[使用position和translate](http://css-tricks.com/centering-percentage-widthheight-elements/)脱帽致敬。

如果你在一些子元素上设定<code>position:absolute</code>，然后在那个子元素上设定<code>left:50%; top:50%</code>，它的左上角会被自动垂直居中和水平居中。呃，我们大概都知道这个。不过这完全没有帮助，除非我们的内容是1x1像素大。

这个技巧的后续，是<code>translate(-50%,-50%)</code>。不同于大多数这类百分比计算，像<code>left</code>和<code>top</code>，是相对于父容器来说，<code>translate(-50%,-50%)</code>中的百分比是相对于这个元素本身。

所以，我们把容器的左上角定位到中心，并且将它按宽度和长度的一半“转换”回来，然后duang!!，奇迹般得在两个方向都居中了。

例：

    <style>
    #ex3_container { 
    	width:200px;
    	height:200px;
    	background-color:yellow;
    	position:relative; }
     #ex3_content { 
    	left:50%; 
    	top:50%; 
    	transform:translate(-50%,-50%); 
    	-webkit-transform:translate(-50%,-50%);
    	background-color:gray;
    	color:white; position:absolute; }
	 </style>
 
	<div id="ex3_container">
		<div id="ex3_content">Hello World</div>
	</div>


 
<style>
  #ex3_container { width:200px; height:200px; background-color:yellow; position:relative; }
  #ex3_content { left:50%; top:50%; transform:translate(-50%,-50%); -webkit-transform:translate(-50%,-50%); background-color:gray; color:white; position:absolute; }
</style>
 
<div id="ex3_container"><div id="ex3_content">Hello World</div></div>
 

这个技巧的鲁棒性明显比其他hack更强，因为它优雅地响应了各种情况，像不定宽内容，<code>min-width</code>,<code>max-height</code>,<code>overflow:scroll</code>,etc。它基本上做到了你希望它做的。无论它收缩或者扩展（除非你加了限制），都保持居中。棒棒哒！

去[把玩](http://jsbin.com/etupoz/1/)一下看看它能做什么。

不过，让我们诚实一些，这个方案不太语义化，而且在语法上比较丑陋（特别是因为那些奇怪的前缀）。并且落到了用力过度的那一类里。

这大概是目前最好的技巧了，考虑到了所有情况。但是让我伤心的是我们依旧需要写这样的文章，玩弄CSS来达到我们需要的效果。

###这是我们能做到的最好吗###

如果这些“技巧”和“hacks”在你听起来像“垃圾科学”，这很正常。为什么这个该死的问题这么难？为什么我们需要根据不同的场景使用不同的技术？

一种统治一切的居中规则听起来如何？难道不该有一种标准的“东西”被加入CSS去处理一切问题，取代我们现在所处的处处hack的情形？

（阴谋论：制定CSS标准的同志们需要继续给博主们猛料，所以他们故意留了一些没有解决的问题）

这里有一个我刚想出来得平常的提议：

    <style>
    #ex4_container { content-positioning:50% 50%; width:200px; 
    height:200px; background-color:yellow; }
    #ex4_content { content-positioning-anchor:50% 50%; 
    background-color:gray; color:white; }
    </style>
 
我建议应该有一个<code>content-positioning</code>属性在父元素上来控制其子元素的位置。还有一个<code>content-positioning-anchor</code>属性控制子元素上起始点的位置。它会，当然，默认设为<code>0px 0px</code>，但是如果你把它设定为<code>50% 50%</code>，你会说，仅此而已，“把我的内容的中心放在容器的中心”。

duang！我刚刚解决了全世界的难题。好吧，这不是真的。这种假设令人上瘾。

###你怎么看？###

让我们讨论一下统一的解决方案会是什么样的。或者可能已经有了我不知道的特别秘密的CSS标准提案。无论哪种情况，让我们免除hack之苦，向这个频繁地使用场景的标准化前进。

你怎么看？告诉我们吧！






