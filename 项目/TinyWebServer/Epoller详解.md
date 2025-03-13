# 为什么要用 epoll？

1. **高效的事件通知机制**：`epoll`采用事件驱动模型，仅通知应用程序那些真正发生事件的文件描述符（fd），避免了不必要的遍历。与`select`和`poll`相比，`epoll`不会随着连接数的增加而性能下降，因为它通过回调机制直接通知活跃的连接。

2. **支持大量并发连接**：`epoll`能够处理数十万甚至上百万的并发连接，适合高并发的Web服务器。它使用红黑树存储文件描述符，查找和操作的时间复杂度为O(1)，性能不会随连接数增加而显著下降。

3. **边缘触发（ET）和水平触发（LT）模式**

   - **边缘触发（ET）**：只在状态变化时通知一次，适合需要高效处理事件的场景，但需要确保一次性处理完所有数据。

   - **水平触发（LT）**：只要文件描述符处于就绪状态就会持续通知，编程更简单，但可能效率稍低。

4. **减少系统调用开销**：`epoll`通过`epoll_wait`一次性获取多个事件，减少了系统调用次数，降低了上下文切换的开销。

5. **内存效率高**：`epoll`使用内存映射技术（`mmap`)，避免了`select`和`poll`中频繁的内存拷贝，进一步提升了性能。

6. **适合高并发需求**：现代\\Web服务器需要处理大量并发连接，`epoll`具有高效事件管理和低延迟特性。


# epoll 介绍

`epoll` 是 Linux 内核提供的一种高效的 I/O 多路复用机制，用于监控多个文件描述符（file descriptors，简称 fd）的状态变化（如可读、可写、异常等）。它是 `select` 和 `poll` 的改进版本，特别适合处理大量并发连接的高性能网络服务器。

## 系统调用

1. `epoll_create`：用于创建一个 epoll 实例，返回一个文件描述符，该文件描述符用于后续的 epoll 操作。


```c
#include <sys/epoll.h>
int epoll_create(int size);
```

- `size`：在早期的内核版本中，该参数表示 epoll 实例能够容纳的最大文件描述符数量，但在现代内核中，此参数已被忽略，但必须大于 0。
- 返回值：成功时返回一个新的 epoll 实例的文件描述符，失败时返回 -1。

2. `epoll_ctl`：用于向 epoll 实例中添加、修改或删除对某个文件描述符的监听事件。

```c
#include <sys/epoll.h>
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
```

- `epfd`：`epoll_create`返回的 epoll 实例的文件描述符。

- `op`：操作类型，有以下三种取值：
  - `EPOLL_CTL_ADD`：将指定的文件描述符`fd`添加到 epoll 实例中，并监听`event`所指定的事件。
  - `EPOLL_CTL_MOD`：修改已经添加到 epoll 实例中的文件描述符`fd`的监听事件为`event`。
  - `EPOLL_CTL_DEL`：从 epoll 实例中删除指定的文件描述符`fd`，此时`event`参数可以为`NULL`。
  
- `fd`：要操作的文件描述符。

- `event`：指向`struct epoll_event`结构体的指针，用于指定要监听的事件类型。

```c
struct epoll_event {
    uint32_t     events;      /* Epoll events */
    epoll_data_t data;        /* User data variable */
};
typedef union epoll_data {
    void        *ptr;
    int          fd;
    uint32_t     u32;
    uint64_t     u64;
} epoll_data_t;
```

- `events`：可以是以下几种事件类型的组合：
  - `EPOLLIN`：文件描述符可读。
  - `EPOLLOUT`：文件描述符可写。
  - `EPOLLRDHUP`：对端关闭连接或半关闭写操作。
  - `EPOLLPRI`：有紧急数据可读。
  - `EPOLLERR`：文件描述符发生错误。
  - `EPOLLHUP`：文件描述符被挂起。
  - `EPOLLET`：设置为边缘触发模式（默认是水平触发模式）。
  - `EPOLLONESHOT`：只监听一次事件，当事件发生后，需要重新注册该文件描述符的监听事件。
  
- `data`：是一个联合体，用于存储用户数据，通常会将文件描述符`fd`存储在`data.fd`中，方便后续处理。

3.  `epoll_wait`：用于等待 epoll 实例中监听的文件描述符上的事件发生。

```c
#include <sys/epoll.h>
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
```

- `epfd`：`epoll_create`返回的 epoll 实例的文件描述符。

- `events`：指向`struct epoll_event`结构体数组的指针，用于存储发生事件的文件描述符信息。

- `maxevents`：`events`数组的最大元素个数，必须大于 0。

- `timeout`：超时时间，单位为毫秒。

  - `-1`：表示无限等待，直到有事件发生。
  - `0`：表示立即返回，即使没有事件发生。
  - 大于 0：表示等待指定的毫秒数，如果在这段时间内没有事件发生，则返回 0。

- 返回值：成功时返回发生事件的文件描述符数量，超时返回 0，失败返回 -1。

## 工作模式

**水平触发（Level-Triggered，LT）**

- 默认模式。
- 只要文件描述符处于就绪状态（例如缓冲区中有数据可读），`epoll_wait` 就会持续通知应用程序。
- 适合编程简单的场景，但可能效率较低，因为会重复通知。

**边缘触发（Edge-Triggered，ET）**

- 需要显式设置（通过 `EPOLLET` 标志）。
- 只在文件描述符状态发生变化时通知一次（例如从无数据变为有数据）。
- 适合高性能场景，但需要应用程序确保一次性处理完所有数据，否则可能会丢失事件。

# Epoller 代码

**对`epoll`封装实现`epoller`类。**

**epoller.h**

```cpp
#ifndef EPOLLER_H
#define EPOLLER_H

#include <unistd.h>
#include <sys/epoll.h>
#include <cassert>
#include <cstring>
#include <vector>

class Epoller {
public:
    explicit Epoller(int max_event = 1024);
    ~Epoller();

    bool AddFd(int fd, uint32_t events);
    bool ModFd(int fd, uint32_t events);
    bool DelFd(int fd);
    int Wait(int timeout_ms = -1);
    int GetEventsFd(size_t i) const;
    uint32_t GetEvents(size_t i) const;

private:
    int epoll_fd_;
    std::vector<struct epoll_event> events_;
};

#endif // EPOOLER_H
```

**epoller.cc**

```cpp
#include "epoller.h"

Epoller::Epoller(int max_event) 
    : epoll_fd_(epoll_create(512)), events_(max_event) {
    assert(epoll_fd_ >= 0 && events_.size() > 0); 
}

Epoller::~Epoller() {
    close(epoll_fd_);
}

bool Epoller::AddFd(int fd, uint32_t events) {
    if (fd < 0)
        return false;
    epoll_event ev;
    memset(&ev, 0, sizeof(epoll_event));
    ev.data.fd = fd;
    ev.events = events;
    return epoll_ctl(epoll_fd_, EPOLL_CTL_ADD, fd, &ev) == 0;
}

bool Epoller::ModFd(int fd, uint32_t events) {
    if (fd < 0)
        return false;
    epoll_event ev;
    memset(&ev, 0, sizeof(epoll_event));
    ev.data.fd = fd;
    ev.events = events;
    return epoll_ctl(epoll_fd_, EPOLL_CTL_MOD, fd, &ev) == 0;
}

bool Epoller::DelFd(int fd) {
    if (fd < 0)
        return false;
    return epoll_ctl(fd, EPOLL_CTL_DEL, fd, 0) == 0;
}

int Epoller::Wait(int timeout_ms) {
    return epoll_wait(epoll_fd_, &events_[0], static_cast<int>(events_.size()), timeout_ms);
}

int Epoller::GetEventsFd(size_t i) const {
    assert(i < events_.size());
    return events_[i].data.fd;
}

uint32_t Epoller::GetEvents(size_t i) const {
    assert(i < events_.size());
    return events_[i].events;
}
```

