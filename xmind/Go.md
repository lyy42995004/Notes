# 多协程打印abc

## channel

```go
package main

import (
	"fmt"
	"sync"
)

func printCharChan(char string, myChan, nextChan chan struct{}, n int, wg *sync.WaitGroup) {
	defer wg.Done()
	for i := range n {
		<-myChan // 等待自己的 turn
		fmt.Print(char)
		if i == n-1 && char == "c" {
			break;
		}
		nextChan <- struct{}{}
	}
}

func main() {
	n := 10;
	var wg sync.WaitGroup;
	wg.Add(3)

	aChan := make(chan struct{})	
	bChan := make(chan struct{})	
	cChan := make(chan struct{})	

	go printChar("a", aChan, bChan, n, &wg)
	go printChar("b", bChan, cChan, n, &wg)
	go printChar("c", cChan, aChan, n, &wg)

	// 启动第一个协程
	aChan <- struct{}{}

	wg.Wait()
	fmt.Println()
}
```

## 有锁

```go
package main

import (
	"fmt"
	"sync"
)

var mtx sync.Mutex
var cond = sync.NewCond(&mtx)
var turn = 0 // 0: a, 1: b, 2: c

func printCharCond(char string, myTurn, n int, wg *sync.WaitGroup) {
	defer wg.Done()
	for i := 0; i < n; i++ {
		cond.L.Lock()
		for turn != myTurn {
			cond.Wait()
		}
		fmt.Print(char)
		turn = (turn + 1) % 3
		cond.Broadcast() // 唤醒所有等待的 goroutine
		cond.L.Unlock()
	}
}

func main() {
	n := 10
	var wg sync.WaitGroup
	wg.Add(3)

	go printCharCondChan("a", 0, n, &wg)
	go printCharCondChan("b", 1, n, &wg)
	go printCharCondChan("c", 2, n, &wg)

	wg.Wait()
	fmt.Println()
}
```

## 无锁

```go
package main

import (
	"fmt"
	"runtime"
	"sync"
	"sync/atomic"
)

var turn int32 = 0 // 0: a, 1: b, 2: c

func printCharAtomic(char string, myTurn int32, n int, wg *sync.WaitGroup) {
	defer wg.Done()
	for i := 0; i < n; i++ {
		for atomic.LoadInt32(&turn) != myTurn {
			runtime.Gosched() // 出让 CPU，防死循环占满
		}
		fmt.Print(char)
		atomic.StoreInt32(&turn, (myTurn+1)%3)
	}
}

func main() {
	n := 10
	var wg sync.WaitGroup
	wg.Add(3)

	go printCharAtomic("a", 0, n, &wg)
	go printCharAtomic("b", 1, n, &wg)
	go printCharAtomic("c", 2, n, &wg)

	wg.Wait()
	fmt.Println()
}
```

