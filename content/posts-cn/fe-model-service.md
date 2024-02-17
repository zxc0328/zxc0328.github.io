---
title: "简单的前端网络层Service封装"
date: 2017-08-16 14:01:31
tags: 
- FE
categories: 
- FE
---

我们编写前端组件时，常常需要拉取数据。最原始的办法就是在组件中调用网络库去请求数据。但这样有一些问题：发送请求的一些代码需要**重复的编写**。

<!-- more -->

比如fetch的`json()`方法，拿到返回数据之后进行错误处理的代码。一个典型的使用场景是这样的：



```
fetch('/api/v2.0' + this.Url + '/?page=' + this.page_num)
  .then(res => {
    return res.json()
  })
  .then(res => {
    if (res.code === 200) {
      this.items = res.blogs
      this.pages_count = res.pages_count
      this.page_num = res.page
      this.blog_num = res.blog_num
    }else {
      util.message("Error:", res.message)
    }
  })
} 

```



而且在发送请求这个操作中，带有请求的URL等等和组件业务逻辑无关系不大的数据，我们希望可以集中管理这些请求的路由，并且集中处理错误。

下面就介绍一种最简单的前端网络层封装，我将其称为Service。



### 简单的Fetch封装

为了避免在每次调用fetch时都要设置各种header和参数，我们可以对fetch做一个简单的封装：

```
function Fetch(url, opt = {}) { 

 // 设置请求方法
 opt.method = opt.method || 'GET';
 
 // 处理要发送的数据
 if (opt.data) {
    if (/GET/i.test(opt.method)) {
      url = `${url}&${obj2query(opt.data)}`;
    } else {
      opt.headers = {
        'Accept': 'application/json',
        'Content-Type': 'application/json',
      };
      opt.body = JSON.stringify(opt.data);
    }
  }
  
  return fetch(url, opt)
    .then(response => {  
      return response.json();
    })
}
```

使用示例：

```
Fetch('/api/dash/message/list', {
   method: 'GET',
   data
})
```

上面演示的封装只处理了json格式的body和URL参数。在平时的开发中，我们会遇到更复杂的情况，比如需要特殊的header，或者是需要上传文件等等。我们可以通过给`Fetch`函数传入不同的`opt.type`来达到差异化的处理。有同学会说，在这个函数里根据不同的type写死业务逻辑，是不是不够解耦？其实不是，`Fetch`在这里已经是一个业务逻辑的封装了。

### 错误处理

API请求的错误处理是一个很重要的话题，我们希望在服务端返回错误代码时，前端应用可以优雅的提示这个错误，而不是让应用直接报错。换句话说，我们希望catch这个错误，并且进行处理。

这个错误处理的逻辑如果写在每个`Fetch`请求返回的Promise后面，无疑是不现实的。因为这样会造成大量代码的重复。所以进行错误处理的最佳场所就是刚才我们封装的`Fetch`函数了。

我们在拿到`response`之后先进行判断，然后再返回结果：

```
// Fetch的json返回时，对状态码进行判断，做不同的处理。
// 比如在服务端错误时使用alert或者notification组件进行全局提示。
return response.json().then((json) => { 
  switch (json.code) {
    case 200:
      return json.result;
    case 502:
      util.message(json.message, 'err');
      throw json.message;
  }
}）
```



### Service层封装


`Fetch`其实还是一个比较底层的封装，Service才是前端组件之间调用的逻辑。这个Service类似传统MVC架构中的Model。提供接口并返回数据。一个Service是一组相关业务逻辑接口的集合。

比如一个新闻应用，那首页的feed流是一个Service。单篇文章相关的接口可以放到一个Service。用户相关的接口可以放到一个Serivce。


```
let service = {
  getNews(nid) {
        return fetch(`/api/news/`, {
            method: 'GET',
            data: {
                id: nid,
            }
        });
  },
  
  getNewsList() {
        return fetch(`/api/news/all/`, {
            method: 'GET'
        });
  },

  getComment( nid ) {
		return fetch(`/api/news/${nid}/comment/`, {
			method: 'GET'
		})
	},
	
  sendComment( data ) {
	  return fetch(`/api/news/${data.nid}/comment/`, {
		  method: 'POST',
		  data: data
	  })
  },
}
```

在组件代码中调用Service（如果应用引入了单独的Model层，那就可以在Model层中调用Service）：

```
import NewsService from '../services/news'

module.export = {
  mounted() {
    NewsService.getNews(this.id)
      .then( (result) => {
        this.news = result;
      })
  }
}

```