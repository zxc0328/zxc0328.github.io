---
title: "ES6探秘-Classes"
date: 2015-10-22 09:23:57
tags: 
- ES6
categories: 
- ES6
---

ES6中增加了一些新特性，但从底层的角度来说，只是一些语法糖。  
但是就我个人来说，如果不了解这些语法糖的本质，是用不安心的。那我们要如何揭开这些语法糖的真实面目呢？  
Babel to the rescue！ Babel是一款将ES6代码转换为ES5代码的编译器，从而让我们可以无视浏览器的支持，直接享受ES6的新特性。同时，我们也可以通过研究Babel编译出的ES5代码，来揭开ES6的面纱。

<!--more-->

###ES6 Classes
ES6中的Classes是在Javascript现有的原型继承的基础上引入的一种语法糖。Class语法并没有引入一种新的继承模式。它为对象创建和继承提供了更清晰，易用的语法。  

我们用class关键字来创建一个类，constructor关键字定义构造函数，用extends关键字来实现继承，super来实现调用父类方法。

好，下面是一个ES6 class语法的完整例子：  

<pre class="prettyprint">
//定义父类View
class View {
  constructor(options) {
    this.model = options.model;
    this.template = options.template;
  }

  render() {
    return _.template(this.template, this.model.toObject());
  }
}
//实例化父类View
var view = new View({
  template: 'Hello, <%= name %>'
});
//定义子类LogView，继承父类View
class LogView extends View {
  render() {
    var compiled = super.render();
    console.log(compiled);
  }
}
</pre>  

这段简短的代码就用到了上述的几个关键词。class语法的确的简洁明确，借鉴了主流OO语言的语法，更易于理解。  

然而我在用这段代码时，又有些犹豫。这还是我熟悉的js原型继承吗，这真的是同一种继承模式的一个语法糖吗？  

真相究竟是如何呢？我们就拿babel编译之后的代码作为切入口，来看看ES6 class语法的本质。

下面是上述ES6代码用babel编译之后的结果：
<pre class="prettyprint" style="height:400px;overflow-y:scroll">
'use strict';
var _get = function get(_x, _x2, _x3) {
    var _again = true;
    _function: while (_again) {
        var object = _x,
            property = _x2,
            receiver = _x3;
        desc = parent = getter = undefined;
        _again = false;
        if (object === null) object = Function.prototype;
        var desc = Object.getOwnPropertyDescriptor(object, property);
        if (desc === undefined) {
            var parent = Object.getPrototypeOf(object);
            if (parent === null) {
                return undefined;
            } else {
                _x = parent;
                _x2 = property;
                _x3 = receiver;
                _again = true;
                continue _function;
            }
        } else if ('value' in desc) {
            return desc.value;
        } else {
            var getter = desc.get;
            if (getter === undefined) {
                return undefined;
            }
            return getter.call(receiver);
        }
    }
};

var _createClass = (function() {
    function defineProperties(target, props) {
        for (var i = 0; i < props.length; i++) {
            var descriptor = props[i];
            descriptor.enumerable = descriptor.enumerable || false;
            descriptor.configurable = true;
            if ('value' in descriptor) descriptor.writable = true;
            Object.defineProperty(target, descriptor.key, descriptor);
        }
    }
    return function(Constructor, protoProps, staticProps) {
        if (protoProps) defineProperties(Constructor.prototype, protoProps);
        if (staticProps) defineProperties(Constructor, staticProps);
        return Constructor;
    };
})();

function _inherits(subClass, superClass) {
    if (typeof superClass !== 'function' && superClass !== null) {
        throw new TypeError('Super expression must either be null or a function, not ' + typeof superClass);
    }
    subClass.prototype = Object.create(superClass && superClass.prototype, {
        constructor: {
            value: subClass,
            enumerable: false,
            writable: true,
            configurable: true
        }
    });
    if (superClass) Object.setPrototypeOf ? Object.setPrototypeOf(subClass, superClass) : subClass.__proto__ = superClass;
}

function _classCallCheck(instance, Constructor) {
    if (!(instance instanceof Constructor)) {
        throw new TypeError('Cannot call a class as a function');
    }
}

var View = (function() {
    function View(options) {
        _classCallCheck(this, View);
			this.model = options.model;
        this.template = options.template;
    }
    _createClass(View, [{
        key: 'render',
        value: function render() {
            return _.template(this.template, this.model.toObject());
        }
    }]);
    return View;
})();

var LogView = (function(_View) {
    _inherits(LogView, _View);
    function LogView() {
        _classCallCheck(this, LogView);
        _get(Object.getPrototypeOf(LogView.prototype), 'constructor', this).apply(this, arguments);
    }
    _createClass(LogView, [{
        key: 'render',
        value: function render() {
            var compiled = _get(Object.getPrototypeOf(LogView.prototype), 'render', this).call(this);
            console.log(compiled);
        }
    }]);
    return LogView;  
    
})(View);
</pre>

这段代码很长，我们只关注里面的函数，可以得到它的结构如下:

<pre class="prettyprint" style="height:400px;overflow-y:scroll">
//用于得到原型链上属性的方法的函数
var _get = function get(_x, _x2, _x3) {
	//······
}

//用于创建对象的函数
var _createClass = (function() {
	 //内部函数，定义对象的属性
    function defineProperties(target, props) {
    	//······
    }
    return function(Constructor, protoProps, staticProps) {
        if (protoProps) defineProperties(Constructor.prototype, protoProps);
        if (staticProps) defineProperties(Constructor, staticProps);
        return Constructor;
    };
})();

//用于实现继承的函数
function _inherits(subClass, superClass) {
		//······
        subClass.prototype = Object.create(superClass && superClass.prototype, {
        constructor: {
            value: subClass,
            enumerable: false,
            writable: true,
            configurable: true
        }
});

var View = (function() {
	//······
    return View;
})();

var LogView = (function(_View) {
	//······
})(View);
</pre>


###View类的实现

我们从一个View类的创建开始分析
<pre class="prettyprint">
class View {
  constructor(options) {
    this.model = options.model;
    this.template = options.template;
  }
  render() {
    return _.template(this.template, this.model.toObject());
  }
}

//ES5代码
var View = (function() {
    function View(options) {
        _classCallCheck(this, View);
			this.model = options.model;
        this.template = options.template;
    }
    _createClass(View, [{
        key: 'render',
        value: function render() {
            return _.template(this.template, this.model.toObject());
        }
    }]);
    return View;
})();

</pre>
我们从编译之后的代码中可以看出，View是一个IIFE，里面是一个同名的函数View，这个函数经过  `_createClass()`函数的处理之后，被返回了。所以我们得出的第一点结论就是，**ES6中的class实际就是函数**。当然这点在各种文档中已经明确了，所以让我们继续分析。 

IIFE中的同名的View实际上就是我们在ES5的原型继承中使用的构造函数，所以ES6中的class是对ES5中的构造函数的一种包装。我们发现，在class中设定的属性被放在ES5的构造函数中，而方法则以键值对的形式传入一个`_createClass()`函数中。那么这个`_createClass()`函数又制造了什么魔法呢？  

<pre class="prettyprint">
var _createClass = (function() {
    function defineProperties(target, props) {
        for (var i = 0; i < props.length; i++) {
            var descriptor = props[i];
            descriptor.enumerable = descriptor.enumerable || false;
            descriptor.configurable = true;
            if ('value' in descriptor) descriptor.writable = true;
            Object.defineProperty(target, descriptor.key, descriptor);
        }
    }
    return function(Constructor, protoProps, staticProps) {
        if (protoProps) defineProperties(Constructor.prototype, protoProps);
        if (staticProps) defineProperties(Constructor, staticProps);
        return Constructor;
    };
})();
</pre> 

`_createClass`也是一个IIFE，有一个内部的函数`defineProperties`，这个函数遍历属性的描述符，进行描述符的默认设置，最后使用`Object.defineProperty()`方法来写入对象的属性。IIFE的renturn部分有两个分支，一个是针对一个类的原型链方法，一个是静态方法，我们看到原型链方法被写入构造函数的原型对象里，**而静态方法则被直接写入构造函数里，因此我们不用实例化对象就可以直接调用一个类的静态方法了**。
>js中的函数是对象，Function构造函数的prototype指向Object.prototype，因此可以写入属性

###类继承的实现

OK，到目前我们已经搞清了ES6的class关键字是如何工作的，那么ES6中的继承有是如何实现的呢？下面让我们看看`_inherits()`函数。

<pre class="prettyprint">
function _inherits(subClass, superClass) {
    if (typeof superClass !== 'function' && superClass !== null) {
        throw new TypeError('Super expression must either be null or a function, not ' + typeof superClass);
    }
    subClass.prototype = Object.create(superClass && superClass.prototype, {
        constructor: {
            value: subClass,
            enumerable: false,
            writable: true,
            configurable: true
        }
    });
    if (superClass) Object.setPrototypeOf ? Object.setPrototypeOf(subClass, superClass) : subClass.__proto__ = superClass;
}
</pre>

`_inherits()`函数的关键部分便是`subClass.prototype = Object.create(···)`。通过`Object.create()`方法来指定新创建对象的原型，由此省去了对父类构造函数的处理，达到了简单的原型继承效果。

然后我们来看看创建`LogView`类的代码：
<pre class="prettyprint">
//ES6
class LogView extends View {
  render() {
    var compiled = super.render();
    console.log(compiled);
  }
}
//ES5
var LogView = (function(_View) {
    _inherits(LogView, _View);
    function LogView() {
        _classCallCheck(this, LogView);
        _get(Object.getPrototypeOf(LogView.prototype), 'constructor', this).apply(this, arguments);
    }
    _createClass(LogView, [{
        key: 'render',
        value: function render() {
            var compiled = _get(Object.getPrototypeOf(LogView.prototype), 'render', this).call(this);
            console.log(compiled);
        }
    }]);
    return LogView;
})(View);
</pre>

`LogView`类和`View`类的最大不同便是增加了一个`_get()`函数的调用，我们仔细看这个`_get()`函数会发现它接收几个参数，子类的原型，"constructor"标识符，还有`this`。再看下面对`super.render()`的处理，同样是用`_get()`函数来处理的。再看`_get()`函数的源码，就可以发现`_get()`函数的作用便是遍历对象的原型链，找出传入的标识符对应的属性，把它用`apply`绑定在当前上下文上执行。

大家是不是对这种继承模式似曾相识呢？对了，这就是所谓的“构造函数窃取”。当我们需要继承父类的属性时，在子类的构造函数内部调用父类构造函数，在加上`Object.create()`大法，然后就可以继承父类的所有属性和方法了。

###结语
以上就是笔者对ES6中的class做的一些解读，依据是babel的编译结果。今天刚好看到getify大神的一篇[博客](http://blog.getify.com/arrow-this/)，将的是ES6中的箭头函数其实并不是一个语法糖，而是一种新的不带this的函数。他还进一步说了，错误的过程得到正确的结果，会导致很多后续的问题。

而本文的结论是建立在class是一种语法糖的基础之上的，根据MDN的文档，我们应该可以确认这个结论是可靠的。而ES6中的特性，如箭头函数，并不都是语法糖，因此在后续的ES6探秘文章中，我们将用其他的途径，来揭示ES6种种神奇魔法的秘密。