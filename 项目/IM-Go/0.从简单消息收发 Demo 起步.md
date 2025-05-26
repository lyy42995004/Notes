**Go语言即时通讯系统开发日志day1**，主要模拟实现的一个简单的发送消息和接受消息的小demo，因为也才刚学习go语言的语法，对go的json、net/http库了解不多，所以了解了一下go语言的**encoding/json库**和**net/http库**，以及**websocket**。

> **总结**：实现了极简的”聊天“demo，不能够支持不同用户的显示，无单聊，群聊，未使用MySQL保存数据，Redis作为缓存。希望明天可以完成数据库对数据存储，支持双人聊天，支持查看历史聊天记录。
>
> **学习方法反思**：**不要好高骛远**，只是完成了一个小demo，却在学习过程找了一些github上的高星IM来看，却实现不了，后续在完成聊天模块，联系人模块，群组模块等时可以在查看github的高星项目向其参考学习；**不要追求完美**，学习到一个知识点的时候，如json，http包，不要过多学习掌握即可，不用在浏览器中搜索过多页面进行查看，仅选择一俩个即可，最好可以写一篇简单的博客进行总结加深印象，使用ai也同样，使用一个或两个即可，不要过对比，浪费时间。

之前只是初略的了解了go语言支持**json格式**，但不了解作用是什么，以及json库的作用；所以我写了一个博客，[Go语言：json 作用和语法](https://blog.csdn.net/lyy42995004/article/details/147905549)，简单介绍了 json 的作用（指定字段名、忽略空值、忽略字段），encoding/json的两个常用函数`Marshal`序列化和`Unmarshal`反序列化。

**定义消息类型**

```go
type Message struct {
	Sender    string `json:"sender"`
	Content   string `json:"content"`
	Timestamp int64  `json:"timestamp"`
}
```

# 构建服务器

```go
func main() {
	// 初始化路由
	http.HandleFunc("/ws", handleConnections)

	// 添加静态文件服务，用于测试HTML客户端
	http.Handle("/", http.FileServer(http.Dir("./web")))

	// 启动服务器
	log.Println("Server started on :8080")
	err := http.ListenAndServe(":8080", nil)
	if err != nil {
		log.Fatal("ListenAndServe: ", err)
	}
}

var upgrater = websocket.Upgrader{
	ReadBufferSize:  1024,
	WriteBufferSize: 1024,
}

// 客户端连接
type Client struct {
	conn *websocket.Conn
	send chan []byte
}

func handleConnections(w http.ResponseWriter, r *http.Request) {
	// 升级 HTTP 为 WebSocket
	conn, err := upgrater.Upgrade(w, r, nil)
	if err != nil {
		log.Println(err)
		return
	}
	defer conn.Close()

	// 创建客户端
	client := &Client{
		conn: conn,
		send: make(chan []byte),
	}

	// 启动读写goroutine
	go client.writePump()
	client.readPump()
}
```

- 每个客户端连接由一个`Client`对象表示
- 每个连接使用两个 goroutine 处理读写操作：
  - **读流程**：`readPump`持续读取消息 → 处理消息 → 可能通过`client.send`发送响应
  - **写流程**：其他 goroutine 向`client.send`发送数据 → `writePump`将数据发送到 WebSocket

- 使用通道`client.send`在两个 goroutine 之间传递消息

# 发送消息

```go
func (c *Client) writePump() {
	defer c.conn.Close()
	for {
		select {
		case message, ok := <-c.send:
			if !ok {
				// 通道关闭
				c.conn.WriteMessage(websocket.CloseMessage, []byte{})
				return
			}

			if err := c.conn.WriteMessage(websocket.TextMessage, message); err != nil {
				log.Println("write error:", err)
				return
			}
		}
	}
}
```

WebSocket 客户端的 "写泵"(write pump) 函数，将消息从客户端的发送通道 (c.send) 持续写入 WebSocket 连接。

- 持续监听`c.send`通道，一旦有消息就发送到 WebSocket 服务器
- 当通道关闭时，主动向服务器发送关闭消息并结束函数
- 处理发送过程中可能出现的错误

# 读取消息

```go
func (c *Client) readPump() {
	defer c.conn.Close()
	for {
		_, message, err := c.conn.ReadMessage()
		if err != nil {
			log.Println("read error:", err)
			break
		}
		// 处理接收到的消息
		log.Printf("Received: %s", message)
	}
}
```

WebSocket 客户端的 "读泵"(read pump) 功能，负责持续从服务器接收消息。

- 持续监听 WebSocket 连接，读取服务器发送的消息
- 处理读取过程中出现的错误
- 将接收到的消息进行日志记录或进一步处理

# 运行

1. 确保项目结构如下：

   ```
   im-go/
   ├── main.go
   └── public/
       ├── index.html
       └── client.js
   ```

2. 安装依赖：`go get github.com/gorilla/websocket`

3. 启动服务器：`go run main.go`

4. 打开浏览器访问：`http://localhost:8080`

![](https://s2.loli.net/2025/05/12/I19RfFyhUT5rba7.png)

![ 2025-05-12 173310.png](https://s2.loli.net/2025/05/12/xsv1Az6weXNu2KZ.png)

# 后端代码

**main.go**

```go
package main

import (
	"log"
	"net/http"

	"github.com/gorilla/websocket"
)

type Message struct {
	Sender    string `json:"sender"`
	Content   string `json:"content"`
	Timestamp int64  `json:"timestamp"`
}

func main() {
	// 初始化路由
	http.HandleFunc("/ws", handleConnections)

	// 添加静态文件服务，用于测试HTML客户端
	http.Handle("/", http.FileServer(http.Dir("./web")))

	// 启动服务器
	log.Println("Server started on :8080")
	err := http.ListenAndServe(":8080", nil)
	if err != nil {
		log.Fatal("ListenAndServe: ", err)
	}
}

var upgrater = websocket.Upgrader{
	ReadBufferSize:  1024,
	WriteBufferSize: 1024,
}

// 客户端连接
type Client struct {
	conn *websocket.Conn
	send chan []byte
}

func handleConnections(w http.ResponseWriter, r *http.Request) {
	// 升级 HTTP 为 WebSocket
	conn, err := upgrater.Upgrade(w, r, nil)
	if err != nil {
		log.Println(err)
		return
	}
	defer conn.Close()

	// 创建客户端
	client := &Client{
		conn: conn,
		send: make(chan []byte),
	}

	// 启动读写goroutine
	go client.writePump()
	client.readPump()
}

func (c *Client) readPump() {
	defer c.conn.Close()
	for {
		_, message, err := c.conn.ReadMessage()
		if err != nil {
			log.Println("read error:", err)
			break
		}
		// 处理接收到的消息
		log.Printf("Received: %s", message)
	}
}

func (c *Client) writePump() {
	defer c.conn.Close()
	for {
		select {
		case message, ok := <-c.send:
			if !ok {
				// 通道关闭
				c.conn.WriteMessage(websocket.CloseMessage, []byte{})
				return
			}

			if err := c.conn.WriteMessage(websocket.TextMessage, message); err != nil {
				log.Println("write error:", err)
				return
			}
		}
	}
}
```

# 前端代码

前端代码虽然是ai生成的，我还是看了 [3小时前端入门教程（HTML+CSS+JS）](https://www.bilibili.com/video/BV1BT4y1W7Aw/)，了解了一下这三者是什么，以及作用

**HTML（超文本标记语言）**

- **是什么**：
  HTML 是网页的​**​结构和内容层​**​，通过标签（如 `<h1>`、`<p>`、`<div>`）定义页面的文本、图片、链接等元素。
- 作用：
  - 搭建网页的骨架（例如标题、段落、表单）。
  - 提供语义化结构（如 `<header>`、`<article>`），便于搜索引擎和屏幕阅读器理解。

**CSS（层叠样式表）**

- **是什么**：
  CSS 是网页的​**​表现层​**​，控制 HTML 元素的外观（颜色、布局、字体等）。
- 作用：
  - 美化页面（如响应式设计、动画效果）。
  - 实现布局（Flexbox、Grid 等）。

**JavaScript（JS）**

- **是什么**：
  JavaScript 是网页的​**​行为层​**​，一种编程语言，用于实现交互和动态功能。
- 作用：
  - 处理用户操作（点击、输入等）。
  - 动态更新内容（如加载数据、DOM 操作）。
  - 与后端交互（通过 AJAX 或 Fetch API）。

后续的话前端可能就用ai生成**vue**的了

**Vue 是什么？**

一个用于构建用户界面的 **JavaScript 框架**，核心特点是：

1. **响应式数据**：数据变，视图自动更新。
2. **组件化**：把页面拆成可复用的组件。
3. **易上手**：类似 HTML 的模板语法，学习成本低。

**Vue 的作用**

- 替代 jQuery，**动态交互更简单**（如实时搜索、表单验证）。
- 开发**单页应用（SPA）**（如后台管理系统）。
- 搭配插件（Vue Router、Pinia）做复杂项目。

**index.html**

```html
<!DOCTYPE html>
<html>

<head>
    <title>Go Chat Demo</title>
    <script src="client.js"></script>
</head>

<body>
    <h1>Go Chat Demo</h1>
    <div id="messages" style="height:300px;overflow:auto;border:1px solid #ccc;"></div>
    <input type="text" id="messageInput" placeholder="Type a message...">
    <button onclick="sendMessage()">Send</button>
</body>

</html>
```

**client.js**

```js
var ws;

function connect() {
    ws = new WebSocket("ws://" + window.location.host + "/ws");

    ws.onopen = function () {
        console.log("Connected to server");
        addMessage("System: Connected to server");
    };

    ws.onmessage = function (event) {
        console.log("Message received:", event.data);
        addMessage("Server: " + event.data);
    };

    ws.onclose = function () {
        console.log("Disconnected from server");
        addMessage("System: Disconnected from server");
        // 尝试5秒后重连
        setTimeout(connect, 5000);
    };

    ws.onerror = function (err) {
        console.log("WebSocket error:", err);
    };
}

function addMessage(msg) {
    var messages = document.getElementById("messages");
    messages.innerHTML += "<p>" + msg + "</p>";
    messages.scrollTop = messages.scrollHeight;
}

function sendMessage() {
    var input = document.getElementById("messageInput");
    var message = input.value;
    if (message && ws.readyState === WebSocket.OPEN) {
        ws.send(message);
        addMessage("You: " + message);
        input.value = "";
    }
}

// 页面加载时连接
window.onload = connect;
```

