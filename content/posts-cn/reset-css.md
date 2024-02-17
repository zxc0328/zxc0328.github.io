---
title: "Thoughts on Reset.css"
date: 2015-11-05 19:53:19
tags: 
- CSS
categories: 
- CSS
---
###导语
CSS reset的作用主要就是清楚浏览器的默认样式，比如我们在Chrome里看到的`User Agent Stylesheet`就是Chrome浏览器的默认样式。不同的浏览器会有不同的默认样式，因此我们需要清除一些默认样式，**以达到在所有浏览器里显示效果的一致**。

不过这里面的学问还是很大的。首先我们只需要reset我们在项目中会使用的元素，这就涉及reset的模块化使用。第二我们只需要reset必要的属性。因为reset会加重浏览器渲染的性能开销，因此我们在css reset时必须要慎之又慎，保证最少的代码量和最佳的效果之间的平衡。

<!--more-->

###确定需要reset的元素

需要reset的元素有以下几类

**1.块级元素**

块级元素所需要的重置主要是针对边距、字体的重置。我们常用的元素有`body,html,h1,h2,h3,h4,h5,h6,p,ul,li,ol,video`等。

**2.表单元素**

块级元素所需要的重置除了边距之外，还有边框，`outline`等表单元素特有的属性的重置。我们常用的元素有`input,button,textarea,fieldset`等。  

**3.带特殊样式的元素**

带特殊样式的元素包括带`list-style`的`ul,ol`，带`text-decoration`的`a`。

**4.HTML5元素**  

在不支持HTML5标签的浏览器，如IE8及以下的浏览器中，在使用html5shiv这样的polyfill之后，我们需要手动给这些元素加上`display:block;`来设置默认的`display`属性。不过目前html5shiv已经包括了默认的样式，所以我们可以不用考虑HTML5元素的`display`属性reset。

**5.images**

`image`标签在IE8以以下版本的浏览器中默认会存在边框，所以我们需要重置`border:0;`。



###几个流行的reset.css

1.[Eric Meyer’s “Reset CSS” 2.0](http://cssreset.com/scripts/eric-meyer-reset-css/)

<pre class="prettyprint" style="height:400px;overflow:auto">
/* http://meyerweb.com/eric/tools/css/reset/ 
   v2.0 | 20110126
   License: none (public domain)
*/

html, body, div, span, applet, object, iframe,
h1, h2, h3, h4, h5, h6, p, blockquote, pre,
a, abbr, acronym, address, big, cite, code,
del, dfn, em, img, ins, kbd, q, s, samp,
small, strike, strong, sub, sup, tt, var,
b, u, i, center,
dl, dt, dd, ol, ul, li,
fieldset, form, label, legend,
table, caption, tbody, tfoot, thead, tr, th, td,
article, aside, canvas, details, embed, 
figure, figcaption, footer, header, hgroup, 
menu, nav, output, ruby, section, summary,
time, mark, audio, video {
	margin: 0;
	padding: 0;
	border: 0;
	font-size: 100%;
	font: inherit;
	vertical-align: baseline;
}
/* HTML5 display-role reset for older browsers */
article, aside, details, figcaption, figure, 
footer, header, hgroup, menu, nav, section {
	display: block;
}
body {
	line-height: 1;
}
ol, ul {
	list-style: none;
}
blockquote, q {
	quotes: none;
}
blockquote:before, blockquote:after,
q:before, q:after {
	content: '';
	content: none;
}
table {
	border-collapse: collapse;
	border-spacing: 0;
}
</pre>

评价：这是最早流行的`reset.css`。本文中的示例也参考了这个著名的`reset.css`。关于`blockquote`和`table`相关的reset可以参考这个文件。


2.[YUI css reset](http://cssreset.com/scripts/yahoo-css-reset-yui-3/)

<pre class="prettyprint" style="height:400px;overflow:auto">
/*
YUI 3.18.1 (build f7e7bcb)
Copyright 2014 Yahoo! Inc. All rights reserved.
Licensed under the BSD License.
http://yuilibrary.com/license/
*/

html {
    color: #000;
    background: #FFF
}

body,
div,
dl,
dt,
dd,
ul,
ol,
li,
h1,
h2,
h3,
h4,
h5,
h6,
pre,
code,
form,
fieldset,
legend,
input,
textarea,
p,
blockquote,
th,
td {
    margin: 0;
    padding: 0
}

table {
    border-collapse: collapse;
    border-spacing: 0
}

fieldset,
img {
    border: 0
}

address,
caption,
cite,
code,
dfn,
em,
strong,
th,
var {
    font-style: normal;
    font-weight: normal
}

ol,
ul {
    list-style: none
}

caption,
th {
    text-align: left
}

h1,
h2,
h3,
h4,
h5,
h6 {
    font-size: 100%;
    font-weight: normal
}

q:before,
q:after {
    content: ''
}

abbr,
acronym {
    border: 0;
    font-variant: normal
}

sup {
    vertical-align: text-top
}

sub {
    vertical-align: text-bottom
}

input,
textarea,
select {
    font-family: inherit;
    font-size: inherit;
    font-weight: inherit;
    *font-size: 100%
}

legend {
    color: #000
}
</pre>

评价：比起Eric Meyer的reset来说要更全面一些。加入了对`html`元素颜色的重置。在块级元素的重置上只设置了`margin`和`padding`。对行高并没有进行重置。

3.[normalize.css](https://necolas.github.io/normalize.css/)

<pre class="prettyprint" style="height:400px;overflow:auto">
/*! normalize.css v3.0.2 | MIT License | git.io/normalize */

/**
 * 1. Set default font family to sans-serif.
 * 2. Prevent iOS text size adjust after orientation change, without disabling
 *    user zoom.
 */

html {
  font-family: sans-serif; /* 1 */
  -ms-text-size-adjust: 100%; /* 2 */
  -webkit-text-size-adjust: 100%; /* 2 */
}

/**
 * Remove default margin.
 */

body {
  margin: 0;
}

/* HTML5 display definitions
   ========================================================================== */

/**
 * Correct `block` display not defined for any HTML5 element in IE 8/9.
 * Correct `block` display not defined for `details` or `summary` in IE 10/11
 * and Firefox.
 * Correct `block` display not defined for `main` in IE 11.
 */

article,
aside,
details,
figcaption,
figure,
footer,
header,
hgroup,
main,
menu,
nav,
section,
summary {
  display: block;
}

/**
 * 1. Correct `inline-block` display not defined in IE 8/9.
 * 2. Normalize vertical alignment of `progress` in Chrome, Firefox, and Opera.
 */

audio,
canvas,
progress,
video {
  display: inline-block; /* 1 */
  vertical-align: baseline; /* 2 */
}

/**
 * Prevent modern browsers from displaying `audio` without controls.
 * Remove excess height in iOS 5 devices.
 */

audio:not([controls]) {
  display: none;
  height: 0;
}

/**
 * Address `[hidden]` styling not present in IE 8/9/10.
 * Hide the `template` element in IE 8/9/11, Safari, and Firefox < 22.
 */

[hidden],
template {
  display: none;
}

/* Links
   ========================================================================== */

/**
 * Remove the gray background color from active links in IE 10.
 */

a {
  background-color: transparent;
}

/**
 * Improve readability when focused and also mouse hovered in all browsers.
 */

a:active,
a:hover {
  outline: 0;
}

/* Text-level semantics
   ========================================================================== */

/**
 * Address styling not present in IE 8/9/10/11, Safari, and Chrome.
 */

abbr[title] {
  border-bottom: 1px dotted;
}

/**
 * Address style set to `bolder` in Firefox 4+, Safari, and Chrome.
 */

b,
strong {
  font-weight: bold;
}

/**
 * Address styling not present in Safari and Chrome.
 */

dfn {
  font-style: italic;
}

/**
 * Address variable `h1` font-size and margin within `section` and `article`
 * contexts in Firefox 4+, Safari, and Chrome.
 */

h1 {
  font-size: 2em;
  margin: 0.67em 0;
}

/**
 * Address styling not present in IE 8/9.
 */

mark {
  background: #ff0;
  color: #000;
}

/**
 * Address inconsistent and variable font size in all browsers.
 */

small {
  font-size: 80%;
}

/**
 * Prevent `sub` and `sup` affecting `line-height` in all browsers.
 */

sub,
sup {
  font-size: 75%;
  line-height: 0;
  position: relative;
  vertical-align: baseline;
}

sup {
  top: -0.5em;
}

sub {
  bottom: -0.25em;
}

/* Embedded content
   ========================================================================== */

/**
 * Remove border when inside `a` element in IE 8/9/10.
 */

img {
  border: 0;
}

/**
 * Correct overflow not hidden in IE 9/10/11.
 */

svg:not(:root) {
  overflow: hidden;
}

/* Grouping content
   ========================================================================== */

/**
 * Address margin not present in IE 8/9 and Safari.
 */

figure {
  margin: 1em 40px;
}

/**
 * Address differences between Firefox and other browsers.
 */

hr {
  -moz-box-sizing: content-box;
  box-sizing: content-box;
  height: 0;
}

/**
 * Contain overflow in all browsers.
 */

pre {
  overflow: auto;
}

/**
 * Address odd `em`-unit font size rendering in all browsers.
 */

code,
kbd,
pre,
samp {
  font-family: monospace, monospace;
  font-size: 1em;
}

/* Forms
   ========================================================================== */

/**
 * Known limitation: by default, Chrome and Safari on OS X allow very limited
 * styling of `select`, unless a `border` property is set.
 */

/**
 * 1. Correct color not being inherited.
 *    Known issue: affects color of disabled elements.
 * 2. Correct font properties not being inherited.
 * 3. Address margins set differently in Firefox 4+, Safari, and Chrome.
 */

button,
input,
optgroup,
select,
textarea {
  color: inherit; /* 1 */
  font: inherit; /* 2 */
  margin: 0; /* 3 */
}

/**
 * Address `overflow` set to `hidden` in IE 8/9/10/11.
 */

button {
  overflow: visible;
}

/**
 * Address inconsistent `text-transform` inheritance for `button` and `select`.
 * All other form control elements do not inherit `text-transform` values.
 * Correct `button` style inheritance in Firefox, IE 8/9/10/11, and Opera.
 * Correct `select` style inheritance in Firefox.
 */

button,
select {
  text-transform: none;
}

/**
 * 1. Avoid the WebKit bug in Android 4.0.* where (2) destroys native `audio`
 *    and `video` controls.
 * 2. Correct inability to style clickable `input` types in iOS.
 * 3. Improve usability and consistency of cursor style between image-type
 *    `input` and others.
 */

button,
html input[type="button"], /* 1 */
input[type="reset"],
input[type="submit"] {
  -webkit-appearance: button; /* 2 */
  cursor: pointer; /* 3 */
}

/**
 * Re-set default cursor for disabled elements.
 */

button[disabled],
html input[disabled] {
  cursor: default;
}

/**
 * Remove inner padding and border in Firefox 4+.
 */

button::-moz-focus-inner,
input::-moz-focus-inner {
  border: 0;
  padding: 0;
}

/**
 * Address Firefox 4+ setting `line-height` on `input` using `!important` in
 * the UA stylesheet.
 */

input {
  line-height: normal;
}

/**
 * It's recommended that you don't attempt to style these elements.
 * Firefox's implementation doesn't respect box-sizing, padding, or width.
 *
 * 1. Address box sizing set to `content-box` in IE 8/9/10.
 * 2. Remove excess padding in IE 8/9/10.
 */

input[type="checkbox"],
input[type="radio"] {
  box-sizing: border-box; /* 1 */
  padding: 0; /* 2 */
}

/**
 * Fix the cursor style for Chrome's increment/decrement buttons. For certain
 * `font-size` values of the `input`, it causes the cursor style of the
 * decrement button to change from `default` to `text`.
 */

input[type="number"]::-webkit-inner-spin-button,
input[type="number"]::-webkit-outer-spin-button {
  height: auto;
}

/**
 * 1. Address `appearance` set to `searchfield` in Safari and Chrome.
 * 2. Address `box-sizing` set to `border-box` in Safari and Chrome
 *    (include `-moz` to future-proof).
 */

input[type="search"] {
  -webkit-appearance: textfield; /* 1 */
  -moz-box-sizing: content-box;
  -webkit-box-sizing: content-box; /* 2 */
  box-sizing: content-box;
}

/**
 * Remove inner padding and search cancel button in Safari and Chrome on OS X.
 * Safari (but not Chrome) clips the cancel button when the search input has
 * padding (and `textfield` appearance).
 */

input[type="search"]::-webkit-search-cancel-button,
input[type="search"]::-webkit-search-decoration {
  -webkit-appearance: none;
}

/**
 * Define consistent border, margin, and padding.
 */

fieldset {
  border: 1px solid #c0c0c0;
  margin: 0 2px;
  padding: 0.35em 0.625em 0.75em;
}

/**
 * 1. Correct `color` not being inherited in IE 8/9/10/11.
 * 2. Remove padding so people aren't caught out if they zero out fieldsets.
 */

legend {
  border: 0; /* 1 */
  padding: 0; /* 2 */
}

/**
 * Remove default vertical scrollbar in IE 8/9/10/11.
 */

textarea {
  overflow: auto;
}

/**
 * Don't inherit the `font-weight` (applied by a rule above).
 * NOTE: the default cannot safely be changed in Chrome and Safari on OS X.
 */

optgroup {
  font-weight: bold;
}

/* Tables
   ========================================================================== */

/**
 * Remove most spacing between table cells.
 */

table {
  border-collapse: collapse;
  border-spacing: 0;
}

td,
th {
  padding: 0;
} 
</pre>

评价：这是一个全面的reset.css。最大的特点就是模块化。分`HTML5 display definitions`,`Links`,`Text-level semantics`,`Embedded content`,`Grouping content`,`Forms`,`Tables`这几个模块来分别设置。都有详细的注释。是一份不错的学习材料。

###`font-famliy`的设置

在根元素处设置的`font-famliy`属性决定了全站所有页面的默认字体。一般我们设置`font-famliy`的顺序为，默认英文字体，默认中文字体，fallback中文字体。

比如:

<pre class="prettyprint">
font-famliy:"Arial","Helvetica","Microsoft YaHei","PingFang SC","STHeiti","sans-serif";
</pre>

首先我们设置默认的英文字体。因为中文字体之中是带有一套英文字体的，所有我们会首先设置默认英文字体来防止被覆盖。然后我们设置windows和OS X上的默认中文字体，这里一般用英文来设置，来避免一些兼容性问题。最后我们设置`sans-serif`衬线字体来做一个fallback。

对于字体要求一般的情况下，这样的设置其实就满足我们的要求了。

<pre class="prettyprint">
font-famliy:"Arial","Helvetica","sans-serif";
</pre>

这种设置下，系统会调用默认的无衬线中文字体。所以如果在某些有很多无衬线中文字体的操作系统，如OS X下没有特定的字体要求的话，可以将第一种设置简化为这种设置。移动端web的字体比较多，所以设置方式与上面的类似。

###一个基本的reset.css示例

<pre class="prettyprint">
/* 全局   
===================================================================== */

/**
 * 重置默认行高
 */
body{
	line-height:1;}
html{
	margin:0;
}


/* 文字语义标签  
===================================================================== */
h1,h2,h3,h4,h5,h6{
	marign:0;
	font-size:100%;
	font-weight: normal;
}
p{
	margin:0;
}

/* 表单
===================================================================== */
fieldset{
	border:0;
}
input,button,textarea{
	border:0;
	outline:0;
	padding:0;
}
/**
 * chrome下button默认的box-sizing属性是border-box
 */
button{
	box-sizing:content-box;
}

/* 链接
===================================================================== */
a{
	color:#fff;
	text-decoration:none;
}

/* 嵌入内容
===================================================================== */
img{
	border:0;
}

/* 列表
===================================================================== */
ol, ul {
	list-style: none;
}

/* 表格
===================================================================== */
table {
    border-collapse: collapse;
    border-spacing: 0
}
</pre>

###结语

上面列出的示例，其实如Meyer说的，只是一个开始。我们需要根据我们的具体需求来做微调，来达到最好的“性价比”。

比如众多的HTML5元素。如果你的项目中不打算使用这些元素，那么你就不用添加对这些元素的reset。之前列的三个流行reset的内容长短不一，很大程度上也是因为大家的目标不一样。所以我们应该以这些优秀的reset.css为模板，来打造自己的reset.css。




