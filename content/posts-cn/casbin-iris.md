---
title: Iris + Casbin 权限控制实战
date: 2018-05-14 21:17:27
hiddenInHomeList : true
tags:
- Iris
- Go
categories:
- Iris
- Go
---


在木犀的 PaaS 云平台的设计中，需要有一个细粒度比较小的权限控制系统。不同用户对不同的资源，拥有不同的权限。土办法已经不管用了，我们需要更系统，更规范的权限控制系统。本文讲的就是如何将权限控制库 [Casbin](https://github.com/casbin/casbin) 接入 Iris Web 框架。

<!-- more -->


### Iris 中间件机制简介

Iris 这个框架是基于中间件的，和 Nodejs 的 Koa 和 Express 很像。所谓中间件机制，就是一个请求到达之后，会生成一个上下文信息，里面包含了这个请求的一些信息。然后我们依次调用中间件函数，把上下文对象作为参数传入。需要注意是中间件函数的调用是嵌套的，在中间件函数中我们可以调用 `ctx.Next()` 方法，进入下一个中间件函数。当最后一个中间件函数返回时，之前调用过的中间件会依次返回。这个数据流被形象的称为“洋葱模型”。

![](https://raw.githubusercontent.com/zxc0328/for-picgo/master/64c45edcgy1frlkp8fedzj20da0c3t91.jpg)


一个典型的中间件是这样的：

```
func middleware(ctx iris.Context) {
   // get info from context
	requestPath := ctx.Path()

   // set info with context
	ctx.Values().Set("info", shareInformation)
	
	// call next middleware
	ctx.Next()
}
```

我们可以在中间件中读取 ctx 结构，根据上面附带的信息，我们可以做一些针对性的事情。

一个常用的中间件场景就是访问控制。我们可以根据 ctx 上带的用户信息，来查看用户的权限，如果用户没有要访问的资源的权限，我们就拒绝这次访问。比如这样：

```
func auth(ctx iris.Context) {
   // check if user has permission
   if !c.Check(ctx.Request()) {
		ctx.StatusCode(http.StatusForbidden) // Status Forbiden
		ctx.StopExecution() 
		return
	}
	ctx.Next()
}
```



在本文的权限控制的场景下，中间件的作用就是在请求验证失败时，提前返回 403 状态码。

### Casbin 简介

[Casbin](https://github.com/casbin/casbin) 是由北大的一位博士生主导开发的一个基于 Go 语言的权限控制库。支持 ACL，RBAC，ABAC 等常用的访问控制模型。

Casbin 的核心是**一套基于 PERM metamodel (Policy, Effect, Request, Matchers) 的 DSL**。Casbin 从用这种 DSL 定义的配置文件中读取访问控制模型，作为后续权限验证的基础。

一个典型的配置文件如下：

```
# Request definition
[request_definition]
r = sub, obj, act

# Policy definition
[policy_definition]
p = sub, obj, act

# Policy effect
[policy_effect]
e = some(where (p.eft == allow))

# Matchers
[matchers]
m = r.sub == p.sub && r.obj == p.obj && r.act == p.act
```

可以看到这个配置文件主要定义了 Request 和 Policy 的组成结构。Policy effect 和 Matchers 则灵活的多，可以包含一些自定义的表达式。

比如我们要加入一个名叫 root 的超级管理员，就可以这样写：

```
[matchers]
m = r.sub == p.sub && r.obj == p.obj && r.act == p.act || r.sub == "root"
```

又比如我们可以用正则匹配来判断权限是否 match：

```
[matchers]
m = r.sub == p.sub && keyMatch(r.obj, p.obj) && regexMatch(r.act, p.act)
```

Casbin 文档中有[一节](https://github.com/casbin/casbin#examples)展示了 Casbin 所支持的权限控制模型的示例配置。我们可以根据那些例子来打造我们自己的权限控制模型。

有了权限控制模型，我们还需要权限规则。权限规则是若干行数据的集合。每行数据就是一条规则。权限规则的具体格式因权限模型中的定义而异，最简单的 ACL 模型的规则是这样的：

```
p, alice, data1, read
p, bob, data2, write
```

意思就是 alice 可以读 data1，bob 可以写 data2。

### 为 Casbin 适配 Iris 中间件

既然实现权限控制的最佳位置是中间件，我们就需要为 Casbin 写一个 Iris 中间件。社区里的 [Casbin-iris 插件](https://github.com/iris-contrib/middleware/tree/master/casbin)就是一个不错的例子，我们可以以这个插件为基础进行开发。

首先我们来看看这个中间件是如何使用的，这个插件有 Warpper 和 Middleware 两种用法。差别在于 Warpper 方法会在所有路由被调用。而 Middleware 让我们可以控制哪些路由启用权限控制。我们选择 Middleware 方式做示例。打开 [`casbin/_examples/middleware/main.go`](https://github.com/iris-contrib/middleware/blob/master/casbin/_examples/middleware/main.go)，其中核心的几行代码是这样的：


```
var Enforcer = casbin.NewEnforcer("casbinmodel.conf", "casbinpolicy.csv")

func newApp() *iris.Application {
	casbinMiddleware := casbinMiddleware.New(Enforcer)

	app := iris.New()
	app.Use(casbinMiddleware.ServeHTTP)

	app.Get("/", hi)

	app.Get("/dataset1/{p:path}", hi) // p, alice, /dataset1/*, GET

	app.Post("/dataset1/resource1", hi)

	app.Get("/dataset2/resource2", hi)
	app.Post("/dataset2/folder1/{p:path}", hi)

	app.Any("/dataset2/resource1", hi)

	return app
}
```

首先通过 NewEnforcer 方法初始化一个 Casbin Enforcer。NewEnforcer 方法接收两个参数，一个是访问控制模型文件的路径，一个是权限规则文件的路径。

然后调用 casbinMiddleware.New 方法，传入 Casbin Enforcer，进行一些初始化工作。最后调用 `app.Use(casbinMiddleware.ServeHTTP)`，应用中间件。

看起来还挺简单的。但这里存在一个问题，这个中间件是如何拿到鉴权需要的用户信息的呢？这个过程对于开发者是不透明的。我们查看[源码](https://github.com/iris-contrib/middleware/blob/master/casbin/casbin.go)，发现里面有这样的代码：


```
// Username gets the username from the basicauth.
func Username(r *http.Request) string {
	username, _, _ := r.BasicAuth()
	return username
}
```

原来这个中间件假设请求通过 HTTP Basic Auth 方式进行认证。然后从请求的 headers 中获取认证信息。

但在实际生产中，我们认证用户身份的方式有很多种，最常见的就是通过 session 得知用户的身份，或者通过 token 这样的凭证来确定用户的身份。这个中间件如果要使用到生产中去，需要进行一些改动。

以下就是修改过的中间件，为了测试，我在 Username 函数中直接返回了用户名，后续使用时可以在这个函数里进行用户身份的获取。

```
package casbin

import (
	"net/http"

	"github.com/kataras/iris/context"

	"github.com/casbin/casbin"
)

func New(e *casbin.Enforcer) *Casbin {
	return &Casbin{enforcer: e}
}

func (c *Casbin) Wrapper() func(w http.ResponseWriter, r *http.Request, router http.HandlerFunc) {
	return func(w http.ResponseWriter, r *http.Request, router http.HandlerFunc) {
		if !c.Check(r) {
			w.WriteHeader(http.StatusForbidden)
			w.Write([]byte("403 Forbidden"))
			return
		}
		router(w, r)
	}
}

func (c *Casbin) ServeHTTP(ctx context.Context) {
	if !c.Check(ctx.Request()) {
		ctx.StatusCode(http.StatusForbidden) // Status Forbiden
		ctx.StopExecution()
		return
	}
	ctx.Next()
}

type Casbin struct {
	enforcer *casbin.Enforcer
}

// Check checks the username, request's method and path and
// returns true if permission grandted otherwise false.
func (c *Casbin) Check(r *http.Request) bool {
	username := Username(r)
	method := r.Method
	path := r.URL.Path
	return c.enforcer.Enforce(username, path, method)
}

// Username gets the username from the basicauth.
func Username(r *http.Request) string {
   // TODO: get username form db using userId in session or token
	return "alice"
}
```

大家可以用[这里](https://github.com/iris-contrib/middleware/tree/master/casbin/_examples/middleware)的代码、[模型](https://github.com/iris-contrib/middleware/blob/master/casbin/_examples/middleware/casbinmodel.conf)和[规则文件](https://github.com/iris-contrib/middleware/blob/master/casbin/_examples/middleware/casbinpolicy.csv)进行测试（将中间件替换为上面的版本）。如果我们 GET /dataset2/resource2 这个路径，就会返回 403。这说明中间件正常工作了。因为 alice 是没有 /dataset2/resource2 这个资源的 GET 权限的。

### 选择合适的访问控制模型


我选择了 RBAC with domains/tenants 这个模型作为木犀云平台的访问控制模型。PaaS 平台中有服务、应用等多种资源，所以需要按领域模型区分。用户中可以存在超级管理员等角色，所以需要角色。需要注意的是 Casbin 的 RBAC 中的角色其实是一种分组。比如我们可以定义一个叫 admin 的用户，这个用户对所有的资源都有权限规则存在，然后我们可以把其他用户和这个用户分为一组，那这些用户也都有了 admin 用户的权限。


```
[request_definition]
r = sub, dom, obj, act

[policy_definition]
p = sub, dom, obj, act

[role_definition]
g = _, _, _

[policy_effect]
e = some(where (p.eft == allow))

[matchers]
m = g(r.sub, p.sub, r.dom) && r.dom == p.dom && r.obj == p.obj && r.act == p.act
```

这个模型定义中的 g 就是指 group。

示例规则如下：

```
p, admin, domain1, data1, read
p, admin, domain1, data1, write
p, admin, domain2, data2, read
p, admin, domain2, data2, write
g, alice, admin, domain1
g, bob, admin, domain2
```

如上所示，alice 和 bob 分别是 domian1 和 domain2 的管理员。


### Casbin Policy 持久化

Casbin 的 policy 可以保存在一个 csv 文件中。也可以被持久化到数据库中。

所谓的“持久化到数据库中”的意思，就是在数据库中创建一个表，把行数据都存放到数据库中。

对于一个 Web 应用，我们想要的当然是后者。所以我们还需要将 Casbin 和数据库连接起来。

Casbin 支持 gorm 和 xorm 等等常见的 orm。我们以 gorm 为例，Canbin 的 gorm 适配库是 [gorm-adapter](https://github.com/casbin/gorm-adapter)。

接入 gorm 并不是很复杂的事情，其实就是把 NewEnforcer 中的第二个 policy file 参数换成 gorm-adapter 的一个实例就可以。


```
// 自动创建一个数据库，叫 casbin
// 如果需要制定数据库名，可以这样 a := gormadapter.NewAdapter("mysql", "mysql_username:mysql_password@tcp(127.0.0.1:3306)/abc", true)
var a = gormadapter.NewAdapter("mysql", "root:muxi@tcp(127.0.0.1:3306)/") 

var Enforcer = casbin.NewEnforcer("casbinmodel.conf", a)
```


然后我们可以调用 API 对规则进行添加和删除等等操作：


```
Enforcer.LoadPolicy()
Enforcer.AddPolicy("admin", "app", "/app/1", "GET")
Enforcer.AddGroupingPolicy("alice", "admin", "app")
```

> 这里踩了一个小坑，这个 gorm-adapter 的 [README](https://github.com/casbin/gorm-adapter) 里没有写 AddGroupingPolicy 这个 API。还是翻源码才看到的。

规则文件中的 p 对应 AddPolicy API，g 对应 AddGroupingPolicy API。

### Warp it up

> TODO: 把文中的示例代码整理到一个仓库中

一点感受：Casbin 的文档主要是 README 中的内容。Iris 的文档则主要要看 Example 这一节。有点 Example Driven Development 的感觉。虽然这么说，不过整体来说，这些资料还是可以覆盖到我们的使用场景的。


