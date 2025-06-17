Gin 是一个用 Go (Golang) 编写的 Web 框架。 它具有类似 martini 的 API，性能要好得多，多亏了 `httprouter`，速度提高了 40 倍。 如果您需要性能和良好的生产力，您一定会喜欢 Gin。Gin 相比于 Iris 和 Beego 而言，更倾向于轻量化的框架，只负责 Web 部分，追求极致的路由性能，功能或许没那么全，胜在轻量易拓展，这也是它的优点。因此，在所有的 Web 框架中，Gin 是最容易上手和学习的。

# 特点

- **快速**：基于 Radix 树的路由，小内存占用。没有反射。可预测的 API 性能。
- **支持中间件**：传入的 HTTP 请求可以由一系列中间件和最终操作来处理。 例如：Logger，Authorization，GZIP，最终操作 DB。
- **Crash 处理**：Gin 可以 catch 一个发生在 HTTP 请求中的 panic 并 recover 它。这样，你的服务器将始终可用。
- **JSON 验证**：Gin 可以解析并验证请求的 JSON，例如检查所需值的存在。
- **路由组**：更好地组织路由。是否需要授权，不同的 API 版本…… 此外，这些组可以无限制地嵌套而不会降低性能。
- **错误管理**：Gin 提供了一种方便的方法来收集 HTTP 请求期间发生的所有错误。最终，中间件可以将它们写入日志文件，数据库并通过网络发送。
- **内置渲染**：Gin 为 JSON，XML 和 HTML 渲染提供了易于使用的 API。
- **可扩展性**：新建一个中间件非常简单。

# 安装

```bash
$ go get -u github.com/gin-gonic/gin
```

示例：

```golang
package main

import "github.com/gin-gonic/gin"

func main() {
    // 创建 Gin 引擎实例
    r := gin.Default()
    
    // 定义路由
    r.GET("/", func(c *gin.Context) {
        c.String(200, "Hello, Gin!")
    })
    
    // 启动服务
    r.Run(":8080") // 默认监听 :8080
}
```

将上面的代码保存并编译执行，然后使用浏览器打开`127.0.0.1:8080/hello`就能看到"Hello, Gin!"字符串。

# RESTful API

REST与技术无关，代表的是一种软件架构风格，REST是Representational State Transfer的简称，中文翻译为“表征状态转移”或“表现层状态转化”。

简单来说，REST的含义就是客户端与Web服务器之间进行交互的时候，使用HTTP协议中的4个请求方法代表不同的动作。

- `GET`用来获取资源
- `POST`用来新建资源
- `PUT`用来更新资源
- `DELETE`用来删除资源。

只要API程序遵循了REST风格，那就可以称其为RESTful API。目前在前后端分离的架构中，前后端基本都是通过RESTful API来进行交互。

例如，我们现在要编写一个管理书籍的系统，我们可以查询对一本书进行查询、创建、更新和删除等操作，我们在编写程序的时候就要设计客户端浏览器与我们Web服务端交互的方式和路径。按照经验我们通常会设计成如下模式：

| 请求方法 |     URL      |     含义     |
| :------: | :----------: | :----------: |
|   GET    |    /book     | 查询书籍信息 |
|   POST   | /create_book | 创建书籍记录 |
|   POST   | /update_book | 更新书籍信息 |
|   POST   | /delete_book | 删除书籍信息 |

同样的需求我们按照RESTful API设计如下：

| 请求方法 |  URL  |     含义     |
| :------: | :---: | :----------: |
|   GET    | /book | 查询书籍信息 |
|   POST   | /book | 创建书籍记录 |
|   PUT    | /book | 更新书籍信息 |
|  DELETE  | /book | 删除书籍信息 |

Gin框架支持开发RESTful API的开发。

```go
func main() {
	r := gin.Default()
	r.GET("/book", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "GET",
		})
	})

	r.POST("/book", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "POST",
		})
	})

	r.PUT("/book", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "PUT",
		})
	})

	r.DELETE("/book", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "DELETE",
		})
	})
}
```

开发RESTful API的时候我们通常使用[Postman](https://www.getpostman.com/)来作为客户端的测试工具。

# 获取参数

## 获取querystring参数

`querystring`指的是URL中`?`后面携带的参数，例如：`/user/search?username=彭于晏&address=深圳`。 获取请求的querystring参数的方法如下：

```go
r := gin.Default()

// 处理 /search?name=彭于晏&location=深圳
r.GET("/search", func(c *gin.Context) {
    // 获取必填参数，若不存在则返回400错误
    name := c.Query("name")
    
    // 获取可选参数，提供默认值
    location := c.DefaultQuery("location", "unknown")
    
    c.JSON(200, gin.H{
        "name":     name,
        "location": location,
    })
})
```

## 获取form参数

当前端请求的数据通过form表单提交时，例如向`/register`发送一个POST请求，获取请求数据的方式如下：

```go
r.POST("/register", func(c *gin.Context) {
    // 获取必填表单字段
    username := c.PostForm("username")
    
    // 获取可选表单字段
    email := c.DefaultPostForm("email", "default@example.com")
    
    c.JSON(200, gin.H{
        "username": username,
        "email":    email,
    })
})
```

## 获取JSON参数

当前端请求的数据通过JSON提交时，例如向`/users`发送一个JSON格式的POST请求，则获取请求参数的方式如下：

```go
type User struct {
    Username string `json:"username" binding:"required"`
    Email    string `json:"email"`
}

r.POST("/users", func(c *gin.Context) {
    var user User
    
    // 自动解析JSON请求体
    if err := c.ShouldBindJSON(&user); err != nil {
        c.JSON(400, gin.H{"error": err.Error()})
        return
    }
    
    c.JSON(201, gin.H{
        "status": "created",
        "user":   user,
    })
})
```

## 获取path参数

请求的参数通过URL路径传递，例如：`/user/123`。 获取请求URL路径中的参数的方式如下。

```go
// 处理 /users/123
r.GET("/users/:id", func(c *gin.Context) {
    id := c.Param("id")
    
    // 处理 /files/*path 通配符路径
    path := c.Param("path")
    
    c.JSON(200, gin.H{
        "id":   id,
        "path": path,
    })
})
```

## 参数绑定

为了能够更方便的获取请求相关参数，提高开发效率，我们可以基于请求的`Content-Type`识别请求数据类型并利用反射机制自动提取请求中`QueryString`、`form表单`、`JSON`、`XML`等参数到结构体中。 下面的示例代码演示了`.ShouldBind()`强大的功能，它能够基于请求自动提取`JSON`、`form表单`和`QueryString`类型的数据，并把值绑定到指定的结构体对象。

```go
// Binding from JSON
type Login struct {
	User     string `form:"user" json:"user" binding:"required"`
	Password string `form:"password" json:"password" binding:"required"`
}

func main() {
	router := gin.Default()

	// 绑定JSON的示例 ({"user": "tian", "password": "123456"})
	router.POST("/loginJSON", func(c *gin.Context) {
		var login Login

		if err := c.ShouldBind(&login); err == nil {
			fmt.Printf("login info:%#v\n", login)
			c.JSON(http.StatusOK, gin.H{
				"user":     login.User,
				"password": login.Password,
			})
		} else {
			c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
		}
	})

	// 绑定form表单示例 (user=tian&password=123456)
	router.POST("/loginForm", func(c *gin.Context) {
		var login Login
		// ShouldBind()会根据请求的Content-Type自行选择绑定器
		if err := c.ShouldBind(&login); err == nil {
			c.JSON(http.StatusOK, gin.H{
				"user":     login.User,
				"password": login.Password,
			})
		} else {
			c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
		}
	})

	// 绑定QueryString示例 (/loginQuery?user=q1mi&password=123456)
	router.GET("/loginForm", func(c *gin.Context) {
		var login Login
		// ShouldBind()会根据请求的Content-Type自行选择绑定器
		if err := c.ShouldBind(&login); err == nil {
			c.JSON(http.StatusOK, gin.H{
				"user":     login.User,
				"password": login.Password,
			})
		} else {
			c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
		}
	})

	// Listen and serve on 0.0.0.0:8080
	router.Run(":8080")
}
```

`ShouldBind`会按照下面的顺序解析请求中的数据完成绑定：

1. 如果是 `GET` 请求，只使用 `Form` 绑定引擎（`query`）。
2. 如果是 `POST` 请求，首先检查 `content-type` 是否为 `JSON` 或 `XML`，然后再使用 `Form`（`form-data`）。

# 文件上传

## 单个文件上传

```go
// 配置上传限制（默认32MB）
router.MaxMultipartMemory = 8 << 20 // 8MB

router.POST("/upload", func(c *gin.Context) {
    file, err := c.FormFile("file")
    if err != nil {
        c.JSON(500, gin.H{"error": err.Error()})
        return
    }

    // 安全保存文件
    dst := filepath.Join("/uploads", file.Filename)
    if err := c.SaveUploadedFile(file, dst); err != nil {
        c.JSON(500, gin.H{"error": "保存失败"})
        return
    }

    c.JSON(200, gin.H{
        "status":   "上传成功",
        "filename": file.Filename,
        "size":     file.Size,
    })
})
```

## 多个文件上传

```go
router.POST("/multi-upload", func(c *gin.Context) {
    form, _ := c.MultipartForm()
    files := form.File["files"]

    var results []gin.H
    for _, file := range files {
        dst := filepath.Join("/uploads", file.Filename)
        if err := c.SaveUploadedFile(file, dst); err == nil {
            results = append(results, gin.H{
                "filename": file.Filename,
                "status":   "success",
            })
        }
    }

    c.JSON(200, gin.H{
        "total":   len(files),
        "success": len(results),
        "results": results,
    })
})
```

# 重定向

## HTTP重定向

HTTP 重定向很容易。 内部、外部重定向均支持。

```go
// 永久重定向
router.GET("/old", func(c *gin.Context) {
    c.Redirect(301, "/new")
})

// 临时重定向
router.GET("/temp", func(c *gin.Context) {
    c.Redirect(302, "https://example.com")
})
```

## 路由重定向

路由重定向，使用`HandleContext`：

```go
router.GET("/route1", func(c *gin.Context) {
    c.Request.URL.Path = "/route2"
    router.HandleContext(c)
})

router.GET("/route2", func(c *gin.Context) {
    c.String(200, "内部路由转发成功")
})
```

# Gin路由

### 普通路由

```go
r.GET("/index", func(c *gin.Context) {...})
r.GET("/login", func(c *gin.Context) {...})
r.POST("/login", func(c *gin.Context) {...})
```

此外，还有一个可以匹配所有请求方法的`Any`方法如下：

```go
r.Any("/test", func(c *gin.Context) {...})
```

为没有配置处理函数的路由添加处理程序，默认情况下它返回404代码，下面的代码为没有匹配到路由的请求都返回`views/404.html`页面。

```go
r.NoRoute(func(c *gin.Context) {
		c.HTML(http.StatusNotFound, "views/404.html", nil)
	})
```

### 路由组

我们可以将拥有共同URL前缀的路由划分为一个路由组。习惯性一对`{}`包裹同组的路由，这只是为了看着清晰，你用不用`{}`包裹功能上没什么区别。

```go
func main() {
	r := gin.Default()
	userGroup := r.Group("/user")
	{
		userGroup.GET("/index", func(c *gin.Context) {...})
		userGroup.GET("/login", func(c *gin.Context) {...})
		userGroup.POST("/login", func(c *gin.Context) {...})

	}
	shopGroup := r.Group("/shop")
	{
		shopGroup.GET("/index", func(c *gin.Context) {...})
		shopGroup.GET("/cart", func(c *gin.Context) {...})
		shopGroup.POST("/checkout", func(c *gin.Context) {...})
	}
	r.Run()
}
```

路由组也是支持嵌套的，例如：

```go
shopGroup := r.Group("/shop")
	{
		shopGroup.GET("/index", func(c *gin.Context) {...})
		shopGroup.GET("/cart", func(c *gin.Context) {...})
		shopGroup.POST("/checkout", func(c *gin.Context) {...})
		// 嵌套路由组
		xx := shopGroup.Group("xx")
		xx.GET("/oo", func(c *gin.Context) {...})
	}
```

通常我们将路由分组用在划分业务逻辑或划分API版本时。

# Gin中间件

Gin框架允许开发者在处理请求的过程中，加入用户自己的钩子（Hook）函数。这个钩子函数就叫中间件，中间件适合处理一些公共的业务逻辑，比如登录认证、权限校验、数据分页、记录日志、耗时统计等。

## 定义中间件

Gin中的中间件必须是一个`gin.HandlerFunc`类型。

**记录接口耗时的中间件**

例如我们像下面的代码一样定义一个统计请求耗时的中间件。

```go
// 耗时统计中间件
func LatencyMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        start := time.Now()
        
        // 前置处理
        c.Set("traceId", uuid.NewString())
        
        // 执行后续处理器
        c.Next()
        
        // 后置处理
        latency := time.Since(start)
        log.Printf("[%s] %s cost %v", 
            c.GetString("traceId"), 
            c.Request.URL.Path, 
            latency)
    }
}
```

**记录响应体的中间件**

我们有时候可能会想要记录下某些情况下返回给客户端的响应数据，这个时候就可以编写一个中间件来搞定。

```go
type responseRecorder struct {
    gin.ResponseWriter
    body *bytes.Buffer
}

func (r *responseRecorder) Write(b []byte) (int, error) {
    r.body.Write(b)
    return r.ResponseWriter.Write(b)
}

func ResponseLogger() gin.HandlerFunc {
    return func(c *gin.Context) {
        recorder := &responseRecorder{
            ResponseWriter: c.Writer,
            body:           bytes.NewBuffer(nil),
        }
        c.Writer = recorder
        
        c.Next()
        
        if c.Writer.Status() >= 400 {
            log.Printf("Error Response: %s", recorder.body.String())
        }
    }
}
```

## 注册中间件

在gin框架中，我们可以为每个路由添加任意数量的中间件。

**为全局路由注册**

```go
r := gin.New()
r.Use(
    gin.Recovery(),    // panic恢复
    LatencyMiddleware(), // 耗时统计
    ResponseLogger(),   // 响应日志
)
```

**为某个路由单独注册**

```go
// 给/metrics路由单独注册中间件（可注册多个）
r.GET("/metrics", 
    BasicAuthMiddleware(),  // 基础认证
    PrometheusHandler(),    // 监控端点
)
```

**为路由组注册中间件**

为路由组注册中间件有以下两种写法。

写法1：

```go
shopGroup := r.Group("/shop", StatCost())
{
    shopGroup.GET("/index", func(c *gin.Context) {...})
    ...
}
```

写法2：

```go
shopGroup := r.Group("/shop")
shopGroup.Use(StatCost())
{
    shopGroup.GET("/index", func(c *gin.Context) {...})
    ...
}
```

## 中间件注意事项

**gin默认中间件**

`gin.Default()`默认使用了`Logger`和`Recovery`中间件，其中：

- `Logger`中间件将日志写入`gin.DefaultWriter`，即使配置了`GIN_MODE=release`。
- `Recovery`中间件会recover任何`panic`。如果有panic的话，会写入500响应码。

如果不想使用上面两个默认的中间件，可以使用`gin.New()`新建一个没有任何默认中间件的路由。

**gin中间件中使用goroutine**

当在中间件或`handler`中启动新的`goroutine`时，**不能使用**原始的上下文（c *gin.Context），必须使用其只读副本（`c.Copy()`）。

# 运行多个服务

我们可以在多个端口启动服务，例如：

```go
func main() {
    var g errgroup.Group
    
    // API服务
    g.Go(func() error {
        return startServer(":8080", apiRouter())
    })
    
    // 监控服务
    g.Go(func() error {
        return startServer(":8081", metricsRouter())
    })
    
    if err := g.Wait(); err != nil {
        log.Fatal(err)
    }
}

func startServer(addr string, handler http.Handler) error {
    srv := &http.Server{
        Addr:         addr,
        Handler:      handler,
        ReadTimeout:  5 * time.Second,
        WriteTimeout: 10 * time.Second,
    }
    return srv.ListenAndServe()
}
```

> 参考资料：
>
> [Gin Web Framework](https://gin-gonic.com/zh-cn/docs/)
>
> [Golang 中文学习文档 - Web开发 - Gin](https://golang.halfiisland.com/community/pkgs/web/gin.html)
>
> [李文周的博客 - Gin框架介绍及使用](https://www.liwenzhou.com/posts/Go/gin/)