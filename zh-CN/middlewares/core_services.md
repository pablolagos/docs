---
root: false
name: 核心服务
sort: 0
---

# 核心服务

Macaron 会注入一些默认服务来驱动您的应用，这些服务被称之为 **核心服务**。也就是说，您可以直接使用它们作为处理器参数而不需要任何附加工作。

## 请求上下文（Context）

该服务通过类型 [`*macaron.Context`](https://gowalker.org/github.com/Unknwon/macaron#Context) 来体现。这是 Macaron 最为核心的服务，您的任何操作都是基于它之上。该服务包含了您所需要的请求对象、响应流、模板引擎接口、数据存储和注入与获取其它服务。

使用方法：

```go
package main

import "github.com/Unknwon/macaron"

func Home(ctx *macaron.Context) {
	// ...
}
```

### Next()

[Context.Next()](https://gowalker.org/github.com/Unknwon/macaron#Context_Next) 是一个可选的函数用于中间件处理器暂时放弃执行直到其他的处理器都执行完毕. 这样就可以很好的处理在 HTTP 请求完成后需要做的操作：

```go
// log before and after a request
m.Use(func(ctx *macaron.Context, log *log.Logger){
	log.Println("before a request")

	ctx.Next()

	log.Println("after a request")
})
```

### Cookie

最基本的 Cookie 用法：

- [ctx.SetCookie](https://gowalker.org/github.com/Unknwon/macaron#Context_SetCookie)
- [ctx.GetCookie](https://gowalker.org/github.com/Unknwon/macaron#Context_GetCookie)

如果需要更加安全的 Cookie 机制，可以先使用 [macaron.SetDefaultCookieSecret](https://gowalker.org/github.com/Unknwon/macaron#Macaron_SetDefaultCookieSecret) 设定密钥，然后使用：

- [ctx.SetSecureCookie](https://gowalker.org/github.com/Unknwon/macaron#Context_SetSecureCookie)
- [ctx.GetSecureCookie](https://gowalker.org/github.com/Unknwon/macaron#Context_GetSecureCookie)

这两个方法将会自动使用您设置的默认密钥进行加密/解密 Cookie 值。

对于那些对安全性要求特别高的应用，可以为每次设置 Cookie 使用不同的密钥加密/解密：

- [ctx.SetSuperSecureCookie](https://gowalker.org/github.com/Unknwon/macaron#Context_SetSuperSecureCookie)
- [ctx.GetSuperSecureCookie](https://gowalker.org/github.com/Unknwon/macaron#Context_GetSuperSecureCookie)

## 路由日志

该服务可以通过函数  [`macaron.Logger`](https://gowalker.org/github.com/Unknwon/macaron#Logger) 来注入。该服务主要负责应用的路由日志。

使用方法：

```go
package main

import "github.com/Unknwon/macaron"

func main() {
	m := macaron.New()
	m.Use(macaron.Logger())
	// ...
}
``` 

**备注** 当您使用 [`macaron.Classic`](https://gowalker.org/github.com/Unknwon/macaron#Classic) 时，该服务会被自动注入。

从 [Gogs Web](https://github.com/gogits/gogs) 项目中提取的样例输出：

```
[Macaron] Started GET /docs/middlewares/core.html for [::1]
[Macaron] Completed /docs/middlewares/core.html 200 OK in 2.114956ms
```

## 容错恢复

该服务可以通过函数 [`macaron.Recovery`](https://gowalker.org/github.com/Unknwon/macaron#Recovery) 来注入。该服务主要负责在应用发生恐慌（panic）时进行恢复。

使用方法：

```go
package main

import "github.com/Unknwon/macaron"

func main() {
	m := macaron.New()
	m.Use(macaron.Recovery())
	// ...
}
``` 

**备注** 当您使用 [`macaron.Classic`](https://gowalker.org/github.com/Unknwon/macaron#Classic) 时，该服务会被自动注入。

## 静态文件

该服务可以通过函数 [`macaron.Static`](https://gowalker.org/github.com/Unknwon/macaron#Static) 来注入。该服务主要负责应用静态资源的服务，当您的应用拥有多个静态目录时，可以对其进行多次注入。

使用方法：

```go
package main

import "github.com/Unknwon/macaron"

func main() {
	m := macaron.New()
	m.Use(macaron.Static("public"))
	m.Use(macaron.Static("assets"))
	// ...
}
``` 

**备注** 当您使用 [`macaron.Classic`](https://gowalker.org/github.com/Unknwon/macaron#Classic) 时，该服务会以 `public` 为静态目录被自动注入。

默认情况下，当您请求一个目录时，该服务不会列出目录下的文件，而是去寻找 `index.html` 文件。

从 [Gogs Web](https://github.com/gogits/gogs) 项目中提取的样例输出：

```
[Macaron] Started GET /css/prettify.css for [::1]
[Macaron] [Static] Serving /css/prettify.css
[Macaron] Completed /css/prettify.css 304 Not Modified in 97.584us
[Macaron] Started GET /imgs/macaron.png for [::1]
[Macaron] [Static] Serving /imgs/macaron.png
[Macaron] Completed /imgs/macaron.png 304 Not Modified in 123.211us
[Macaron] Started GET /js/gogsweb.min.js for [::1]
[Macaron] [Static] Serving /js/gogsweb.min.js
[Macaron] Completed /js/gogsweb.min.js 304 Not Modified in 47.653us
[Macaron] Started GET /css/main.css for [::1]
[Macaron] [Static] Serving /css/main.css
[Macaron] Completed /css/main.css 304 Not Modified in 42.58us
```

### 使用示例

假设您的应用拥有以下目录结构：

```
public/
	|__ html
			|__ index.html
	|__ css/
			|__ main.css
```

响应结果：

请求 URL|匹配文件
-----------|----------
`/html/main.html`|匹配失败
`/html/`|index.html
`/css/main.css`|main.css

### 自定义选项

该服务允许接受第二个参数来进行自定义选项操作（[`macaron.StaticOptions`](https://gowalker.org/github.com/Unknwon/macaron#StaticOptions)）：

```go
package main

import "github.com/Unknwon/macaron"

func main() {
	m := macaron.New()
	m.Use(macaron.Static("public", 
		macaron.StaticOptions{
			// 请求静态资源时的 URL 前缀，默认没有前缀
			Prefix: "public", 	
			// 禁止记录静态资源路由日志，默认为不禁止记录
			SkipLogging: true, 	
			// 当请求目录时的默认索引文件，默认为 "index.html"
			IndexFile: "index.html",	
			// 用于返回自定义过期响应头，不能为不设置
			// https://developers.google.com/speed/docs/insights/LeverageBrowserCaching
			Expires: func() string { return "max-age=0" },
		}))
	// ...
}
```

## 其它服务

### 全局日志

该服务通过类型 [`*log.Logger`](http://gowalker.org/log#Logger) 来体现。该服务为可选，只是为没有日志器的应用提供一定的便利。

使用方法：

```go
package main

import (
	"log"

	"github.com/Unknwon/macaron"
)

func main() {
	m := macaron.Classic()
	m.Get("/", myHandler)
	m.Run()
}

func myHandler(ctx *macaron.Context, logger *log.Logger) string {
	logger.Println("the request path is: " + ctx.Req.RequestURI)
	return "the request path is: " + ctx.Req.RequestURI
}
```

**备注** 所有 [Macaron 实例](../intro/core_concepts#macaron-%E5%AE%9E%E4%BE%8B) 都会自动注册该服务。

### 响应流

该服务通过类型 [`http.ResponseWriter`](http://gowalker.org/net/http/#ResponseWriter) 来体现。该服务为可选，一般情况下可直接使用 `*macaron.Context.Resp`。

使用方法：

```go
package main

import (
	"github.com/Unknwon/macaron"
)

func main() {
	m := macaron.Classic()
	m.Get("/", myHandler)
	m.Run()
}

func myHandler(ctx *macaron.Context) {
	ctx.Resp.Write([]byte("the request path is: " + ctx.Req.RequestURI))
}
```

**备注** 所有 [Macaron 实例](../intro/core_concepts#macaron-%E5%AE%9E%E4%BE%8B) 都会自动注册该服务。

### 请求对象

该服务通过类型 [`*http.Request`](http://gowalker.org/net/http/#Request) 来体现。该服务为可选，一般情况下可直接使用 `*macaron.Context.Req`。

除此之外，该服务还提供了 3 个便利的方法来获取请求体：

- [`*macaron.Context.Req.Body().String()`](https://gowalker.org/github.com/Unknwon/macaron#RequestBody_String)：获取 `string` 类型的请求体
- [`*macaron.Context.Req.Body().Bytes()`](https://gowalker.org/github.com/Unknwon/macaron#RequestBody_Bytes)：获取 `[]byte` 类型的请求体
- [`*macaron.Context.Req.Body().ReadCloser()`](https://gowalker.org/github.com/Unknwon/macaron#RequestBody_ReadCloser)：获取 `io.ReadCloser` 类型的请求体

使用方法：

```go
package main

import (
	"github.com/Unknwon/macaron"
)

func main() {
	m := macaron.Classic()
	m.Get("/body1", func(ctx *Context) {
		reader, err := ctx.Req.Body().ReadCloser()
		// ...
	})
	m.Get("/body2", func(ctx *Context) {
		data, err := ctx.Req.Body().Bytes()
		// ...
	})
	m.Get("/body3", func(ctx *Context) {
		data, err := ctx.Req.Body().String()
		// ...
	})
	m.Run()
}
```

需要注意的是，请求体在每个请求中只能被读取一次。

**备注** 所有 [Macaron 实例](../intro/core_concepts#macaron-%E5%AE%9E%E4%BE%8B) 都会自动注册该服务。