# TimerNode 结构体

```C++
struct TimerNode {
    int id;
    TimeStamp expires;  // 超时时间点
    TimeoutCallBack cb; //  回调函数
    TimerNode(int id_, TimeStamp expires_, TimeoutCallBack cb_) 
        : id(id_), expires(expires_), cb(cb_) {}
    bool operator<(const TimerNode& t) {
        return expires < t.expires;
    }
    bool operator>(const TimerNode& t) {
        return expires > t.expires;
    }
};
```

- `id`：定时器的唯一标识符。
- `expires`：定时器的超时时间点。
- `cb`：定时器到期时要执行的回调函数。
- 重载了 `<` 和 `>` 运算符，用于比较两个定时器节点的超时时间，用于后续建堆。

# HeapTimer 类

 `HeapTimer`类是**基于小根堆的定时器**，用于管理多个定时任务。在 webserver 中，定时器是一种重要的工具，常用于处理连接超时、定时任务等场景。使用最小堆来管理定时器的好处是可以高效地插入、删除和获取最近到期的定时器。

```C++
class HeapTimer {
public:
    HeapTimer() { heap_.reserve(64); }
    ~HeapTimer() { Clear(); }

    void Add(int id, int timeout, const TimeoutCallBack& cb);
    void Adjust(int id, int timeout);
    void DoWork(int id);
    void Tick();
    int GetNextTick();
    void Pop();
    void Clear();

    size_t Size() { return heap_.size(); }

private:
    void SwapNode(size_t i, size_t j);
    void SiftUp(size_t child);
    bool SiftDown(size_t parent, size_t n);
    void Delete(size_t i);

    std::vector<TimerNode> heap_;
    // id 对应的在 heap_ 中的下标
    std::unordered_map<int, size_t> ref_;
};
```

# 堆操作函数

## 实现 SwapNode() 函数

```C++
void HeapTimer::SwapNode(size_t i, size_t j) {
    assert(i < heap_.size());
    assert(j < heap_.size());
    std::swap(heap_[i], heap_[j]);
    ref_[heap_[i].id] = i;
    ref_[heap_[j].id] = j;
}
```

## 实现 SiftUp() 函数

```C++
// 向上调整算法
void HeapTimer::SiftUp(size_t child) {
    assert(child < heap_.size());
    size_t parent = (child - 1) / 2;
    while (parent > 0) {
        if (heap_[parent] > heap_[child]) {
            SwapNode(child, parent);
            child = parent;
            parent = (child - 1) / 2;
        } else {
            break;
        }
    }
}
```

## 实现 SiftDown() 函数

```C++
// 向下调整算法
// 返回：是否调整
bool HeapTimer::SiftDown(size_t parent, size_t n) {
    assert(parent < heap_.size());
    assert(n < heap_.size());
    size_t child = 2 * parent + 1; // 左孩子
    size_t pos = parent;
    while (child < n) {
        if (child + 1 < n && heap_[child + 1] < heap_[child])
            ++child; // 右孩子
        if (heap_[parent] > heap_[child]) {
            SwapNode(child, parent);
            parent = child;
            child = 2 * parent + 1;
        } else {
            break;
        }
    }
    return pos < parent;
}
```

## 实现 Delete() 函数

- 将该节点与堆的最后一个节点交换位置。
- 调用 `SiftDown` 函数，如果未进行向下调整，再调用 `SiftUp` 函数调整堆。

```C++
void HeapTimer::Delete(size_t i) {
    assert(!heap_.empty() && i < heap_.size());
    size_t n = heap_.size() - 1;
    if (i < n - 1) { // 跳过n
        SwapNode(i, n);
        if (!SiftDown(i, n))
            SiftUp(i);
    }
    ref_.erase(heap_.back().id);
    heap_.pop_back();
}
```

# 实现 Add() 函数

```C++
void HeapTimer::Add(int id, int timeout, const TimeoutCallBack& cb) {
    if (ref_.count(id)) {
        Adjust(id, timeout);
        heap_[ref_[id]].cb = cb;
    } else {
        size_t n = heap_.size();
        ref_[id] = n;
        heap_.emplace_back(id, Clock::now() + MS(timeout), cb);
        SiftUp(n);
    }
}
```

# 实现 Adjust() 函数

```C++
void HeapTimer::Adjust(int id, int timeout) {
    assert(ref_.count(id));
    heap_[ref_[id]].expires = Clock::now() + MS(timeout);
    if (!SiftDown(ref_[id], heap_.size()))
        SiftUp(ref_[id]);
}
```

# 实现 DoWork() 函数

```C++
void HeapTimer::DoWork(int id) {
    if (heap_.empty() || ref_.count(id))
        return;
    size_t i = ref_[id];
    heap_[i].cb();
    Delete(i);
}
```

# 实现 Tick() 函数

```C++
void HeapTimer::Tick() {
    if (heap_.empty())
        return;
    while (!heap_.empty()) {
        TimerNode node = heap_.front();
        if (std::chrono::duration_cast<MS>(node.expires - Clock::now()).count() > 0)
            break;
        node.cb();
        Pop();
    }
}
```

# 实现 GetNextTick() 函数

```C++
int HeapTimer::GetNextTick() {
    Tick();
    int res = -1;
    if (!heap_.empty()) {
        res = std::chrono::duration_cast<MS>(heap_.front().expires - Clock::now()).count();
        if (res < 0) res = 0;   
    }
    return res;
}
```

# 实现 Pop() 函数

```C++
void HeapTimer::Pop() {
    assert(!heap_.empty());
    Delete(0);
}
```

# 实现 Clear() 函数

```C++
void HeapTimer::Clear() {
    heap_.clear();
    ref_.clear();
}
```

# HeapTimer 代码

**heap_timer.h**

```C++
#ifndef HEAP_TIMER_H
#define HEAP_TIMER_H

#include <cassert>
#include <vector>
#include <unordered_map>
#include <algorithm>
#include <chrono>
#include <functional>

typedef std::function<void()> TimeoutCallBack;
typedef std::chrono::high_resolution_clock Clock;
typedef std::chrono::milliseconds MS;
typedef std::chrono::high_resolution_clock::time_point TimeStamp;

// 定时器节点
struct TimerNode {
    int id;
    TimeStamp expires;  // 超时时间点
    TimeoutCallBack cb; //  回调函数
    TimerNode(int id_, TimeStamp expires_, TimeoutCallBack cb_) 
        : id(id_), expires(expires_), cb(cb_) {}
    bool operator<(const TimerNode& t) {
        return expires < t.expires;
    }
    bool operator>(const TimerNode& t) {
        return expires > t.expires;
    }
};

class HeapTimer {
public:
    HeapTimer() { heap_.reserve(64); }
    ~HeapTimer() { Clear(); }

    void Add(int id, int timeout, const TimeoutCallBack& cb);
    void Adjust(int id, int timeout);
    void DoWork(int id);
    void Tick();
    int GetNextTick();
    void Pop();
    void Clear();

    size_t Size() { return heap_.size(); }

private:
    void SwapNode(size_t i, size_t j);
    void SiftUp(size_t child);
    bool SiftDown(size_t parent, size_t n);
    void Delete(size_t i);

    std::vector<TimerNode> heap_;
    // id 对应的在 heap_ 中的下标
    std::unordered_map<int, size_t> ref_;
};

#endif // HEAP_TIMER_H
```

**heap_timer.cc**

```C++
#include "heap_timer.h"

void HeapTimer::Add(int id, int timeout, const TimeoutCallBack& cb) {
    if (ref_.count(id)) {
        Adjust(id, timeout);
        heap_[ref_[id]].cb = cb;
    } else {
        size_t n = heap_.size();
        ref_[id] = n;
        heap_.emplace_back(id, Clock::now() + MS(timeout), cb);
        SiftUp(n);
    }
}

void HeapTimer::Adjust(int id, int timeout) {
    assert(ref_.count(id));
    heap_[ref_[id]].expires = Clock::now() + MS(timeout);
    if (!SiftDown(ref_[id], heap_.size()))
        SiftUp(ref_[id]);
}

// 触发回调函数，并删除id
void HeapTimer::DoWork(int id) {
    if (heap_.empty() || ref_.count(id))
        return;
    size_t i = ref_[id];
    heap_[i].cb();
    Delete(i);
}

void HeapTimer::Tick() {
    if (heap_.empty())
        return;
    while (!heap_.empty()) {
        TimerNode node = heap_.front();
        if (std::chrono::duration_cast<MS>(node.expires - Clock::now()).count() > 0)
            break;
        node.cb();
        Pop();
    }
}

int HeapTimer::GetNextTick() {
    Tick();
    int res = -1;
    if (!heap_.empty()) {
        res = std::chrono::duration_cast<MS>(heap_.front().expires - Clock::now()).count();
        if (res < 0) res = 0;   
    }
    return res;
}

void HeapTimer::Pop() {
    assert(!heap_.empty());
    Delete(0);
}

void HeapTimer::Clear() {
    heap_.clear();
    ref_.clear();
}

void HeapTimer::SwapNode(size_t i, size_t j) {
    assert(i < heap_.size());
    assert(j < heap_.size());
    std::swap(heap_[i], heap_[j]);
    ref_[heap_[i].id] = i;
    ref_[heap_[j].id] = j;
}

// 向上调整算法
void HeapTimer::SiftUp(size_t child) {
    assert(child < heap_.size());
    size_t parent = (child - 1) / 2;
    while (parent > 0) {
        if (heap_[parent] > heap_[child]) {
            SwapNode(child, parent);
            child = parent;
            parent = (child - 1) / 2;
        } else {
            break;
        }
    }
}

// 向下调整算法
// 返回：是否调整
bool HeapTimer::SiftDown(size_t parent, size_t n) {
    assert(parent < heap_.size());
    assert(n < heap_.size());
    size_t child = 2 * parent + 1; // 左孩子
    size_t pos = parent;
    while (child < n) {
        if (child + 1 < n && heap_[child + 1] < heap_[child])
            ++child; // 右孩子
        if (heap_[parent] > heap_[child]) {
            SwapNode(child, parent);
            parent = child;
            child = 2 * parent + 1;
        } else {
            break;
        }
    }
    return pos < parent;
}

void HeapTimer::Delete(size_t i) {
    assert(!heap_.empty() && i < heap_.size());
    size_t n = heap_.size() - 1;
    if (i < n - 1) { // 跳过n
        SwapNode(i, n);
        if (!SiftDown(i, n))
            SiftUp(i);
    }
    ref_.erase(heap_.back().id);
    heap_.pop_back();
}
```

# HeapTimer 测试

测试 HeapTimer 的`Add`，`Tick`，`GetNextTick`函数。

```C++
#include "../code/heap_timer/heap_timer.h"
#include "../code/log/log.h"
#include <cassert>

void CallbackFunction() {
    LOG_DEBUG("Callback function is called!")
}

// 测试 add 函数
void TestAdd() {
    HeapTimer timer;
    int id = 1;
    int timeout = 1000;  // 超时时间为 1000 毫秒

    timer.Add(id, timeout, CallbackFunction);
    assert(timer.Size() == 1);
}

// 测试 tick 函数
void TestTick() {
    HeapTimer timer;
    int id = 1;
    int timeout = 100;  // 超时时间为 100 毫秒

    timer.Add(id, timeout, CallbackFunction);

    // 等待一段时间，确保定时器超时
    std::this_thread::sleep_for(std::chrono::milliseconds(200));

    timer.Tick();
    assert(timer.Size() == 0);
}

// 测试 GetNextTick 函数
void TestGetNextTick() {
    HeapTimer timer;
    int id = 1;
    int timeout = 10000;  // 超时时间为 10000 毫秒

    timer.Add(id, timeout, CallbackFunction);
    int next_tick = timer.GetNextTick();
    assert(next_tick >= 0);
}

int main() {
    Log* logger = Log::GetInstance();
    logger->Init(0, "./logs/", ".log", 1024);

    TestAdd();
    TestTick();
    TestGetNextTick();
    return 0;
}
```

**CMakeList.txt**

```cmake
cmake_minimum_required(VERSION 3.10)
project(tests)

# 设置 C++ 标准和编译器选项
set(CMAKE_BUILD_TYPE Debug)
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra")


set(COMMON ../code/buffer/buffer.cc ../code/log/log.cc)
set(HEAP_TIMER ../code/heap_timer/heap_timer.cc)

add_executable(heap_timer_test heap_timer_test.cc ${COMMON} ${HEAP_TIMER})
```

