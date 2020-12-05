---
title: Golang Gin 框架 Middleware
date: 2020-06-30 09:51:35
tags:
    - Golang
    - Gin
    - 记录
categories:
    - 技术
---

最近在学习 Golang + Gin + Vue.js 的项目, 在处理 CORS 跨域问题上发现了个不算坑的坑.

```golang
func New() *gin.Engine {
    router := gin.Default()
    registerPostApis(router)
    registerUserApis(router)
    router.Use(cors.Default())
    return router
}
```

这个是我一开始写的, 然后发现不管是用 Gin 官方的 CORS 中间件, 还是自己写一个 CORS 中间件, 在前端 Vue.js 用 axios 调用的时候, 会出现

> cross-Origin Request Blocked: The Same Origin Policy disallows reading the remote 
> resource at http://127.0.0.1:8081/user/login. (Reason: CORS header ‘Access-Control-Allow-Origin’ missing).

但这不应该, 我都有设置好 `Access-Control-Allow-Origin`, `Access-Control-Allow-Header`等 Headers

通过抓包也能看到 response 里面没有这些信息

然后, 我把中间件放到了注册路由的前面, 即

```golang
func New() *gin.Engine {
    router := gin.Default()
    router.Use(cors.Default())
    registerPostApis(router)
    registerUserApis(router)
    return router
}
```

一切都顺畅了, 不会出现跨域的问题了. 

那么, 是为什么呢?

```golang
func Default() *Engine {
    debugPrintWARNINGDefault()
    engine := New()
    engine.Use(Logger(), Recovery())
    return engine
}

func (engine *Engine) Use(middleware ...HandlerFunc) IRoutes {
    engine.RouterGroup.Use(middleware...)
    engine.rebuild404Handlers()
    engine.rebuild405Handlers()
    return engine
}

func (group *RouterGroup) Use(middleware ...HandlerFunc) IRoutes {
    group.Handlers = append(group.Handlers, middleware...)
    return group.returnObj()
}
```
可以看到, 调用 `engine.Use()`的时候, 是调用了`engine.RouterGroup.Use()`, 然后将中间件加入到了 `group.Handlers` 里.

而在注册路由的时候, 用到了 `engine.Group()`

> 即使不用 `Group()`, 直接用 `engine.GET("/URL")` 也是一样的

```golang
func (group *RouterGroup) Group(relativePath string, handlers ...HandlerFunc) *RouterGroup {
    return &RouterGroup{
        Handlers: group.combineHandlers(handlers),
        basePath: group.calculateAbsolutePath(relativePath),
        engine:   group.engine,
    }
}

func (group *RouterGroup) combineHandlers(handlers HandlersChain) HandlersChain {
    finalSize := len(group.Handlers) + len(handlers)
    if finalSize >= int(abortIndex) {
        panic("too many handlers")
    }
    mergedHandlers := make(HandlersChain, finalSize)
    copy(mergedHandlers, group.Handlers)
    copy(mergedHandlers[len(group.Handlers):], handlers)
    return mergedHandlers
}
```

对于这个路由组的中间件, 是把 engine 的中间件复制一份给了 mergedHandlers, 
也就是把已有的中间件和新增的中间件和在一起使用了

也就是说, 如果在注册路由之后, 新增了一个中间件, 那这个中间件不会对已经注册的路由产生效果.