---
title: "实战之IE兼容常见问题"
date: 2015-07-25 11:40:10
tags: 
- 兼容 
- 实战
categories: 
- 兼容 
- 实战
---
最近因为在实习，所以做的东西要测试，做了IE8的兼容，对IE8的常见CSS兼容问题有了一点认识，在这里总结一下。

### Html5标签问题

IE8不支持Html5的语义化新标签，`<header>,<nav>,<footer>`之类。对此Google出了一个html5shiv.js，来实现这些标签。

 这个html5shiv.js的原理就是使用document.createElement()这个dom方法来动态创建一个html元素对象。html5shiv.js中处理了IE在这个方法上的一些问题（某些元素动态加入的属性不管用），还提供了api，以及基础的CSS，使得元素在默认的display属性为block。
 
<!-- more -->
### RGBA支持问题

IE8不支持rgba这种颜色的表示方式，但我们如果在背景色使用rgba的话，还是有替代方案的：

    //IE8下
    filter:progid:DXImageTransform.Microsoft.gradient(startColorstr=#19fff
     fff,endColorstr=#19ffffff);
     
这句代码其实是用来实现渐变的，不过我们把startColor和endColor设成一样的便可以用来设置纯色背景了。

startColor的后六位便是正常的十六进制颜色代码，而前两位是alpha通道，对应关系如下：

透明度 |代码 | 
------------ | ------------- | 
0.1 | 19 | 
0.2| 33 | 
0.3|  4C| 
0.4 |  66| 
0.5| 7F |
0.6| 99 | 
0.7| B2 | 
0.8| C8 | 
0.9| E5 | 


### CSS3支持

这部分是硬伤，media query有成熟的方案支持，但是像border-radius这种虽也有办法可以实现，但是太过于麻烦。我们毕竟得move on，不能太过拘泥，有些时候平稳退化，优雅降级就可以了。

### To Be Contiued···