---
title: "Http Basic Authorization的使用"
date: 2015-11-04 13:03:42
tags: 
- Networking 
categories: 
- Networking
---
###1.Http Basic Authorization的请求格式

关于发起请求的http头格式，大家可以搜到很多相关资料。这里我们采用的方式比较特别，我们用mitmproxy这款http代理软件来拦截我们在终端的通过httpie发出的请求，分析其内容，从而得出Http Basic Authorization的请求格式。

<!-- more --->
首先我们发送一个http POST请求：

<pre class="prettyprint">
http --auth neo1218@yeah.net POST http://121.43.230.104:5000/api/v1.0/token
</pre>

拦截到的http header如下：
![http-header](http://7oxh2b.com1.z0.glb.clouddn.com/blog-http-header.png)

我们看到在http请求的header里有一个`Authorization`字段，内容有`Basic`标识符和一串编码组成。这串编程在经过解析之后，发现是`username:password`的base64编码。

这样就得到了我们Http Basic Authorization的请求格式，在http的请求头中加入字段格式是`Authorization : Basic base64(username:password)`



###2.Http Basic Authorization客户端使用，以ajax为例

下面以web端的ajax技术发起请求，来作为Http Basic Authorization客户端使用方法的实例。

````
var auth = btoa($(".username").val()+":"+$(".password").val());
this.options.userModel.fetch({
		headers:{
			"Authorization":"Basic "+ auth
		}
	})

````
这是backbone中使用model的fetch api进行ajax请求的例子。在请求之前，我们配置headers字段，设置为base64编码后的auth信息。最后发送请求。

这个http请求头是这样的：

![http请求头](http://7oxh2b.com1.z0.glb.clouddn.com/屏幕快照%202015-12-21%20上午3.29.51.png)

ok，到这里我们就成功使用了http basic authorization来向服务器发送请求。

###3.拓展阅读

《Http权威指南》中有关验证的章节

