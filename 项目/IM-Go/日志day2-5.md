# Go语言即时通讯系统开发日志day2

> **计划**：学习go中MySQL，Redis的使用，使用MySQL和Redis完成一个单聊demo。
>
> **总结**：现在每天下午用来开发这个项目，如果有课的话可能学习时间只有3-4个小时，再加上今天的学习效率不高；今天只做了一些开发规划，并了解了go语言如何使用**MySQL**，**Redis**，下了两篇博客，[Go 语言 sqlx 库使用：对 MySQL 增删改查](https://blog.csdn.net/lyy42995004/article/details/147963776)、[Go 语言 Redis 使用 ：安装、配置与操作](https://blog.csdn.net/lyy42995004/article/details/147963838)，并把第一天的欠下的**net/http库**学习了解了一下，[Go 语言 net/http 包使用：HTTP 服务器、客户端与中间件](https://blog.csdn.net/lyy42995004/article/details/147963755)。打算先参照GitHub上的[go-chat](https://github.com/kone-net/go-chat)这个IM进行参考学习，后续有能力了再对项目的模块进行优化和重构。

## 项目实现顺序分析

大概分析了一下，后续开发的顺序，实际随情况而变，主要是为了防止每天开始学习时，不知从而下手的处境。

- 基础架构搭建
  - 搭建项目结构
  - 日志系统封装
  - 配置系统
- WebSocket基础服务
  - 实现WebSocket
  - 连接管理
- 单聊核心功能
  - 数据库表设计
  - 数据模型
  - 消息流转
- 前端实现（AI实现）
- 其他功能
  - 消息状态
  - 通知系统
  - 用户认证

# Go语言即时通讯系统开发日志day3

> **计划**：了解Gin，Gorm框架的使用，构建项目结构，了解Go项目的开发流程。
>
> **总结**：通过[【最新Go Web开发教程】基于gin框架和gorm的web开发实战 (七米出品)](https://www.bilibili.com/video/BV1gJ411p7xC)学习了Go语言的Gin框架，Gorm框架。

## Gin

[Go语言 Gin框架 使用指南](https://blog.csdn.net/lyy42995004/article/details/148031673)

**Gin** 是一个用 Go 编写的高性能 Web 框架，以轻量级和快速路由著称，适合构建 RESTful API 和微服务。它提供了中间件支持、JSON 解析、错误处理等常用功能，代码简洁且易于上手，是 Go 生态中最流行的 Web 框架之一。

## Gorm

[Go语言 GORM框架 使用指南](https://blog.csdn.net/lyy42995004/article/details/148031695)

**Gorm** 是 Go 的 ORM（对象关系映射）库，支持 MySQL、PostgreSQL 等数据库，通过结构体模型操作数据，简化了数据库交互。它提供链式调用、事务管理、关联查询等功能，兼顾灵活性和易用性，适合快速开发数据驱动的应用。

两者常结合使用（Gin 处理 HTTP，Gorm 操作数据库）。可以看我写了的两篇博客了解其快速入门并使用。

# Go语言即时通讯系统开发日志day4

> **计划**：搭建项目基本结构，学习了解项目开发（架构、规范、开发了流程）等内容。
>
> **总结**：写了两篇相关博客，[Go语言 Gin框架 使用指南](https://blog.csdn.net/lyy42995004/article/details/148031673)，[Go语言 GORM框架 使用指南](https://blog.csdn.net/lyy42995004/article/details/148031695)，快速浏览了一下 [Go 语言项目开发实战（付费课程）](https://time.geekbang.org/column/intro/100079601)。

## 项目架构

简单了解一下各种项目架构：

**1. 分层架构（Layered Architecture）**

- **核心思想**：将系统分为若干层（如表现层、业务层、数据层），每层仅依赖下层。
- **优点**：结构简单，易于分工和维护。
- **缺点**：层间可能冗余，性能瓶颈（如数据库访问集中）。
- **场景**：传统企业应用（如ERP、CRM）、教学示例。

------

**2. MVC架构（Model-View-Controller）**

- **核心思想**：分离数据（Model）、界面（View）和逻辑（Controller）。
- **优点**：开发快速，适合UI交互多的应用。
- **缺点**：Controller易臃肿，不适合复杂业务逻辑。
- **场景**：Web应用（如博客、内容管理系统）。

------

**3. 微服务架构（Microservices）**

- **核心思想**：将单体应用拆分为独立的小服务，通过API通信。
- **优点**：高扩展性、技术栈灵活、故障隔离。
- **缺点**：分布式复杂性（如事务、网络延迟）、运维成本高。
- **场景**：大型复杂系统（如电商平台、Uber）。

------

**4. 事件驱动架构（Event-Driven Architecture, EDA）**

- **核心思想**：通过事件（消息）异步触发组件行为，松耦合。
- **优点**：高实时性，易于扩展。
- **缺点**：事件顺序难保证，调试复杂。
- **场景**：实时系统（如金融交易、IoT）、最终一致性需求（如库存管理）。

------

**5. 六边形架构（Hexagonal Architecture）**

- **核心思想**：核心业务与外部依赖（如DB、UI）解耦，通过“端口-适配器”交互。
- **优点**：技术无关性，测试友好。
- **缺点**：设计成本高，可能过度工程化。
- **场景**：领域驱动设计（DDD）、长期演化的核心系统（如银行核心业务）。

------

**6. CQRS（Command Query Responsibility Segregation）**

- **核心思想**：读写分离，命令（写操作）和查询（读操作）使用不同模型。
- **优点**：独立优化读写性能，适合高并发查询。
- **缺点**：数据同步延迟，实现复杂。
- **场景**：社交平台、数据分析看板。

------

**7. 无服务器架构（Serverless）**

- **核心思想**：开发者无需管理服务器，按需运行函数（如AWS Lambda）。
- **优点**：零运维成本，自动扩缩容。
- **缺点**：冷启动延迟，不适合长任务。
- **场景**：低频任务（如定时作业、Webhook）、轻量API。

------

**8. 空间基架构（Space-Based Architecture）**

- **核心思想**：通过分布式内存网格（如Redis）共享数据，避免数据库瓶颈。
- **优点**：超高并发、低延迟。
- **缺点**：数据一致性难保证，成本高。
- **场景**：金融交易系统、实时游戏服务器。

------

**9. 插件化架构（Plugin Architecture）**

- **核心思想**：核心系统提供扩展点，功能通过插件动态加载。
- **优点**：灵活扩展，主系统轻量。
- **缺点**：插件管理复杂，接口需严格设计。
- **场景**：IDE（如VS Code）、可定制化软件（如WordPress）。

------

**10. 前后端分离架构（Frontend-Backend Separation）**

- **核心思想**：前端（React/Vue）与后端（API服务）独立开发，通过REST/GraphQL交互。
- **优点**：团队解耦，多端适配（Web/移动）。
- **缺点**：API设计需严谨，SEO需额外处理。
- **场景**：现代Web应用（如社交平台、管理后台）。

------

**11. IAM架构（Identity and Access Management）**

- **核心思想**：集中管理身份认证（Authentication）和权限授权（Authorization）。
- **优点**：统一安全管控，最小权限原则。
- **缺点**：集成复杂，可能成为单点故障。
- **场景**：企业级系统（如OA、云平台）、合规性要求高的领域（金融）。

------

**12. 单体架构（Monolithic Architecture）**

- **核心思想**：所有功能集中在一个代码库中，统一部署。
- **优点**：开发简单，适合小型项目。
- **缺点**：难以扩展，维护成本随规模增长。
- **场景**：初创项目、内部工具。

**如何选择？**

|      **需求**      |    **推荐架构**    |
| :----------------: | :----------------: |
|  快速开发简单应用  |   单体架构、MVC    |
| 高并发、大规模系统 | 微服务、空间基架构 |
|    实时数据处理    |    事件驱动架构    |
|    灵活扩展功能    |     插件化架构     |
|   低成本、无运维   |    无服务器架构    |
|    严格权限控制    |      IAM架构       |
|    读写性能分离    |        CQRS        |
| 长期演化的核心业务 |     六边形架构     |

## 规范设计

**开源规范**

开源项目应遵循明确的许可证（如MIT、Apache-2.0），并在README中说明项目目标、使用方式及贡献指南。代码需保持模块化、可复用，并提供清晰的API文档，方便他人理解和使用。

**文档规范**

项目文档应包括README（快速入门）、API文档（如Swagger）、设计文档（架构图）和贡献指南（PR流程）。文档需结构化、定期更新，并采用Markdown等通用格式，确保易读性和可维护性。

**版本规范**

推荐使用语义化版本（SemVer）：主版本号.次版本号.修订号（如`v1.2.3`），分别对应不兼容改动、新增功能、Bug修复。版本变更需在CHANGELOG中记录，明确影响范围，便于用户升级。

**commit规范**

| 类型     | 类别        | 说明                                                         |
| -------- | ----------- | ------------------------------------------------------------ |
| feat     | Production  | 新增功能                                                     |
| fix      | Production  | Bug修复                                                      |
| perf     | Production  | 提高代码性能的变更                                           |
| style    | Development | 代码格式类的变更，比如用gofmt 格式化代码、删除空行等         |
| refactor | Production  | 其他代码类的变更，这些变更不属于feat、fix、perf 和 style，例 如简化代码、重命名变量、删除冗余代码等 |
| test     | Development | 新增测试用例或是更新现有测试用例                             |
| ci       | Development | 持续集成和部署相关的改动，比如修改Jenkins、GitLab CI等CI 配置文件或者更新systemdunit文件 |
| docs     | Development | 文档类的更新，包括修改用户文档或者开发文档等                 |
| chore    | Development | 其他类型，比如构建流程、依赖管理或者辅助工具的变动等         |

![image.png](https://s2.loli.net/2025/05/17/ZjSHWmAvdbyPaRC.png)

> 参考资料： [Go 语言项目开发实战 - 规范设计（付费课程）](https://time.geekbang.org/column/intro/100079601)

# Go语言即时通讯系统开发日志day5

> **计划**：选择项目架构，搭建项目结构，构建表结构。
>
> **总结**：完成了项目构建，了解了MVC三层架构，了解了zap并对其完成了简单封装，使其可以向`log`库一样使用。

## 项目结构

**构建项目结构**

```
.
├── api
├── cmd
├── configs
├── docs
├── pkg
├── test
├── go.mod
├── go.sum
├── internal
│   ├── config
│   ├── dao
│   ├── dto
│   ├── model
│   └── service
└── web
    ├── public
    └── src
```

1. **`api/`**：存放 API 协议定义文件，用于描述接口规范，不包含具体实现。

2. **`cmd/`**：存放 可执行程序的入口文件（每个子目录对应一个独立的二进制程序）。

3. **`configs/`**：存放 配置文件模板或默认配置（如开发/生产环境配置）。​

4. **`pkg/`**：存放 可复用的公共库代码（允许被其他项目导入）。

5. **`test/`**：存放 集成测试/端到端测试代码（单元测试应直接放在各包内）。

6. **`internal/`**：存放 **私有业务逻辑**（禁止被外部项目导入），是项目核心

   - **`onfig/`**：定义配置结构体及加载逻辑。

   - **`dao/`** (Data Access Object)：数据库操作层（直接与 MySQL/Redis 等交互）。

   - **`dto/`** (Data Transfer Object)：定义 API 请求/响应的数据结构。

   - **`model/`**定义数据库表对应的结构体（ORM 模型）。

   - **`service/`**：实现**核心业务逻辑**（如消息发送、用户鉴权）。

7. **`web/`**：存放 **前端代码和静态资源**。

   - **`public/`**：编译后的静态文件（可直接通过 HTTP 访问）。

   - **`src/`**：前端源码（如 React/Vue 项目）。】

**用户发送消息流程**：

![image.png](https://s2.loli.net/2025/05/18/WuFfRaLx3d9jv6X.png)

> 参考资料：[第 25 章 - Golang 项目结构](https://blog.csdn.net/hummhumm/article/details/143949493) 

## MVC 三层架构

![image.png](https://s2.loli.net/2025/05/18/CxtJbPgdsFYj8lL.png)

**Model 层（数据 + 业务核心）**

对应目录：

- `internal/model/`：定义数据结构（如 `User`、`Message`）
- `internal/dao/`：数据库操作（CRUD）
- `internal/service/`：**业务逻辑**（如发送消息的校验、权限控制）

**View 层（数据展示）**

对应目录：`web/`

- **前端代码**（Vue/React）通过 API 或 WebSocket 获取数据并渲染。

**实时更新**：通过 `pkg/websocket` 监听服务端推送。

**Controller 层（请求协调）**

对应目录：

- `api/`：定义接口协议（如 REST 路由、gRPC 方法）
- `internal/dto/`：请求/响应的数据结构
- `internal/service/`：**实际控制层**（处理参数校验、调用 Model 层）

![image.png](https://s2.loli.net/2025/05/18/CSWFBHG6oxTmRs5.png)

> 参考资料：[MVC 三层架构案例详细讲解](https://blog.csdn.net/BASK2311/article/details/130760490) 

## 日志封装 zap

### 核心结构

```go
type Logger struct {
    l  *zap.Logger       // 实际的 zap logger 实例
    al *zap.AtomicLevel  // 支持动态调整日志级别
}
```

- 封装了 `zap.Logger` 并添加了原子级别的日志级别控制

### 初始化机制

```go
var (
    once     sync.Once   // 确保只初始化一次
    std      *Logger     // 默认 logger 实例
    logPath  string      // 日志路径
    initDone bool        // 初始化标志
)

func initLogger() {
    // 从配置获取日志路径
    conf := config.GetConfig()
    logPath = filepath.Join(conf.LogConfig.LogPath, "app.log")
    
    // 创建日志目录
    os.MkdirAll(filepath.Dir(logPath), 0755)
    
    // 创建带轮转功能的 logger
    std = NewWithRotate(InfoLevel, NewProductionRotateConfig(logPath))
    initDone = true
}

func Default() *Logger {
    once.Do(initLogger)  // 线程安全的单次初始化
    return std
}
```

- 使用 `sync.Once` 确保线程安全的单次初始化
- 自动从配置系统获取日志路径
- 自动创建所需目录结构

> TOML配置文件(`internal/config/config.go`):
>
> ```go
> package config
> 
> import (
> 	"github.com/BurntSushi/toml"
> )
> 
> type LogConfig struct {
> 	LogPath string `toml:"logPath"`
> }
> 
> type Config struct {
> 	LogConfig `toml:"logConfig"`
> }
> 
> var cfg *Config
> 
> func GetConfig() *Config {
> 	if cfg == nil {
> 		cfg = &Config{}
> 		if _, err := toml.DecodeFile("config.toml", cfg); err != nil {
> 			panic(err)
> 		}
> 	}
> 	return cfg
> }
> ```
>
> 对应的TOML文件 (`configs/config.toml`) :
>
> ```toml
> [logConfig]
> logPath = "logs/"
> ```

### **日志轮转实现**

```go
func NewWithRotate(level Level, cfg *RotateConfig, opts ...zap.Option) *Logger {
    // 选择轮转策略
    var writer io.Writer
    if cfg.MaxSize > 0 {
        writer = NewRotateBySize(cfg)
    } else {
        writer = NewRotateByTime(cfg)
    }

    // 创建 zap core
    al := zap.NewAtomicLevelAt(level)
    encoderCfg := zap.NewProductionEncoderConfig()
    encoderCfg.EncodeTime = zapcore.RFC3339TimeEncoder

    core := zapcore.NewCore(
        zapcore.NewJSONEncoder(encoderCfg),
        zapcore.AddSync(writer),
        al,
    )

    return &Logger{
        l:  zap.New(core, opts...),
        al: &al,
    }
}
```

- 根据配置自动选择轮转策略
- 使用 JSON 格式编码日志
- 支持动态日志级别调整

### 日志级别控制

```go
func (l *Logger) SetLevel(level Level) {
    if l.al != nil {
        l.al.SetLevel(level)  // 原子操作修改日志级别
    }
}
```

- 支持运行时动态调整日志级别

### 日志方法

```go
// 各级别日志方法
func (l *Logger) Debug(msg string, fields ...Field)
func (l *Logger) Info(msg string, fields ...Field)
// ...其他级别方法

// 全局快捷方法
func Debug(msg string, fields ...Field) { Default().Debug(msg, fields...) }
// ...其他全局方法
```

- 提供从 Debug 到 Fatal 的全级别日志支持
- 支持结构化日志字段
- 提供全局快捷访问方法

> 代码地址：[IM-Go/pkg/zap](https://github.com/lyy42995004/IM-Go/tree/master/pkg/zap)
>
> 参考资料：[Go 第三方 log 库之 zap 使用](https://jianghushinian.cn/2023/03/19/use-of-zap-in-go-third-party-log-library/)、[如何基于 zap 封装一个更好用的日志库](https://jianghushinian.cn/2023/04/16/how-to-wrap-a-more-user-friendly-logging-package-based-on-zap/)、[github-log](https://github.com/jianghushinian/gokit/blob/main/log/zap/log.go)

