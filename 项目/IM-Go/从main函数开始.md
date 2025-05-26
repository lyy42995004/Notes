# 重构logger

上次写的logger.go过于繁琐，有很多没用到的功能；重构后只提供了简洁的日志接口，支持日志轮转、多级别日志记录等功能，并采用单例模式确保全局只有一个日志实例

### 全局变量

```go
var (
    once    sync.Once        // 用于实现单例模式的同步控制
    Logger  *zap.Logger      // 全局日志实例
    String  = zap.String     // 导出常用的 zap.Field 构造方法
    Any     = zap.Any
    Err     = zap.Error
    Int     = zap.Int
    Float32 = zap.Float32
)
```

### 默认配置常量

```go
const (
    defaultLogPath    = "./logs"      // 默认日志存储目录
    defaultLogLevel   = "debug"        // 默认日志级别
    defaultMaxSize    = 100           // 单个日志文件最大大小(MB)
    defaultMaxBackups = 30            // 保留的旧日志文件数量
    defaultMaxAge     = 7             // 日志保留天数
    defaultCompress   = true          // 是否压缩旧日志
)
```

### 初始化日志记录器

```go
// Init 初始化日志记录器(单例模式)
func Init() *zap.Logger {
	once.Do(func() {
		// 确保日志目录存在
		if err := os.MkdirAll(defaultLogPath, 0755); err != nil {
			panic(err)
		}

		// 设置日志级别
		level := getLogLevel(defaultLogLevel)

		// 日志文件路径
		logFile := getDatedLogFilename(defaultLogPath)

		// 设置日志轮转
		writer := zapcore.AddSync(&lumberjack.Logger{
			Filename:   logFile,
			MaxSize:    defaultMaxSize,    // MB
			MaxBackups: defaultMaxBackups, // 保留的旧日志文件数量
			MaxAge:     defaultMaxAge,     // 保留天数
			Compress:   defaultCompress,   // 是否压缩
		})

		// 编码器配置
		encoderConfig := zap.NewProductionEncoderConfig()
		encoderConfig.EncodeTime = zapcore.ISO8601TimeEncoder
		encoderConfig.EncodeLevel = zapcore.CapitalLevelEncoder // 不带颜色

		// 核心配置
		core := zapcore.NewCore(
			// zapcore.NewJSONEncoder(encoderConfig), // json格式
			zapcore.NewConsoleEncoder(encoderConfig), // 使用 Console 编码器
			writer,
			level,
		)

		// 创建Logger
		Logger = zap.New(core)
	})

	return Logger
}
```

### 获取日志实例

```go
func GetLogger() *zap.Logger {
    if Logger == nil {
        Init() // 自动初始化
    }
    return Logger
}
```

### 直接可用的日志方法

```go
func Debug(msg string, fields ...zap.Field) {
    GetLogger().Debug(msg, fields...)
}

func Info(msg string, fields ...zap.Field) {
    GetLogger().Info(msg, fields...)
}

func Warn(msg string, fields ...zap.Field) {
    GetLogger().Warn(msg, fields...)
}

func Error(msg string, fields ...zap.Field) {
    GetLogger().Error(msg, fields...)
}

func Fatal(msg string, fields ...zap.Field) {
    GetLogger().Fatal(msg, fields...)
}

func Sync() error {
    return GetLogger().Sync()
}
```

> 代码地址：[logger.go](https://github.com/lyy42995004/IM-Go/tree/master/pkg/log)

# 从main函数开始

```go
func main() {
	defer log.Sync() // 记录日志

	log.Info("start chat server...")

	newRouter := router.NewRouter()

	go chat.MyServer.Start()

	s := &http.Server{
        Addr:           ":8080",
		Handler:        newRouter,
		ReadTimeout:    10 * time.Second,
		WriteTimeout:   10 * time.Second,
		MaxHeaderBytes: 1 << 20,
	}
	err := s.ListenAndServe()
	if err != nil {
		log.Error("server start error", log.Err(err))
	}
}
```

1. **初始化路由**
   - 创建 HTTP 请求路由器。
   - 会在这里定义所有的 API 端点（endpoints）和对应的处理函数。
   - 返回一个实现了 `http.Handler` 接口的路由器对象。
2. **启动后台服务**
   - 使用 `go` 关键字启动一个 goroutine，异步运行 `chat.MyServer.Start()` 方法。
   - 这通常用于启动需要长期运行的 WebSocket 服务。
3. **配置 HTTP 服务器**
4. **启动 HTTP 服务器**：开始监听指定端口（8080）的 HTTP 请求。

# 自定义Gin路由

![](https://s2.loli.net/2025/05/20/cFGtXl9aEBPrpoD.png)

### NewRouter 函数解析

```go
func NewRouter() *gin.Engine {
	gin.SetMode(gin.ReleaseMode)

	server := gin.Default()
	server.Use(Cors())
	server.Use(Recovery)

	socket := RunSocket
	
	group := server.Group("")
	{
		// 用户管理功能
		group.GET("/user", api.GetUserList)
		group.GET("/user/:uuid", api.GetUserDetails)
		group.GET("/user/name", api.GetUserOrGroupByName)
		group.POST("/user/register", api.Register)
		group.POST("/user/login", api.Login)
		group.PUT("/user", api.ModifyUserInfo)

		group.POST("/friend", api.AddFriend)

		group.GET("/message", api.GetMessage)

		group.GET("/file/:fileName", api.GetFile)
		group.POST("/file", api.SaveFile)

		group.GET("/group/:uuid", api.GetGroup)
		group.POST("/group/:uuid", api.SaveGroup)
		group.POST("/group/join/:userUuid/:groupUuid", api.JoinGroup)
		group.GET("/group/user/:uuid", api.GetGroupUsers)

		group.GET("/socket.io", socket)
	}
	return server
}
```

1. **Gin 模式设置**：`gin.SetMode(gin.ReleaseMode)` 将 Gin 设置为发布模式，减少调试信息输出。
2. **中间件使用**：`Cors()` 中间件处理跨域请求，`Recovery` 中间件捕获并处理 panic。
3. **路由分组**：所有路由都定义在根分组 (`""`) 下，清晰的 RESTful 风格路由设计。
4. **路由类型**：用户管理：GET/POST/PUT 操作；文件管理：文件上传/下载；WebSocket：实时通信端点。

### 跨域处理中间件 (Cors)

```go
func Cors() gin.HandlerFunc {
    return func(c *gin.Context) {
        method := c.Request.Method
        origin := c.Request.Header.Get("Origin")
        if origin != "" {
            // 设置跨域响应头
            c.Header("Access-Control-Allow-Origin", "*")
            c.Header("Access-Control-Allow-Methods", "POST, GET, OPTIONS, PUT, DELETE, UPDATE")
            c.Header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept, Authorization")
            c.Header("Access-Control-Expose-Headers", "Content-Length, Access-Control-Allow-Origin, Access-Control-Allow-Headers, Cache-Control, Content-Language, Content-Type")
            c.Header("Access-Control-Allow-Credentials", "true")
        }
        
        // 处理 OPTIONS 预检请求
        if method == "OPTIONS" {
            c.JSON(http.StatusOK, "ok!")
            return
        }

        // 异常捕获
        defer func() {
            if err := recover(); err != nil {
                log.Error("HttpError", log.Any("HttpError", err))
            }
        }()

        c.Next() // 处理请求
    }
}
```

1. **跨域头设置**：
   - 允许所有来源 (`*`)，生产环境应替换为具体域名
   - 允许的 HTTP 方法
   - 允许的请求头
   - 允许客户端访问的响应头
   - 允许携带凭证
2. **OPTIONS 请求处理**：直接返回 200 状态码，满足浏览器的预检请求。
3. **异常捕获**：使用 defer+recover 捕获处理过程中的 panic，记录错误日志。

### 恢复中间件 (Recovery)

```go
func Recovery(c *gin.Context) {
    defer func() {
        if r := recover(); r != nil {
            log.Error("gin catch error", log.Any("error", r))
            c.JSON(http.StatusOK, response.FailMsg("系统内部错误"))
        }
    }()
    c.Next()
}
```

**panic 恢复**：捕获路由处理函数中可能发生的 panic，记录错误日志

### WebSocket 实现 (RunSocket)

```go
var upGrader = websocket.Upgrader{
    CheckOrigin: func(r *http.Request) bool {
        return true // 允许所有跨域 WebSocket 连接
    },
}

func RunSocket(c *gin.Context) {
    user := c.Query("user") // 获取用户标识
    if user == "" {
        return // 无用户标识则拒绝连接
    }
    log.Info("newUser", log.String("newUser", user))

    // 升级 HTTP 连接为 WebSocket 连接
    ws, err := upGrader.Upgrade(c.Writer, c.Request, nil)
    if err != nil {
        return
    }

    // 创建客户端对象
    client := &server.Client{
        Name: user,
        Conn: ws,
        Send: make(chan []byte),
    }

    // 注册客户端到服务器
    server.MyServer.Register <- client
    
    // 启动读写协程
    go client.Read()
    go client.Write()
}
```

1. **WebSocket 升级**：
   - 使用 `gorilla/websocket` 包的 `Upgrader` 将 HTTP 连接升级为 WebSocket 连接
   - `CheckOrigin` 返回 true 允许所有跨域连接（生产环境应做限制）
2. **客户端管理**：
   - 通过 `user` 查询参数识别用户，创建客户端对象，包含 WebSocket 连接和消息通道
   - 通过注册通道将客户端注册到中央服务器。
3. **并发处理**：为每个客户端启动独立的读 (`client.Read()`) 和写 (`client.Write()`) 协程，实现全双工通信。
