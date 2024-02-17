---
title: "DOM API详解（一）"
date: 2016-01-23 20:49:30
tags: 
- DOM
categories: 
- DOM
---

### 一、什么是DOM

Document Object Model(DOM)是用编程语言对HTML，XML以及SVG文档进行操作的接口。我们经常使用JavaScript来操纵DOM，不过DOM其实的语言无关的。它并不是JavaScript的一部分。

<!--more-->

虽然如此，DOM在前端开发中经常和JS一起讨论，并且对JS造成了极大的负面影响。这个原因就在于早年间各大浏览器厂商（主要是老版本IE），对DOM的实现并没有按W3C的标准来做，而是自己制定了一套接口，由此给前端开发造成了极大的麻烦。我们需要使用jQuery这样的基础库，使用封装好的跨平台兼容API，才能使代码兼容主要浏览器。

直到今天，jQuery这样的基础库依然是不可或缺的。好消息是，随着时代的进步，DOM API的兼容性在主流浏览器中已经不是问题。在没有低版本IE的移动端web中，我们就可以不用为这个问题劳神了。

然而相比于CSS和JS所拥有的明确版本迭代和丰富的文档，DOM无论是在版本的碎片化程度还是文档的数量上来说都很难和前两者相比。

打开MDN的[DOM文档首页](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model)，我们可以看到密密麻麻的API列表，点开其中的一个，可以看到更多的子API。其中除了DOM Core，有CSS DOM，还有HTMl5相关的DOM，更有Worker这样的浏览器相关的接口。可谓是一个大杂烩，CSS，JS，浏览器各种功能的API都在其中。因此本文就是一个DOM API的分类梳理，带大家看清DOM众多API的组成和层级。

### 二、DOM标准

我们从W3C的[DOM Technical Reports](https://www.w3.org/DOM/DOMTR)来看，Document Object Model (DOM)目前有Level 1，Level 2，Level 3，以及Level4三个标准。每一个Level的标准下又分为多个模块，其中DOM2 Core Specification的发布时间为2000-11-13，而DOM3 Core Specification的发布时间为2004-04-07，最新的DOM4发布于2015-11-19。

DOM2由许多个标准组成是，如图：

![DOM2模块](http://7oxh2b.com1.z0.glb.clouddn.com/DOM2%20modules.png)

然后我们来看看DOM3。DOM3标准构建在DOM2的基础上，因此数量上少于DOM2的模块。DOM3中的XPath Specification等等标准还没有正式成为推荐标准，因此没有加入图表。

![DOM3模块](http://7oxh2b.com1.z0.glb.clouddn.com/DOM3%20modules%20v3.png)

### 三、DOM2与DOM3，以及非核心API

和CSS标准一样，DOM标准的版本也是各自独立的。我们所说的DOM2，DOM3其实主要指DOM Core Specification的版本。在DOM Core Specification中会写明这个版本的DOM Core和哪些版本的标准一起组成新一代DOM标准。

因此我们也就可以解释为什么DOM3的组成标准要大大少于DOM2了。DOM2中的[Style Specification](https://www.w3.org/TR/DOM-Level-2-Style/)后来发展为了一个新的[CSSOM Specification](https://drafts.csswg.org/cssom/)，事实上这个标准已经不属于DOM的范畴，而被划入CSS标准。而[Events Specification](http://www.w3.org/TR/DOM-Level-2-Events/)的Level 3最后发展为了新的[UI Events Specification](https://www.w3.org/TR/uievents/)以及[Touch Events Specification](https://www.w3.org/TR/2013/REC-touch-events-20131010/)等等独立的Event Specification，补充了DOM2 Events Specification中缺失的事件。[Traversal and Range Specification](https://www.w3.org/TR/DOM-Level-2-Traversal-Range/)中的Traversal部分发展为了新的[Element Traversal Specification](https://www.w3.org/TR/ElementTraversal/)。

具体标准之间的关系可以看下表:

| **DOM2模块** | **新标准** |
| ---------------------------------------- | :--------------------------------------- |
| [Style Specification Level 2](https://www.w3.org/TR/DOM-Level-2-Style/) | [CSSOM Specification（Not a part of DOM now）](https://drafts.csswg.org/cssom/) |
| [Traversal and Range Specification Level 2](https://www.w3.org/TR/DOM-Level-2-Traversal-Range/) | [Element Traversal Specification（Part of DOM4）](https://www.w3.org/TR/ElementTraversal/) |
| [Events Specification Level 2](http://www.w3.org/TR/DOM-Level-2-Events/) | [UI Events Specification](https://www.w3.org/TR/uievents/)</br>   [Touch Events Specification](https://www.w3.org/TR/2013/REC-touch-events-20131010/)</br>   [Pointer Events Specification](https://www.w3.org/TR/2015/REC-pointerevents-20150224/)</br>   [Progress Events Specification](https://www.w3.org/TR/2014/REC-progress-events-20140211/) |

然后我们来看看所谓的非核心API。所谓的非核心API就是那些在W3C标准中不属于DOM的API，但是这些API和DOM的核心部分有着或多或少的关系。如CSS相关的[CSSOM Specification](https://drafts.csswg.org/cssom/)，[Selectors API](https://www.w3.org/TR/2013/REC-selectors-api-20130221/)等等。以及与HTML5相关的众多JavaScript API，比如[Web Storage](https://www.w3.org/TR/2015/PR-webstorage-20151126/)，[Web Workers](https://www.w3.org/TR/2015/WD-workers-20150924/)等等

本文关注的主要是DOM2，DOM3以及部分新模块的API，解读它们的构成，以及在浏览器的兼容情况。