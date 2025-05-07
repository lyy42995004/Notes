# 1

```go
c.mu.RUnlock() // 58 Unlock() -> RUnlock()
```

```go
c.usedBytes += int64(len(key) + value.Len()) // 122 = -> +=
```

# 2

```go
c.dlnk[c.last] = [2]uint16{0, c.dlnk[0][suc]} // 330 c.dlnk[0][pred] -> [suc]
```

```go
// 341 顺序
prev, next := c.dlnk[idx][p], c.dlnk[idx][s]
c.dlnk[next][p], c.dlnk[prev][s] = prev, next

// 插入
c.dlnk[idx][p] = 0            // 更新当前节点的前置节点为哨兵节点
c.dlnk[idx][s] = c.dlnk[0][s] // 更新当前节点的后继节点为原头节点

c.dlnk[c.dlnk[0][s]][p] = idx // 更新原头节点的前置节点
c.dlnk[0][s] = idx            // 更新哨兵节点的后继节点
```

```go
func init() {
	go func() {
		for {
			atomic.StoreInt64(&clock, time.Now().UnixNano()) // 每秒校准一次
			for range 9 {
				time.Sleep(100 * time.Millisecond)
				atomic.AddInt64(&clock, int64(100*time.Millisecond)) // 保持 clock 在一个精确的时间范围内，同时避免频繁的系统调用
			}
			time.Sleep(100 * time.Millisecond)
		}
	}()
} // 247
```

```go
s.locks[i].Unlock() // 174
```

