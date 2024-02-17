---
title: "css像素与理想viewport宽度"
date: 2015-05-12 19:57:37
tags: 
- CSS
categories: 
- CSS
---
最近在开发移动端网页时遇到了两大问题，一是移动端的触摸事件的实现；二是移动端页面的宽度设置问题。今天先来说一说移动端的页面宽度问题。  

首先要说的是css的像素，一个css像素，一个px，和设备的物理像素不是一回事。  
拿苹果的iphone来说，iphone3的分辨率是320\*480，而retina屏的iphone4是640\*960,  但如果在浏览器里输出视窗的宽度，两者都是320px宽。这说明Retina屏的iPhone用4个物理像素来渲染了一个css像素。iPad上的情况是相似的，新老ipad的css分辨率都是1024\*768。安卓上的情况也是类似的，比如nexus 5的物理分辨率是1920\*1080，而css分辨率是640\*360。  关于这个可以参考一下[quirksmode上的博文《此像素非彼像素》](http://www.quirksmode.org/blog/archives/2010/04/a_pixel_is_not.html)
<!--more-->
接下来是关于viewport的问题，我在阅读了[这篇博文](http://www.cnblogs.com/2050/p/3877280.html)之后对viewport的尺寸问题清楚了不少。
其中我们需要加以重点关注的主要是ideal viewport size（下称理想视窗宽度）。

    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=0">
以上代码中的width=device-width可以设置视窗宽度为设备宽度，也就是理想的宽度。而上文说到的css分辨率，便是默认情况下viewport按理想宽度来显示时css的像素值。

因此对于移动端的网页，一般设html和body的宽为100%即可，但是由此带来的思考便是移动端网页图片尺寸的设置问题。

![一加手机微信页面](http://muxistudio.qiniudn.com/blog-5-12.png)

这个微信页面的图片的宽度是640px，虽然手机的理想视窗宽度是320px左右，但是图片的像素还是应该和物理像素相近，不然就会出现一个物理像素渲染多个图片像素的情况。  
而640px正是大部分iphone手机的物理像素值。如果图片尺寸按最高的设备物理像素值来设置，便会使图片过大，使网页性能下降，而iPhone的640px便达到了一个平衡，让大部分设备都有一个良好的使用体验。（当然也不排除工程师偷懒，就拿市占50%的iPhone当最佳实践了）

找到同样关于这个主题的[一篇不错的博文](http://colachan.com/post/3435)我自愧不如···



