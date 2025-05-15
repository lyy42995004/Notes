Go 语言标准库中的`net/http`包十分的优秀，提供了非常完善的 HTTP 客户端与服务端的实现，仅通过几行代码就可以搭建一个非常简单的 HTTP 服务器。几乎所有的 go 语言中的 web 框架，都是对已有的 http 包做的封装与修改，因此，十分建议学习其他框架前最好先行掌握 http 包。

# HTTP服务器

我们先初步介绍以下`net/http`包的使用，通过`http.HandleFunc()`和`http.ListenAndServe()`两个函数就可以轻松创建一个简单的Go web服务器，示例代码如下:

```go
package main

import (
    "fmt"
    "net/http"
)

func helloHandler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello, World!")
}

func main() {
    http.HandleFunc("/hello", helloHandler)
    fmt.Println("Server starting on port 8080...")
    http.ListenAndServe(":8080", nil)
}
```

## 路由处理

http包提供了两种路由注册方式：

**处理器函数（HandlerFunc）**：`http.HandleFunc(pattern string, handler func(ResponseWriter, *Request))`

```go
func helloHandler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello, World!")
}

http.HandleFunc("/hello", helloHandler)
```

这种形式最为简单直接，适合处理简单的路由逻辑。`http.HandleFunc`内部会将函数转换为`HandlerFunc`类型，它实现了`Handler`接口。

----

**处理器对象（Handler）**：`http.Handle(pattern string, handler Handler)`

```go
type CounterHandler struct {
    count int
}

func (h *CounterHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    h.count++
    fmt.Fprintf(w, "Visitor count: %d", h.count)
}

handler := &CounterHandler{}
http.Handle("/counter", handler)
```

## ServeMux路由

`http.ServeMux`是Go默认提供的多路复用器（路由器），它实现了`Handler`接口，可以看作是一个高级的路由管理器。它的工作原理是：

1. 注册路由时，将路径模式(pattern)和处理器(handler)存储在内部映射中
2. 收到请求时，按照最长匹配原则查找对应的处理器
3. 如果找不到精确匹配，会尝试查找带斜杠的路径
4. 最终都找不到则返回404错误

```go
mux := http.NewServeMux()
mux.HandleFunc("/products/", productsHandler)
mux.HandleFunc("/articles/", articlesHandler)
mux.HandleFunc("/", indexHandler)
```

这种路由设计虽然简单，但对于RESTful API和传统Web应用已经足够。对于更复杂的需求，可以考虑第三方路由库如gorilla/mux。

# HTTP客户端

## GET请求

```go
resp, err := http.Get("https://example.com")
if err != nil {
    // 处理错误
}
defer resp.Body.Close()

body, err := io.ReadAll(resp.Body)
if err != nil {
    // 处理错误
}
fmt.Println(string(body))
```

## POST请求

```go
data := url.Values{}
data.Set("key", "value")

resp, err := http.PostForm("https://example.com/form", data)
if err != nil {
    // 处理错误
}
defer resp.Body.Close()

// 处理响应...
```

## 自定义请求

```go
req, err := http.NewRequest("GET", "https://example.com", nil)
if err != nil {
    // 处理错误
}

req.Header.Add("Authorization", "Bearer token123")
req.Header.Add("Content-Type", "application/json")

client := &http.Client{
    Timeout: time.Second * 10,
}

resp, err := client.Do(req)
if err != nil {
    // 处理错误
}
defer resp.Body.Close()

// 处理响应...
```

# 中间件模式

中间件是构建模块化HTTP服务的关键模式。Go的中间件本质上是处理器的包装函数，可以在不修改核心业务逻辑的情况下，添加横切关注点功能。

## 基本中间件结构

```go
func loggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        
        // 调用下一个处理器
        next.ServeHTTP(w, r)
        
        // 记录请求日志
        log.Printf(
            "%s %s %s %v",
            r.Method,
            r.URL.Path,
            r.RemoteAddr,
            time.Since(start),
        )
    })
}
```

这种模式的工作流程是：

1. 中间件接收一个处理器作为参数
2. 返回一个新的处理器
3. 新处理器在执行原有逻辑前后添加额外功能

## 中间件链

多个中间件可以串联形成处理链：

```go
func authMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        if !isAuthenticated(r) {
            http.Error(w, "Unauthorized", http.StatusUnauthorized)
            return
        }
        next.ServeHTTP(w, r)
    })
}

func main() {
    mux := http.NewServeMux()
    mux.HandleFunc("/secure", secureHandler)
    
    // 构建中间件链
    handler := loggingMiddleware(authMiddleware(mux))
    
    http.ListenAndServe(":8080", handler)
}
```

中间件的执行顺序是从外到内，即先注册的中间件后执行。上面的例子中，请求会先经过日志记录，再进行认证检查。

## 上下文传递

中间件常用于在请求间传递值，这时应该使用`context.Context`：

```go
func requestIDMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // 生成唯一请求ID
        requestID := uuid.New().String()
        
        // 创建新上下文并存储请求ID
        ctx := context.WithValue(r.Context(), "requestID", requestID)
        
        // 设置响应头
        w.Header().Set("X-Request-ID", requestID)
        
        // 使用新上下文继续处理
        next.ServeHTTP(w, r.WithContext(ctx))
    })
}

// 在处理器中获取请求ID
func handler(w http.ResponseWriter, r *http.Request) {
    requestID := r.Context().Value("requestID").(string)
    fmt.Fprintf(w, "Request ID: %s", requestID)
}
```

这种方式是线程安全的，避免了全局变量的问题，是Go中处理请求间数据传递的推荐方式。

> 参考资料：
>
> [Golang 中文学习文档 标准库 http包](https://golang.halfiisland.com/essential/std/http.html)

