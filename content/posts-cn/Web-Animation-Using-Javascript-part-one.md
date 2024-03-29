---
title: "Web Animation Using Javascript part one"
date: 2015-05-23 22:47:00
tags: 
- 翻译
- Javascript
categories: 
- 翻译
- Javascript
---
##第二章：Velocity.js动画##
在这一章，你将学到由Velocity.js提供的特性，指令，和选项。如果你熟悉基于jQuery的动画，那么你已经知道如何使用Velocity.js了；它的功能几乎和jQuery的$.animate()函数一模一样。

不过抛开你现有的知识，本章中对特性的井井有条的分类将会向你介绍动画引擎行为的细微差别。掌握这些细微差别将会帮助你从新手成为专业人士。即使你已经对jQuery动画和Velocity.js相当熟悉了，也给自己一个机会，快速浏览本章。**你必定会发现一些你没意识到的可行之事。**
<!-- more -->
###JavaScript动画库类型###

JavaScript动画库有很多类型。有些在浏览器中重现物理接触效果。有些使WebGL和Canvas动画更容易维护。有些专注于SVG动画。有些改善了UI动画————最后这种类型正是本书的重点。

两种广受欢迎的UI动画库是GSAP和Velocity。你将在本书中使用Velocity，因为它在MIT许可下是免费的，外加它拥有极其强大的功能以供编写整洁且富有表现力的动画代码。Velocity被很多知名站点使用，包括Tumblr，Gap，还有Scribd。

噢，而且它是由本书的作者创造的！

###安装jQuery和Velocity###

你可以从jQuery.com下载jQuery，从VelosityJS.org下载Velosity。在你的页面上使用它们——和任何JavaScript库一样——简单地把指向相应库文件的`<script> </script> `标签放在你页面的`</body>`标签之前。如果你想链接预部署版本的库文件（而不是你电脑上的本地拷贝），你的代码看起来可能是这样：


    <html>
      	<head>My Page</head>
       <body>
        	 My content.
        	 <script src=”//code.jquery.com/jquery-2.1.1.min.js”>
           </script>
           <script src=”//cdn.jsdelivr.net/velocity/1.1.0/
           velocity.min.js”>
           </script>
      </body>
    </html>

当一起使用jQuery和Velocity时，在Velocity前引入jQuery。    
这就对了！现在你已经准备好了。

##使用Velocity:基础##

为了熟悉Velocity，我们将从基本的组件开始：参数，属性，值，和链式调用。因为jQuery几乎无所不在，看看Velocity和jQuery的关系也是有必要的。


###Velocity和jQuery###

Velocity可以独立于jQuery运行，但是二者可以一起使用。一般我们推荐这么做来获得jQuery的链式调用能力：当你已经使用jQuery预选择了一个元素，你可以调用.velocity()来拓展它从而施加动画效果：

    // 把jQuery元素对象赋值给一个变量
    var $div = $(“div”);
    // 使用Velocity对元素施加动画
    $div.velocity({ opacity: 0 });
    这种语法和jQuery自带的animate函数一模一样：
    $div.animate({ opacity: 0 });
    
本书所有的例子都使用Velocity和jQuery的结合，因此可以使用这种语法。

###参数###

Velocity接收多个参数。第一个参数是一个映射CSS属性到其最终值的对象。属性及其接收值的类型直接与CSS中使用的相对应（如果你不熟悉基础的CSS属性，在读这段代码之前读一本HTML和CSS的介绍性书籍）：

     // 对元素施加动画使width变为“500px”，opacity变为1。
     $element.velocity({ width: “500px”, opacity: 1 });
 --
 *Tip*  
 在JavaScript中，如果你将要提供一个包含字母（而不是只有整数）的属性值，把属性值放在引号中。
 --

你可以传递一个指定了动画选项的对象作为第二个参数。

    $element.velocity({ width: “500px”, opacity: 1 }, { duration: 400, 
    easing: “swing” });
    
这里也有一个简写的参数语法：你可以使用逗号分隔语法，而不是传递一个包含选项的对象作为第二个参数。这要求以任何逗号分隔的顺序列出动画时长的值（接收一个整数），缓动模式（一个字符串），以及回调函数（一个函数）。（你一会将会学到这些选项的作用。）

      //时长1000ms的动画 (使用默认缓动模式swing)
      $element.velocity({ top: 50 }, 1000);
      //时长1000ms的动画，并且使用缓动模式“ease-in-out”
      $element.velocity({ top: 50 }, 1000, “ease-in-out”);
     //使用缓动模式“ease-out”，使用默认时长值400ms)
     $element.velocity({ top: 50 }, “ease-out”);
     //时长1000ms的动画,并且在动画完成时触发一个回调函数        
     $element.velocity({ top: 50 }, 1000, function() 
     { alert(“Complete.”) });
     
当你只需要指定基本选项(时长，缓动模式，回调函数)的时候，这个简写语法是一种传递动画选项的快捷方法。如果你在这三个选项之外传递一个动画选项，你必须把所有选项调回对象语法。  
因此，如果你想要指定一个延时选项，改变以下的语法：

    $element.velocity({ top: 50 }, 1000, “ease-in-out”);
 
到这种语法：

    //重新指定上面使用的动画选项，但包括一个值为500ms的延时选项  
    $element.velocity({ top: 50 }, { duration: 1000, easing: “ease-in-
    out”, delay: 500 });
    
你不能这么做：

    //错误：把动画选项分为逗号分隔语法和对象语法来写
    $element.velocity({ top: 50 }, 1000, { easing: “ease-in-out”, 
    delay: 500 });
    
###属性###

基于CSS的属性动画和基于JavaScript的属性动画有两处区别。

首先，不像在CSS中那样，Velocity中每个CSS属性只接收单个数值。所以，你可以传入：

    $element.velocity({ padding: 10 });
    
或者：

    $element.velocity({ paddingLeft: 10, paddingRight: 10 });
    
但是你不能传入:

    // 错误：CSS属性被传入了多个数值
    $element.velocity({ padding: “10 10 10 10” });
    
如果你想要把四个padding值(top, right, bottom, 和 left)加入动画，将它们列为分离的属性。

    // 正确
    $element.velocity({
      paddingTop: 10,
      paddingRight: 10,
      paddingBottom: 10,
      paddingLeft: 10
    });
    
其他可以接收多个数值的常见CSS属性包括`magrin`，`transform`， `text-shadow`，和`box-shadow`。

为了形成动画而将组合属性分离到它们的子属性中给了你对缓动值的进一步控制。举个例子，在CSS中，当对父padding属性下的多个子属性施加动画时，你只可以指定一种属性范围的缓动类型。在JavaScript中，你可以为每个子属性指定独立的缓动值——这个特性的优势会在之后这章关于CSS transform属性的讨论中变得明确。

列出我们的独立子属性也可以使你的动画代码更易阅读和维护。

基于CSS的属性动画和基于JavaScript的属性动画的第二处不同是JavaScript属性没有词与词之间的横杠，第一个词之后的所有词必须大写。例如，padding-left变成了paddingLeft，而background-color变成了backgroundColor。还有一点，JavaScript属性名不应该被置于引号中：

    // 正确
    $element.velocity({ paddingLeft: 10 });
    // 错误: 使用了横杠且没有大写
    $element.velocity({ padding-left: 10 });
    // 错误：在JavaScript格式的属性名两边使用了引号
    $element.velocity({ “paddingLeft”: 10 });

###值###

Velocity支持`px`,`em`,`rem`,`%`,`deg`,`vw`和`vh`单位。如果你没有为数值提供某个类型的单位，一个基于CSS属性类型的合适的单位会被自动添加。对于大多数属性，px是默认单位，不过一个接收旋转角度的属性，比如rotateZ,会被自动添加deg单位：

    $element.velocity({
      top: 50, // 默认设置为px单位类型
      left: “50%”, // 我们手动指定了%单位类型      
      rotateZ: 25 // 默认设置为deg单位类型
    });
    
为你所有的属性值显式声明单位类型，通过使px单位和它的替代选择之间的对比更加明显来增加代码在快速浏览时的清晰程度。

另一个Velocity相对于CSS的优势是它支持可以被选择性添加在属性值之前的四种数值运算符：+，—，*，和/。它们和JavaScript中的数学运算符一一对应。你可以把这些数值运算符与一个等号组合来进行相应的数学运算。请参考实例中的行内代码注释：

    $element.velocity({
      top: “50px”, // 没有运算符。不出所料地向前运动50px。
      left: “-50”, // 负运算符。不出所料地向前运动-50px。
      width: “+=5rem”, // 将当前宽度值转换为对应的rem值并加上5个单位值。
      height: “-10rem”, // 将当前高度值转换为对应的rem值并减去10个单位值。
      paddingLeft: “*=2” // 把当前的paddingLeft值加倍。
      paddingRight: “/=2” // 把当前的paddingLeft值除以2。
     });

Velocity的简写特性，像数值运算符，把动画逻辑完全保留在动画引擎中。这不仅仅因排除了手动数值计算而使代码更加简洁，也通过告诉Velocity更多你计划如何对元素施加动画来提升了性能。Velocity处理的逻辑越多，它优化你的代码来达到更高的帧数的能力就越强。

###链式调用###

当多个Velocity调用被连续链接在一个元素（或一系列元素）上的时候，它们会自动形成队列。这说明每个动画在前一个动画完成时开始：

    $element
      // 对width和height属性施加动画
      .velocity({ width: “100px”, height: “100px” })
      // 当宽度和高度的动画完成之后，对top属性施加动画
      .velocity({ top: “50px” });
      
##使用Velocity:选项##

为了完善对Velocity的介绍，让我们快速浏览最常用的选项:时长，缓动，起始回调及结束回调，循环，延时，和显示。

###时长###

你可以指定时长选项，它决定了一个动画调用要多长时间结束，以毫秒（1/1000秒）为单位或是三种简写时长之一：“慢”（相当于600ms），“普通”（400ms），或者“快”（200ms）。当以毫秒指定一个时长值时，应提供一个不带任何单位类型的整数：

    // 施加时长1000ms（1秒）的动画
    $element.velocity({ opacity: 1 }, { duration: 1000 });
    
或者：

    
    $element.velocity({ opacity: 1}, { duration: “slow” });
   
当你回顾你的代码时，使用命了名的简写时长的好处是它们表达了一个动画的节奏（是慢还是快？）。如果你全部使用这些简写，它们自然也将带给你的站点更统一的动画设计，因为所有的动画将会落在三个速度分类中而不是被传递一个随意的值。


###缓动###

缓动是定义一个动画的整个过程中不同部分发生快慢的数学函数。举个例子,“ease-in-out”缓动类型表明动画应该在第一部分缓缓加速（淡入）然后在最后部分缓缓减速（淡出）。相比之下，“ease-in”缓动类型产生的动画在第一部分加速到一个目标速度但随后保持一个恒定速度直到动画完成。“ease-out”缓动类型是“ease-in”的相反情况，动画开始并且保持一个恒定速度直到在动画的最后部分缓缓减速。

与第一章，“JavaScript动画的优势”中讨论的基于物理的运动很相似，缓动给你力量来向你的动画注入人格。拿一个使用线性缓动的动画会让人感到多么的机械来说。（线性缓动产生一种以相同速率开始，运行，和结束的动画。）这种机械的感受是与现实世界的线性机械运动相联系的结果：自我导航的机械物体往往以直线移动并且以恒定速度操作，因为没有任何美学的抑或生理的原因去让它们不那么做。

与此相对，有生命的东西-不管是人体或是正被风吹的树-在真实世界中从不以一个恒定的速度移动。摩擦力和其他外部力量令它们以不同的速度移动。

伟大的动画设计师对有机的运动怀有敬意，因为这让人感觉界面正流畅地回应用户的互动。在移动应用中，举个例子，你希望一个菜单在你将它滑出屏幕时马上加速离开你的手指。如果菜单只是以一个恒定速度从你的手指移开-像一个机械手臂-你将会感到滑动只是触发了一连串不受你控制的运动事件。

关于缓动类型的力量，你将会在第三章：“动画设计理论”中学到更多。对于现在，让我们快速浏览Velocity的所有可用的缓动类型：

+jQuery UI的三角函数缓动。关于这些缓动方程的完整列表，以及它们的加速效果简介的互动演示，请查阅easing.net上的demo。

    $element.velocity({ width: “100px” }, “easeInOutSine”);

+CSS缓动："ease-in", "ease-out", "ease-in-out", 和 "ease" (一个与"ease-in-out"有细微不同的版本)。

    $element.velocity({ width: “100px” }, “ease-in-out”);

+CSS贝塞尔曲线：贝塞尔曲线缓动允许对一个缓动加速曲线结构的完全控制。一条贝塞尔曲线通过指定一张图表上四个等距点的高度来定义，Velocity接收的图表格式是有四项二进制值的数组。访问cubic-bezier.com来查看一个创建贝塞尔曲线的互动指南。

    $element.velocity({ width: “100px” }, [ 0.17, 0.67, 0.83, 
    0.67 ]);
    
+弹簧模型：这种缓动类型模仿一个被拉伸然后突然释放的弹簧的弹性形变。正如定义弹簧运动的经典物理方程，这种缓动类型允许你传递一个形式为[张力，摩擦力]的二项数组。一个更高的张力值（默认为500）增加了总速度及弹性。一个更低的摩擦力（默认为20）增加了振动结束时的速度。

    $element.velocity({ width: “100px” }, [ 250, 15 ]);
    
+“spring”缓动是一种预定义的弹簧模型的实现，它在你不想试验张力和摩擦力值的时候使用方便。

    $element.velocity({ width: “100px” }, “spring”);
    
记住你也可以传递缓动选项作为一个选项对象参数中的一个直接定义的属性。

    $element.velocity({ width: 50 }, { easing: “spring” });
    
不要被你可用的缓动选项的数量吓倒。你将经常依靠CSS缓动类型和“spring”缓动，它们适合绝大部分动画使用实例。最复杂的缓动类型，贝塞尔曲线，被脑中有一个高度具体的缓动方式且不怕麻烦的开发者使用最多。

*注意*

本节其余的Velocity选项必须被直接传递入一个选项对象。不像那些已经被描述的，这些附加选项不能以简写的逗号分隔语法在Velocity中使用。

###起始回调和结束回调###

`begin`和`complete`选项允许你指定在动画中的特定节点被触发的函数：给`begin`选项传递一个在动画开始前被调用的函数。相反地，传给`complete`选项一个在动画完成时被调用的函数。

在这两种选项中，函数在每次动画调用时只被调用一次，就算多个元素同时被施加动画：

    var $divs = $(“div”);
    $divs.velocity(
       { opacity: 0 },
       // 在动画开始前打开一个警告窗口
       {
       begin: function () { console.log(“Begin!”); },
        // 一旦动画完成就打开一个警告窗口
       complete: function () { console.log(“Complete!”); }
    } );
    
*回调函数*

这些选项常被叫做“回调函数”（或“回调”）因为它们将在特定的事件发生时被“调用”。回调函数对触发依赖于元素可见度的事件十分有用。举个例子，如果一个元素在开始时不可见，然后发生动画使透明度变为1，那么随后触发一个UI事件，一旦用户能看见新内容时便更改内容，可能是合适的。

记住你不需要使用回调函数来依次排列动画；当多个动画被指派在单个元素或一组元素上时，动画会自动按顺序触发。回调函数用来使非动画逻辑形成队列。

###循环###

把循环选项设成一个整数，指定了一个动画在被调用时的属性映射表中的值与调用前元素的这些值之间应交替的次数：

    $element.velocity({ height: “10em” }, { loop: 2 });

如果元素的初始高度是5em，它的高度会在5em和10em之间交替两次。

如果`begin`和`complete`选项在一个循环的调用中被使用，它们会被各触发一次-分别在最开始和整个循环队列的终点；它们不会在每次循环交替中被重复触发。

你也可以传递`true`来触发无限循环，而不是传递一个整数：

    $element.velocity({ height: “10em” }, { loop: true });
    
无限循环忽略了`complete`回调，因为它们不会自然结束。然而，它们可以通过Velocity的`stop`命令被手动结束：
  
    $element.velocity(“stop”);
    
非无限循环对动画队列是有用的，不然它们将需要重复链式动画的代码。举个例子，如果你想要让一个元素弹上弹下两次（也许是警告用户有一条新消息在等待他们），没有优化的代码看起来大概是这样：

    $element
      // 假定translateY开始时为“0px”
      .velocity({ translateY: “100px” })
      .velocity({ translateY: “0px” })
      .velocity({ translateY: “100px” })
      .velocity({ translateY: “0px” });
      
更紧凑且易维护的代码版本看起来大概是这样：

    // 重复（循环）这段动画两次
    $element.velocity({ translateY: “100px” }, { loop: 2 });
    
有了这个优化的版本，如果你已经在心里想好了最大值应该被改变多少（当前是100px），你只需要在一部分代码中更改最大值。如果在你的代码中有很多这种重复的例子，那么循环对你的工作流多么有益，是显而易见的。

无限循环对加载指示器有巨大的帮助，加载指示器一般无限循环动画直到数据被加载完成。

首先，通过使加载指示器元素的透明度在可见和不可见之间无限循环，令其表现为有节奏的闪动：

    // 假定透明度开始时是1（完全可见）
    $element.velocity({ opacity: 0 }, { loop: true });
    
然后，一旦数据结束加载，你可以停止动画，然后隐藏这个元素：

    $element
      // 首先停止无限循环
      .velocity(“stop”)
      // ... 所以你可以对元素施加一个新动画，
      // 你可以施加它来使元素变回不可见。
      .velocity({ opacity: 0 });
      
###延时

以毫秒指定延时选项,来在动画开始之前插入一个暂停。延时选项的目标是把动画的计时逻辑完整保留在Velocity中-与在一个Velocity动画开始时依赖使用jQuery的$.delay()函数来改变相反：

    //在进行动画使透明度变为0之前等待100ms
    $element.velocity({ opacity: 0 }, { delay: 100 });
    
你可以把loop选项和delay选项一起设定来创建一个循环交替间的暂停：

    // 循环四次，在每次循环前等待100ms
    $element.velocity({ height: “+=50px” }, { loop: 4, delay: 
    100 });

###显示与可见度

Velocity的显示与可见度选项与它们的CSS同仁直接对应，并且接收同样的值，包括：“none”，“inline”，“inline-block”，“block”，“flex”，等等。另外，Velocity允许“auto”值，这指定`display`属性为元素的默认值。（作为参考，a和span标签默认为“inline”，而div和大部分其他元素默认为“block“）。Velocity的可见度选项，像它的CSS同仁一样，接收”hidden“，”visible“，和”collapse“值。

在Velocity中，当`display`选项被设为”none“(或可见度被设为”hidden“)，一旦动画完成，元素的CSS属性即被相应地设置。这有效地使元素在动画完成时被隐藏，并且在与将元素的透明度变为0的动画联合使用时很有用（这里的意图是将一个元素淡出至页面外）：

    // 使一个元素的透明度渐变为0，然后把它移出页面文档流
    $element.velocity({ opacity: 0 }, { display: “none” });
    
*注意*

上面的代码有效地替换了jQuery中的等效代码：

    $element
            .animate({ opacity:0 })
            .hide();
            
            
*快速回顾：可见度与显示*

以供参考，CSS`display`属性指定了一个元素如何影响它周围的元素以及被它包含的元素的定位。对比之下，CSS`visibility`属性仅仅影响一个元素是否能被看见。如果一个元素被设为"visibility:hidden",它将继续在页面中占据空间，但是这个空间将简单地表现为一个空间隔-这个元素的每一部分都是不可见的。作为替代，如果一个元素被设为“display:none”，这个元素将完全从页面文档流中被移除，并且所有在其中或环绕它的元素将填补被移除元素的空间，好像这个元素从未存在过。

注意，你可以设置元素的visibility为”hidden“来简单地把元素同时标记为不可见和无法交互，而不是把这个元素移出页面文档流。当你想隐藏一个继续在页面上占位的元素时这很有用：

    // 将一个元素淡入到opacity:0，然后让它变得无法交互
    $element.velocity({ opacity: 0 }, { visibility: “hidden” });
    
现在，让我们考虑相反方向的动画（显示元素而不是隐藏元素）：当`diaplay`或`visibility`被设为”none“或”hidden“之外的值，这个值会在动画开始前被设置，因此元素在即将到来的动画过程中是可见的。换句话说，你正在取消之前元素被移出视图时发生的隐藏过程。

以下，`display`在元素开始淡入之前被设为”block“：

    $element.velocity({ opacity: 1 }, { display: “block” });


这有效地替换了等效的jQuery代码：

    $element
      .show()
      .animate({ opacity: 0 });
      
*提示*

查看Velocity动画选项的完整概述，请查阅Velocity.org的文档。

*包含动画逻辑*

加上Velocity的`delay`选项，Velocity对CSS`display`和`visibility`设定的包含允许动画逻辑被完全保留在Velocity中。在生产环境代码中，每当一个元素被淡入或淡出视图时，几乎总伴随着`display`和`visibility`上的改变。借助像这样的Velocity简写帮助你保持你的代码干净且易于维护，因为这样对外部jQuery函数的依赖更少，并且避免了重复使用通常会使动画逻辑臃肿的辅助函数。

注意Velocity包括了以上演示的切换透明度动画的简写方式。它们的功能和jQuery的`fadeIn
`及`fadeOut`函数一模一样。你仅需传递相应地传递命令给Velocity作为第一个参数，并且，如果想要的话，你可以传入一个选项对象，像往常一样。

    $element.velocity(“fadeIn”, { duration: 1000 });
    $element.velocity(“fadeOut”, { duration: 1000 });


##使用Velocity：附加特性

附加的Velocity.js特性中值得注意的包括：回退命令，滚动，颜色，和变形（平移，旋转，和缩放）。

###回退命令

传递“reverse”作为Velocity的第一个参数，来使元素发生动画返回至上一个Velocity调用前的值。`reverse`命令和一个标准的Velocity命令表现一样；它可以带有参数并且会和其他链式Velocity调用一起被加入队列。

回退默认设置了元素的上一个Velocity的调用中使用的选项（时长，缓动，等等）。然而，你可以传递一个新选项对象来覆写这些选项：

    // 使用上一个Velocity调用的选项来施加动画返回初始值
    $element.velocity(“reverse”);
    
或

    // 做和上面一样的事，不过把上一个Velocity调用的时长值替换为2000ms
    $element.velocity(“reverse”, { duration: 2000 });
    
*注意*

前一个调用的`begin`和`complete`选项被`reverse`命令忽视了；`reverse`从不重复调用回调函数。

###滚动

传递“scroll”作为Velocity的第一个参数来滚动浏览器至一个元素的顶部。`scroll`命令和一个标准的Velocity调用表现一模一样；它可以带有参数并且会和其他链式Velocity调用一起被加入队列：

    $element
      .velocity(“scroll”, { duration: 1000, easing: “spring” })
      .velocity({ opacity: 1 });

这使用1000ms的时长和“spring”缓动将浏览器滚动到元素的顶部。然后，一旦元素被滚动进入视窗，它会完全淡入。

为了向一个父元素有滚动条的元素滚动，你可以使用`container`选项，它接收一个jQuery对象或者一个原始的元素。注意CSS`position`属性必须被设为`relative`,`absolute`,或者`fixed`中的一个-`static`不会起作用。

    // 滚动元素进入$(“#container”)元素的视图中
    $element.velocity(“scroll”, { container: $(“#container”) });
    
在两种情况中-不管滚动是相对于浏览器窗口还是相对于一个父元素-滚动命令总是被调用在*正被滚动进入视窗*的元素上。

默认情况下，滚动发生在y轴。传入`axis：x`选项来水平滚动来取代垂直滚动：

    // 滚动浏览器到目标div的左边缘
    $element.velocity(“scroll”, { axis: “x” });
    
最后，滚动命令还独特地接收一个以px设定的`offset`选项，它偏移了目标滚动位置：

    // 滚动到距离元素上边缘上方50px的位置
    $element.velocity(“scroll”, { duration: 1000, offset: “-50px” });
    // 滚动到距离元素上边缘下方250px的位置
    $element.velocity(“scroll”, { duration: 1000, offset: “250px” });
    
###颜色

Velocity支持这些CSS属性：`color`,`backgroundColor`, `borderColor`,和`outlineColor`的颜色动画。在Velocity中，颜色属性只接收16进制，举个例子，#000000（黑色）或#e2e2e2（浅灰）。为了达到颗粒度更小的颜色控制，你可以对颜色属性的单个红，绿，和蓝分量进行动画，也包括alpha通道分量。红，绿和蓝的数值范围在0到255之间，alpha通道（等同于透明度）的范围在0到1之间。

参考以下例子中的行内注释：

    $element.velocity({
      // 施加动画使背景颜色变到以十六进制表示的黑色
      backgroundColor: “#000000”,
      // 同步地施加动画使背景的alpha分量（透明度）变到50%
      backgroundColorAlpha: 0.5,
      // 也对元素的文本颜色的red分量施加动画使其变为总量的一半
      colorRed: 125
    });
    
###变形

CSS变形属性对在2D和3D空间的元素施加平移，缩放，和旋转操作。它包括很多子组件，其中Velocity支持以下几种：

+`translateX`:沿x轴移动一个元素
+`translateY`:沿y轴移动一个元素
+`rotateZ`:沿z轴旋转一个元素（在2D表面上实际为顺时针或逆时针）
+`rotateX`:沿x轴旋转一个元素（在3D空间里实际为移向用户或远离用户）
+`rotateY`:沿y轴旋转一个元素（在3D空间里实际为向左移动或向右移动）
+`scaleX`:增加一个元素的宽度值
+`scaleY`:增加一个元素的高度值

在Velocity中，你可以在一个属性对象中以单独属性来施加这些组件带来的动画效果：

    $element.velocity({
      translateZ: “200px”,
      rotateZ: “45deg”
    });

##使用Velocity：不使用jQuery（中级）

如果你是一个宁愿不借助jQuery的帮助来使用JavaScript工作的中级开发者，你将会高兴地得知Velocity也可以在jQuery不出现在页面上的时候工作。相应地，目标元素被直接传递入动画调用作为第一个参数，而不是把一个动画调用链接到一个jQuery元素对象上-就像本章之前的示例：

    Velocity(element, { opacity: 0.5 }, 1000); // Velocity
    
就算Velocity脱离jQuery被使用，它也保持和jQuery的$.animate()一样的语法；区别在于所有的参数都被向右移动来腾出位置以便在首位传入目标元素。另外，全局Velocity对象而不是具体的jQuery元素对象被用于调用动画。

当你脱离jQuery使用Velocity时，你不再对jQuery对象施加动画，而是原生文档对象模型（DOM）元素。原生DOM元素可以通过以下函数获取：

+`document.getElementByID()`：用ID属性获取一个元素
+`document.getElementsByTagName()`获取带有特定标签名的所有元素
+`document.getElementsByClassName()`获取带有特定CSS类的所有元素
+`document.querySelectorAll()`这个函数和jQuery的选择引擎的作用一模一样

让我们进一步探索`document.querySelectorAll()`，因为它可能将成为你在不借助jQuery帮助时选择元素的利器。（这是一个性能强大的且被众浏览器广泛支持的函数。）使用jQuery的元素选择器语法，你可以简单地传递给`querySelectorAll`一个CSS选择器（和你在样式表中用来选择目标元素的选择器一样），并且它将以一个数组的形式返回所有符合的元素：

    document.querySelectorAll(“body”); // 获取body元素
    document.querySelectorAll(“.squares”); // 获取所有带“square”类的元素     
    document.querySelectorAll(“div”); // 获取所有div
    document.querySelectorAll(“#main”); //  获取所有id为“main”的元素 
    document.querySelectorAll(“#main div”); // 获取所有id为“main”的元素中的
    div
        
如果你把这些查找之一的结果赋值给一个变量，随后你可以重复使用这个变量来对目标元素施加动画：

    // 获取所有元素
    var divs = document.querySelectorAll(“div”);
    // 对所有div施加动画
    Velocity(divs, { opacity: 0 }, 1000);
    
因为你不再拓展jQuery元素对象，你可能在想如何把元素一个个链接起来，像这样：

    // 它们彼此链接
    $element
       .velocity({ opacity: 0.5 }, 1000)
       .velocity({ opacity: 1 }, 1000);
       
为了不借助jQuery来再现这个模式，简单地把一个函数接着另一个函数调用：

    // 对同样地元素施加的动画彼此自动链接起来
    Velocity(element, { opacity: 0 }, 1000);
    Velocity(element, { opacity: 1 }, 1000);
    
###结语

现在你已经有了对使用JavaScript进行web动画的好处的认识，加上对Velocity基础的一些掌握，你已经准备好去探索专业动画设计之下的迷人理论基础。



























