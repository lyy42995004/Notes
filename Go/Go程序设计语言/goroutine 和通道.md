# goroutine

```go
f() 	// 等待 f() 返回
go f()  // 新建一个调用 f() 的 goroutine，不用等待
```

在 Go 语言里，goroutine 是并发执行的活动单元。与顺序执行程序不同，在有多个 goroutine 的并发程序中，不同函数可同时执行。程序启动时，首个调用`main`函数的 goroutine 为主 goroutine，新的 goroutine 通过`go`语句创建，`go`语句在函数或方法调用前加上`go`关键字，且`go`语句本身执行立即完成，不等待函数执行结束。

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	go sinner(100 * time.Microsecond)
	const n = 45
	fibN := fib(n)
	fmt.Printf("\rFibonacci(%d) = %d\n", n, fibN)
}

func sinner(delay time.Duration) {
	for {
		for _, r := range `-\|/` {
			fmt.Printf("\r%c", r)
			time.Sleep(delay)
		}
	}
}

func fib(x int) int {
	if x < 2 {
		return x
	}
	return fib(x - 1) + fib(x - 2)
}
```

**示例**：主 goroutine 计算第 45 个斐波那契数（使用低效递归算法，耗时较长），同时启动另一个 goroutine 运行`spinner`函数，`spinner`函数不断打印旋转字符提示程序在运行。当`fib(45)`计算完成，`main`函数输出结果后返回，此时所有 goroutine 强制终结，程序退出。除程序正常返回或退出外，无法从外部直接停止一个 goroutine，但可通过通信让其自行停止。**程序中两个并发活动（指示器显示和斐波那契数计算）相互独立运行** 。

# 示例：并发时钟服务器

**顺序时钟服务器**

```go
func main() {
	listener, err := net.Listen("tcp", "localhost:8000")
	if err != nil {
		log.Fatal(err)
	}
	for {
		conn, err := listener.Accept()
		if err != nil {
			log.Print(err)
			continue
		}
		handleConn(conn)
	}
}

func handleConn(conn net.Conn) {
	defer conn.Close()
	for {
		_, err := io.WriteString(conn, time.Now().Format("15:04:05\n"))
		if err != nil {
			return
		}
		time.Sleep(1 * time.Second)
	}
}
```

- **实现原理**：使用`net.Listen`创建`net.Listener`在`localhost:8000`监听 TCP 连接，`Accept`方法阻塞等待连接，接收到连接后由`handleConn`函数处理。`handleConn`函数在循环中，每秒通过`time.Now`获取当前时间，利用`net.Conn`满足`io.Writer`接口的特性，将时间格式化后写入连接发送给客户端。若写入失败（如客户端断开连接），则关闭连接，继续等待下一个连接请求。

```go
func main() {
	conn, err := net.Dial("tcp", "localhost:8000")
	if err != nil {
		log.Fatal(err)
	}
	defer conn.Close()
	mustCopy(os.Stdout, conn)
}

func mustCopy(dst io.Writer, src io.Reader) {
	if _, err := io.Copy(dst, src); err != nil {
		log.Fatal(err)
	}
}
```

- **客户端连接**：客户端可使用`telnet`，`nc`（`netcat`）工具或自定义的基于`net.Dial`实现的 Go 版`netcat`程序连接服务器。顺序服务器一次只能处理一个客户端请求，第二个客户端需等第一个结束才能正常工作。

**并发时钟服务器**

```go
func main() {
	listener, err := net.Listen("tcp", "localhost:8000")
	if err != nil {
		log.Fatal(err)
	}
	for {
		conn, err := listener.Accept()
		if err != nil {
			log.Print(err)
			continue
		}
		go handleConn(conn)
	}
}
```

在顺序时钟服务器基础上，只需在调用`handleConn`函数处添加`go`关键字，使其在新的 goroutine 内执行，就能让服务器并发处理多个客户端连接，多个客户端可同时接收到时间信息。

# 示例：并发回声服务器

**简单回声服务器实现**

```go
func handleConn(c net.Conn) {
    io.Copy(c, c) // 注意: 忽略错误
    c.Close()
}
```

普通回声服务器可通过`handleConn`函数，利用`io.Copy`将读取到的内容写回客户端，之后关闭连接，但此版本忽略错误处理。

**模拟真实回声的回声服务器**

```go
func echo(c net.Conn, shout string, delay time.Duration) {
    fmt.Fprintln(c, "\t", strings.ToUpper(shout))
    time.Sleep(delay)
    fmt.Fprintln(c, "\t", shout)
    time.Sleep(delay)
    fmt.Fprintln(c, "\t", strings.ToLower(shout))
}

func handleConn(c net.Conn) {
    input := bufio.NewScanner(c)
    for input.Scan() {
        echo(c, input.Text(), 1*time.Second)
    }
    // 注意: 忽略input.Err()中可能的错误
    c.Close()
}
```

- **`echo`函数**：定义`echo`函数，接收`net.Conn` 、字符串和延迟时间参数，先将字符串转为大写输出，延迟后输出原字符串，再延迟后转为小写输出，模拟真实回声效果。
- **`handleConn`函数**：在`handleConn`函数中，使用`bufio.NewScanner`读取客户端输入，循环调用`echo`函数处理输入内容，同样忽略了输入错误处理，处理完后关闭连接。

```go
func main() {
    conn, err := net.Dial("tcp", "localhost:8000")
    if err!= nil {
        log.Fatal(err)
    }
    defer conn.Close()
    go mustCopy(os.Stdout, conn)
    mustCopy(conn, os.Stdin)
}

```

- **`netcat`程序**：客户端程序使用`net.Dial`连接服务器，通过两个`mustCopy`调用，一个将标准输入发送到服务器，另一个将服务器回复输出到标准输出，主 goroutine 处理输入，另一个 goroutine 处理输出，输入结束时程序停止。

**实现并发回声效果**

```go
func handleConn(c net.Conn) {
    input := bufio.NewScanner(c)
    for input.Scan() {
        go echo(c, input.Text(), 1*time.Second)
    }
    // 注意: 忽略input.Err()中可能的错误
    c.Close()
}
```

为使回声真正并发，在调用`echo`函数时添加`go`关键字，让每个回声处理在单独的 goroutine 中执行，实现多个回声在时间上相互重合的并发效果。在添加`go`关键字实现并发时，要考虑`net.Conn`并发调用的安全性。

# 通道

通道是 Go 程序中 goroutine 之间的通信机制，可看作是特定类型元素的导管，如`chan int`表示存放`int`类型元素的通道。它是 goroutine 间发送特定值的通信桥梁，是并发编程中实现同步和数据传递的重要工具。

```go
ch := make(chan int)
```

- **创建方式**：使用内置的`make`函数创建通道，如`ch := make(chan int)` 。通道和`map`类似，是引用类型，复制或传递时传递的是引用，指向同一份数据结构，其零值为`nil` 。同种类型通道可用`==`比较，若为同一通道引用则比较值为`true` ，也可与`nil`比较 。

```go
ch <- x  // 发送语句
x = <-ch // 接收语句
<-ch 	 // 丢弃
```

- **发送与接收**：通道有发送（`send` ）和接收（`receive` ）两个主要操作，使用`<-`操作符。发送语句如`ch <- x` ；接收语句如`x = <-ch` ，也可丢弃接收结果，如`<-ch` 。

```go
close(ch)
```

- **关闭操作**：可使用`close`函数关闭通道，关闭后发送操作会导致宕机，接收操作会获取已发送的值，通道为空时接收立即完成并获取元素类型零值 。

**类型**

```go
ch = make(chan int)    // 无缓冲通道
ch = make(chan int, 0) // 无缓冲通道
ch = make(chan inr, 3) // 容量为3的缓冲通道
```

使用简单`make`调用创建的是无缓冲通道，`make`还可接受第二个可选参数表示通道容量。容量为 0 时是无缓冲通道，如`ch = make(chan int)`或`ch = make(chan int, 0)` ；指定容量（如`ch = make(chan int, 3)` ）则为缓冲通道 ，下面会再进行介绍缓冲通道 。

## 无缓冲通道

无缓冲通道上的发送操作会阻塞，直到有另一个 goroutine 在对应通道上执行接收操作，此时值传送完成，两个 goroutine 可继续执行；反之，接收操作先执行也会阻塞，直到有 goroutine 发送值。这种通信机制使发送和接收的 goroutine 同步化，所以无缓冲通道也叫同步通道 。

```go
func main() {
    conn, err := net.Dial("tcp", "localhost:8000")
    if err!= nil {
        log.Fatal(err)
    }
    done := make(chan struct{})
    go func() {
        io.Copy(os.Stdout, conn) // 注意: 忽略错误
        log.Println("done")
        done <- struct{}{} // 指示主 goroutine
    }()
    mustCopy(conn, os.Stdin)
    conn.Close()
    <-done // 等待后台 goroutine 完成
}
```

**示例**：客户端程序基础上，通过创建无缓冲通道`done`来同步主 goroutine 和后台 goroutine。主 goroutine 等待从`done`通道接收值，后台 goroutine 在完成任务（如`io.Copy`操作）后，向`done`通道发送一个值，主 goroutine 接收到值后才继续执行后续操作（如关闭连接 ）。当用户关闭标准输入流，`mustCopy`返回，后台 goroutine 记录消息并向`done`通道发送值，主 goroutine 接收到值后关闭连接，保证程序按预期顺序执行 。

**消息与事件的概念**

- **通过通道发送消息时，不仅消息的值重要，通信本身及发生时间也很关键，这种用于同步、不携带额外信息的消息称为事件**。可使用`struct{}`元素类型的通道来强调同步功能，`bool`或`int`类型通道也可以 。

## 管道

管道是利用通道连接 goroutine 的一种方式，使一个 goroutine 的输出成为另一个的输入。通过通道在多个包含无限循环的 goroutine 间进行全生命周期通信 。

```go
func main() {
    naturals := make(chan int)
    squares := make(chan int)

    // counter
    go func() {
        for x := 0; x < 100; x++ {
            naturals <- x
        }
        close(naturals)
    }()

    // squarer
    go func() {
        for x := range naturals {
            squares <- x * x
        }
        close(squares)
    }()

    // printer (在主 goroutine 中)
    for x := range squares {
        fmt.Println(x)
    }
}
```



![image.png](https://s2.loli.net/2025/04/15/iRyv9zClxwnMDGX.png)

- **示例**：程序由`counter` 、`squarer` 、`printer`三个 goroutine 和两个通道组成。`counter`产生自然数序列并通过`naturals`通道发送给`squarer` ，`squarer`计算平方后通过`squares`通道发送给`printer` ，`printer`输出结果 。
- **通道关闭处理**：当发送方知道无更多数据发送时，可关闭通道告知接收方停止等待 。如`counter`在发送一定数量元素（如 100 个 ）后关闭`naturals`通道，`squarer`接收到通道关闭信号后处理并关闭`squares`通道 。接收操作有变体，可返回接收值和表示是否成功的布尔值 ，利用此特性可判断通道是否关闭并处理 。也可使用`range`循环语法，在通道接收完所有值后自动结束循环 ，简化通道关闭和数据处理逻辑 。

- **关闭通道的必要性**：不是必须的，仅在需通知接收方数据发送完毕时进行 。通道回收由垃圾回收器根据可访问性决定，与文件关闭操作不同 。关闭已关闭通道会导致宕机，关闭通道还可作为广播机制  。

## 缓冲通道

```go
ch = make(chan string, 3)
```

- **创建**：缓冲通道通过`make`函数创建，可设置容量参数，如`ch = make(chan string, 3)`创建一个能容纳三个字符串的缓冲通道 。
- **发送与接收操作**：发送操作向通道队列尾部插入元素，接收操作从头部移除元素 。通道满时发送操作阻塞，通道空时接收操作阻塞，部分填满时发送和接收操作不阻塞 。可通过`cap`函数获取通道容量，`len`函数获取当前元素个数 。

```go
func mirroredQuery() string {
    responses := make(chan string, 3)
    go func() { responses <- request("asia.gopl.io") }()
    go func() { responses <- request("europe.gopl.io") }()
    go func() { responses <- request("americas.gopl.io") }()
    return <-responses // return the quickest response
}
```

**示例**：它并发向三个镜像地址发送请求，将响应通过缓冲通道发送，只接收最早返回的响应 。缓冲通道在并发场景中解耦发送和接收 goroutine 的作用 。同时使用无缓冲通道可能导致 goroutine 泄漏（长时间阻塞无法结束 ），要合理选择无缓冲或缓冲通道以及设置缓冲通道容量。

**缓冲通道与性能**

以蛋糕店厨师工作场景类比，无缓冲通道类似厨师需等待下一个接收，同步性强；缓冲通道可容纳一定量任务，容量为 1 时可消除速率差异，容量更大可应对更大速率波动 。还提到可通过创建额外 goroutine 辅助处理任务，以优化程序性能 。

# 并行循环

- **初步并行**：直接在循环中添加`go`关键字启动 goroutine 进行并行处理，但此版本存在问题，外层 goroutine 可能在内部 goroutine 完成任务前就返回，导致任务未真正完成 。
- **使用通道同步**：创建无缓冲通道，内层 goroutine 完成任务时向通道发送信号，外层 goroutine 通过接收通道信号计数，等待所有任务完成 。此过程需注意循环变量在闭包中的使用问题，避免 goroutine 读取到错误的变量值 。
- **处理错误返回**：为使外层 goroutine 能获取内层 goroutine 执行函数的错误，创建用于接收错误的通道，内层 goroutine 将错误发送到通道，外层 goroutine 接收并处理错误 。但简单处理方式可能导致 goroutine 泄漏（如遇到错误时未正确结束 goroutine ），可通过使用有足够容量的缓冲通道或其他方案解决 。
- **返回处理结果**：进一步改进，创建缓冲通道，内层 goroutine 将生成的内容及错误信息发送到通道，外层 goroutine 接收并处理 。

**使用`sync.WaitGroup`同步**

```go
func makeThumbnails6(filenames <-chan string) int64 {
    sizes := make(chan int64)
    var wg sync.WaitGroup // 工作goroutine的个数
    for f := range filenames {
        wg.Add(1)
        go func(f string) {
            defer wg.Done()
            thumb, err := thumbnail.ImageFile(f)
            if err != nil {
                log.Println(err)
                return
            }
            info, _ := os.Stat(thumb) // 可以忽略错误
            sizes <- info.Size()
        }(f)
    }
    // closer
    go func() {
        wg.Wait()
        close(sizes)
    }()
    var total int64
    for size := range sizes {
        total += size
    }
    return total
}
```

为更好地控制和等待所有 goroutine 结束，引入`sync.WaitGroup` 。在启动每个工作 goroutine 前使用`Add`方法增加计数，工作 goroutine 结束时调用`Done`方法减少计数，主 goroutine 通过`Wait`方法阻塞等待计数为 0，即所有工作 goroutine 完成 。同时，结合通道传递处理结果（如文件大小 ），实现更健壮的并行处理 。

# 使用 select 多路复用

```go
select {
case <-ch1:
    //...
case x := <-ch2:
    //...use x...
case ch3 <- y:
    //...
default:
    //...
}
```

`select`语句用于在多个通道操作中进行选择，实现多路复用。它类似`switch`语句，有一系列情况和可选的默认分支，每个情况指定一次通道上的发送或接收操作及关联代码块 。`select`会一直等待，直到有一个通道操作可执行，然后执行对应语句，其他未满足条件的操作不会执行；若没有对应情况且无默认分支，`select`将永远等待 。

**示例**

```go
func main() {
    fmt.Println("Commencing countdown.")
    tick := time.Tick(1 * time.Second)
    for countdown := 10; countdown > 0; countdown-- {
        fmt.Println(countdown)
        <-tick
    }
    launch()
}
```

- **火箭发射倒计时**：以火箭发射倒计时为例，最初的倒计时程序通过`time.Tick`函数按固定时间间隔发送事件进行倒计时 。为实现可取消发射，启动一个 goroutine 从标准输入读取字符，若成功则向`abort`通道发送值 。使用`select`语句等待`time.Tick`通道的计时事件或`abort`通道的取消事件 。还可结合`time.After`函数设置超时，若在指定时间（如 10s ）内未收到取消事件则开始发射 。

```go
ch := make(chan int, 1)
for i := 0; i < 10; i++ {
    select {
    case x := <-ch:
        fmt.Println(x) // "0" "2" "4" "6" "8"
    case ch <- i:
    }
}
```

- **通道状态操作选择**：对于缓冲区大小为 1 的通道`ch` ，通过`select`语句根据通道状态（空或满 ）及循环变量`i`的奇偶性，决定是从通道接收还是向通道发送数据 。当多个情况同时满足时，`select`随机选择一个执行 。

```go
func main() {
    //...创建中止通道...
    fmt.Println("Commencing countdown.  Press return to abort.")
    tick := time.Tick(1 * time.Second)
    for countdown := 10; countdown > 0; countdown-- {
        fmt.Println(countdown)
        select {
        case <-tick:
            // 什么操作也不执行
        case <-abort:
            fmt.Println("Launch aborted!")
            return
        }
    }
    launch()
}
```

- **带输出的倒计时**：改进火箭发射倒计时程序，在每次迭代中使用`select`语句使程序等待 1s 以检测中止事件，同时输出倒计时数值 。

**注意事项**

```go
ticker := time.NewTicker(1 * time.Second)
<-ticker.C // 从ticker的通道接收
ticker.Stop() // 造成ticker的goroutine终止

select {
case <-abort:
    fmt.Printf("Launch aborted!\n")
    return
default:
    // 不执行任何操作
}
```

- `time.Tick`函数使用可能导致的 goroutine 泄漏问题，因为即使不再接收其通道事件，对应的 goroutine 仍在运行 。建议使用`time.NewTicker`创建定时器，使用完后通过`Stop`方法终止相关 goroutine 。
- `select`语句可实现非阻塞通信，通过添加默认分支，在没有通道操作就绪时立即执行默认代码块 ，重复此操作可实现对通道轮询 。
- `nil`通道上的操作永远不会被选中 ，可利用这一特性开启或禁用特定情况 。

# 取消

在一些场景下，需要让 goroutine 停止当前任务，如 Web 服务器处理客户端请求时客户端断开连接 。但直接终止一个 goroutine 会使共享变量状态不确定，且难以准确知道有多少 goroutine 在工作，简单通过通道发送固定数量事件来取消多个 goroutine 存在计数不准确、操作卡住等问题 。

**基于通道关闭的广播式取消机制**

```go
var done = make(chan struct{})
func cancelled() bool {
    select {
    case <-done:
        return true
    default:
        return false
    }
}
```

利用通道关闭特性实现广播式取消 。创建取消通道`done` ，不向其发送值，而是通过关闭它来表明取消操作 。定义`cancelled`函数，使用`select`语句检测`done`通道是否关闭 ，若关闭则返回`true` ，否则返回`false` 。同时，启动一个 goroutine 读取标准输入，一旦检测到输入（如用户按回车键 ），就关闭`done`通道广播取消事件 。

```go
// 当检测到输入时取消遍历
go func() {
    os.Stdin.Read(make([]byte, 1)) // 读一个字节
    close(done)
}()

for {
    select {
    case <-done:
        // 耗尽fileSizes以允许已有的goroutine结束
        for range fileSizes {
            // 不执行任何操作
        }
        return
    case size, ok := <-fileSizes:
        //...
    }
}
```

- **主 goroutine**：在主 goroutine 的`select`语句中添加从`done`通道接收的情况 。当接收到取消信号时，先耗尽`fileSizes`通道中的值（防止卡住 ），然后返回 。

```go
func walkDir(dir string, n *sync.WaitGroup, fileSizes chan<- int64) {
    defer n.Done()
    if cancelled() {
        return
    }
    for _, entry := range dirents(dir) {
        //...
    }
}
```

- **`walkDir`函数**：`walkDir`函数开始时轮询取消状态，若检测到取消（`cancelled`为`true` ），则立即返回 ，不再执行后续操作 。在`walkDir`循环中也可进行取消状态轮询 ，避免取消后创建新的 goroutine 。

```go
func dirents(dir string) []os.FileInfo {
    select {
    case sema <- struct{}{}: // 获取令牌
    case <-done:
        return nil // 取消
    }
    defer func() { <-sema }() // 释放令牌
    //...read directory...
}
```

- **`dirents`函数**：在`dirents`函数中，使用`select`语句同时处理获取信号量令牌和检测取消通道`done` ，若检测到取消则直接返回`nil` ，实现更快速的取消响应 。通过这些措施，当取消事件发生时，后台 goroutine 能迅速停止，`main`函数返回，程序退出。

> 参考资料：《Go程序设计语言》