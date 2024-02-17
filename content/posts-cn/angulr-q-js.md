+++
title = "Angular源码分析:$q"
date = 2015-08-24 15:27:51
tags = ['Angular']
categories = ['Angular']
hiddenInHomeList = true
+++
---
### 概述
Augular中的$q服务提供了Promise的实现，这个服务叫做“q”是因为它是[Kris Kowal's Q](https://github.com/kriskowal/q)的一种植入。  

从Augular中$q服务的文档中可以了解到，这个服务有两种用法，一种在某种程度上和ES6的Promise规范相似，而另一种则和Q以及jQuery的Deferred接口类似。

<!--more-->

因此在Angular中$q的具体代码实现有两种，一种是将$q作为一个构造函数，并且接收一个`resolver()`函数作为唯一参数。这和ES6的原生实现相似：

<pre class="prettyprint">
// for the purpose of this example let's assume that variables `$q` and `okToGreet`
// are available in the current lexical scope (they could have been injected or passed in).

function asyncGreet(name) {
  // perform some asynchronous operation, resolve or reject the promise when appropriate.
  return $q(function(resolve, reject) {
    setTimeout(function() {
      if (okToGreet(name)) {
        resolve('Hello, ' + name + '!');
      } else {
        reject('Greeting ' + name + ' is not allowed.');
      }
    }, 1000);
  });
}

var promise = asyncGreet('Robin Hood');
promise.then(function(greeting) {
  alert('Success: ' + greeting);
}, function(reason) {
  alert('Failed: ' + reason);
});
</pre>

而另一种实现则是更为传统的CommonJS风格的实现，将Promise描述成一个接口，来和一个代表了未来结果的异步请求发生交互。

代码如下：  

<pre class="prettyprint">
// for the purpose of this example let's assume that variables `$q` and `okToGreet`
// are available in the current lexical scope (they could have been injected or passed in).

function asyncGreet(name) {
  var deferred = $q.defer();

  setTimeout(function() {
    deferred.notify('About to greet ' + name + '.');

    if (okToGreet(name)) {
      deferred.resolve('Hello, ' + name + '!');
    } else {
      deferred.reject('Greeting ' + name + ' is not allowed.');
    }
  }, 1000);

  return deferred.promise;
}

var promise = asyncGreet('Robin Hood');
promise.then(function(greeting) {
  alert('Success: ' + greeting);
}, function(reason) {
  alert('Failed: ' + reason);
}, function(update) {
  alert('Got notification: ' + update);
});

</pre>


##源码分析：deferred API
因为$q服务中的ES6形式实现实际上是在deferred api的基础上的一种封装实现，所以我们先来看看deferred API的源代码。  
这部分代码的结构看起来是这样的：

<pre class="prettyprint">
//deferred method: return a instance of Deferred object
var deferred = function() {
    return new Deferred();
};

//construct function of Promise object
function Promise() {
    this.$$state = { status: 0 };
}

//then, catch, and finally method of Promise object
extend(Promise.prototype, {
    then: function(onFulfilled, onRejected, progressBack) {
      //...
    },

    "catch": function(callback) {
      //...
    },

    "finally": function(callback, progressBack) {
      //...
    }
});


//construct function of Deferred object, the promise object is a method 
//of Deferred object. And resolve, reject, notify apis are binded here.
function Deferred() {
    this.promise = new Promise();
    //Necessary to support unbound execution :/
    this.resolve = simpleBind(this, this.resolve);
    this.reject = simpleBind(this, this.reject);
    this.notify = simpleBind(this, this.notify);
}



extend(Deferred.prototype, {
    resolve: function(val) {
    	//...
    },

    $$resolve: function(val) {
    	//...
    },

    reject: function(reason) {
    	//...
    },

    $$reject: function(reason) {
    	//...
    },

    notify: function(progress) {
   		//...
    }
  });

</pre>


###具体分析：Deferred.promise.then()

<pre class="prettyprint">
then: function(onFulfilled, onRejected, progressBack) {
	   //check if at least one para was passed in, otherwise return.
      if (isUndefined(onFulfilled) && isUndefined(onRejected) && isUndefined(progressBack)) {
        return this;
      }
      //instantiate a new Deferred object as result
      var result = new Deferred();
      
	   //instantiate $$state.pending
      this.$$state.pending = this.$$state.pending || [];
      //push paras of this into $$state.pending array
      this.$$state.pending.push([result, onFulfilled, onRejected, progressBack]);
      //if $$state.status changed, add this.$$state to the process 
      queue
      if (this.$$state.status > 0) scheduleProcessQueue(this.$$state);
		
	  //return the new Deferred object instance's promise object
      return result.promise;
}
</pre>

then()方法接收三个参数，promise条件满足时执行的函数`onFulfilled`，被拒绝时的回调函数`onRejected`，以及一个通知用回调函数`progressBack`。  
这里的逻辑是`then()`方法接收参数之后，实例化一个新`Deferred`对象，并将参数推入pending数组中。
关于then,有趣的一点在于then的链式调用的实现，在分析这点之前，我们把目光移到这个api的大局上来，看看promise的状态变化以及状态转移是如何实现的。

###具体分析：Deferred.resolve()

在promise对象内部，收到请求后具体执行状态转移的便是`resolve()`和`reject()`函数了，我们还可以向其中传递一个唯一的参数，来作为`resolve()`的value或是`reject()`的reason。因为这两个方法原理大致类似（在源码中这两个方法的相关代码有较大的不同，应该和这个两种行为不同的结果有关），所以我们就以`resolve()`方法为例来说明。

<pre class="prettyprint">
resolve: function(val) {
		//if $$state not equals 0, which means the state had already 
		changed, return.
      if (this.promise.$$state.status) return;
      //reject if val is this.promise its self.
      if (val === this.promise) {
        this.$$reject($qMinErr(
          'qcycle',
          "Expected promise to be resolved with value other than itself '{0}'",
          val));
      } else {
      //resolve val with inner method $$resolve
        this.$$resolve(val);
      }

    },

$$resolve: function(val) {
      var then, fns;
	  //warp the functions in order to be called only once
      fns = callOnce(this, this.$$resolve, this.$$reject);
      try {
        if ((isObject(val) || isFunction(val))) then = val && val.then;
        if (isFunction(then)) {
          this.promise.$$state.status = -1;
          then.call(val, fns[0], fns[1], this.notify);
        } else {
          this.promise.$$state.value = val;
          this.promise.$$state.status = 1;
          scheduleProcessQueue(this.promise.$$state);
        }
      } catch (e) {
        fns[1](e);
        exceptionHandler(e);
      }
}
</pre>

`resolve()`方法接收一个值val，这个值可能是对象/函数，也有可能是普通的数值或是字符串。`resolve()`方法首先判断`Deferred.promise.$$state.status`的值是否不为0，如果不为0则说明状态已经转移了，那么就不再继续执行。如果一切正常，在执行`resolve()`方法时，`Deferred.promise.$$state.status`的值为初始的0，然后val被传入一个内部函数`$$resolve()`,来进行真正的处理。`$$resolve()`函数首先调用`callOnce()`函数来确保自己只被调用一次，然后判断val的数据类型，如果是对象或者是函数，则将val赋值给then，其中`then = val && val.then;`这个语句用来将`resolve()`中可能传入的promise对象的`then()`方法赋值给then。下一步，如果then是函数，也就是说传入的val是一个promise对象的话，则将`Deferred.promise.$$state.status`的值设为-1，之后`then.call(val, fns[0], fns[1], this.notify);`这是将fns[0], fns[1], this.notify三个函数作为参数传入了这个promise对象的then方法中，其中前两者便是经过了`callOnce()`函数处理的`this.$$resolve`和`this.$$reject`函数，这一步设置了被传入的promise对象的pending list。  
如果val不是函数，那么接下去的逻辑便很容易理解了，`this.promise.$$state.value`被赋值为val的值，`this.promise.$$state.status`的值被设置为1，1这个状态码便代表了promise对象目前处于**fulfilled**状态。  
接下去调用`scheduleProcessQueue()`函数，并传入`this.promise.$$state`对象。这个函数实际是调用了`ProcessQueue()`函数，不过加入了一些angular内部的检查机制，来保证函数调用和angular内部运行的同步。而`ProcessQueue()`函数则真正执行`then()`方法推入到pending list中的回调函数。  

下面我们简单的来看看ProcessQueue函数:

<pre class="prettyprint">
function processQueue(state) {
    var fn, deferred, pending;

    pending = state.pending;
    state.processScheduled = false;
    state.pending = undefined;
    for (var i = 0, ii = pending.length; i < ii; ++i) {
      deferred = pending[i][0];
      fn = pending[i][state.status];
      try {
        if (isFunction(fn)) {
          deferred.resolve(fn(state.value));
        } else if (state.status === 1) {
          deferred.resolve(state.value);
        } else {
          deferred.reject(state.value);
        }
      } catch (e) {
        deferred.reject(e);
        exceptionHandler(e);
      }
    }
  }
</pre>

`processQueue()`函数简单地将当前promise对象的pending list按传入的状态码进行处理。`fn = pending[i][state.status];`中，status为1，那么就调用pending list的第二项，也就是onFulfilled情况下的回调函数。之后
`deferred.resolve(fn(state.value));`中，实际上执行了一次`eval(fn(state.value))`,回调函数就是在这里被执行的，之后`deferred.resolve(val)`便将回调函数处理之后的值传给了新创建的deferred对象。


###实例分析：Deferred.promise.then()的链式调用机制

<pre class="prettyprint">
          var Deferred = $q.defer();
          var promise1 = Deferred.promise;
          var promise2 = promise1.then(function (data) {
				 return data  + 1;
          });
          promise2.then(function (val) {
               console.log(val);
          }); 
          Deferred.resolve(10);
</pre>

这段代码的预期结果是promise2的回调函数会输出11。让我们来看看这是怎样实现的。

首先，promise1的`then()`方法被执行,回调函数被推入pending list中，一个新的`deferred`对象被创建,`deferred.promise`对象被返回，我们将这个新创建的promise对象赋值给promise2。同样的，promise2的`then()`方法也被执行。所以我们把promise1的pending list称谓list1，promise2的pending list称为list2。

之后，`Deferred.resolve(10);`被执行，在这里这是同步的，但一般来说这是异步执行的。此时执行resolve()方法，后数据被传入内部的`$$resolve()`方法，因为val是数值，因此`$$state`对象的value被设置为10，而状态码也被设置为相应的1，代表解析成功。  

之后的任务交给了`processQueue()`函数，因为list1中的第二个参数是我们通过`then()`方法传入的回调函数，这个函数执行`deferred.resolve(fn(state.value));`，回调函数执行，并返回值11，这里的deferred对象便是promise1的`then()`方法创建的新deferred对象，也就是list1中的deferred对象，是promise2的宿主对象。  

所以`deferred.resolve(fn(state.value));`的执行，执行了又一次`resolve()`函数，只是这次的上下文换成了promise2所在的上下文，那么接下去的流程便和promise1发生的一致了。最后list2中的回调执行，输出值11。
  
让我们总结一下，`then()`方法实际上是负责在运行时挂载回调函数列表，而`resolve()`函数则负责异步触发函数执行。链式调用是通过返回新promise对象，并配合`then()`方法的同步加载回调以便获取新上下文来实现的。

tbc···
****

> [参考-A look at Angular's promise implementation](http://olim7t.github.io/2013/08/19/angular-promise-implementation.html)



