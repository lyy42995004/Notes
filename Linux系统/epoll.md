# epoll是什么？

**epoll** 是从 Linux 2.6 起，Linux内核提供的一种高性能**I/O事件通知机制**，用于解决传统 select 和 poll 在处理大量并发连接时遍历、最大数量限制、频繁拷贝数据等问题。epoll 可以用来监听多个文件描述符（socket、管道、eventfd、timerfd 等）上的I/O事件。

**核心特点**：

- 高效的事件驱动模型

  - **O(1) 时间复杂度**：不管监控多少文件描述符（FD），epoll 的事件检测效率几乎不变，而 select/poll 是 O(n) 的。
  - **仅返回活跃的 FD**：不像 select/poll 需要遍历所有 FD，epoll 只返回有事件发生的 FD，减少无效遍历。

- 支持大并发连接

  - 单进程可轻松管理 **数十万甚至百万级** 的并发连接（如 Nginx、Redis 使用 epoll 实现高并发）。

  - 使用 **红黑树（RB-tree）存储 FD**，查找、插入、删除高效。

- 边缘触发（ET）和水平触发（LT）模式
  - **水平触发（LT，默认模式）**：只要 FD 可读/可写，epoll 会一直通知应用程序（类似 poll）。
  - **边缘触发（ET）**：仅在 FD 状态变化时通知一次（更高效，但需正确处理，否则可能丢失事件）。

# epoll 怎么用？

## epoll_create

```c
int epoll_create(int size);
```

**作用**：创建 epoll 实例，得到 epoll 文件描述符，用于后续对 epoll 的所有调用。

当不在需要时调用 close() 关闭，当所有引用 epoll 实例的文件描述符都关闭后，内核将销毁该实例并释放相关资源以供重用。

**返回值**：成功时，返回文件描述符；错误时，返回 -1，并设置 errno。

## epoll_ctl

```c
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
```

**作用**：添加/修改/删除要监听的 fd；通过对文件描述符 **epfd** 引用的 epoll 实例执行控制操作，对目标文件描述符 **fd** 执行 **op** 。

**event** 参数描述了链接到文件描述符 **fd** 的对象。

**op 参数**：

| op              | 功能                    | 说明                      |
| --------------- | ----------------------- | ------------------------- |
| `EPOLL_CTL_ADD` | 添加新 fd 到 epoll 实例 | 已存在返回 `EEXIST`       |
| `EPOLL_CTL_MOD` | 修改 fd 的监听事件      | 不存在返回 `ENOENT`       |
| `EPOLL_CTL_DEL` | 从 epoll 实例中移除 fd  | `event` 参数可以是 `NULL` |

## 事件

```c
// eventpoll.h
#define EPOLLIN     (__force __poll_t)0x00000001
#define EPOLLOUT    (__force __poll_t)0x00000004
#define EPOLLERR    (__force __poll_t)0x00000008
#define EPOLLHUP    (__force __poll_t)0x00000010
#define EPOLLRDHUP  (__force __poll_t)0x00002000
#define EPOLLEXCLUSIVE  ((__force __poll_t)(1U << 28))
#define EPOLLET     ((__force __poll_t)(1U << 31))
```

 *struct epoll_event* 定义如下：

```c
typedef union epoll_data {
    void        *ptr;
    int          fd;
    uint32_t     u32;
    uint64_t     u64;
} epoll_data_t;

struct epoll_event {
    uint32_t     events;      /* Epoll events */
    epoll_data_t data;        /* User data variable */
};
```

**epoll 的事件类型**，就是在 `epoll_event.events` 里设置的，epoll 会根据这些事件类型来判断是否要把 fd 加入活跃队列，最后由 `epoll_wait()` 返回。

| 宏定义         | 含义                                            |
| -------------- | ----------------------------------------------- |
| `EPOLLIN`      | 表示对应的文件描述符可以读（recv/read）         |
| `EPOLLOUT`     | 表示对应的文件描述符可以写（send/write）        |
| `EPOLLRDHUP`   | 表示对方关闭了写端或者半关闭（对 TCP 非常有用） |
| `EPOLLERR`     | 表示对应的文件描述符发生错误                    |
| `EPOLLHUP`     | 表示对应的文件描述符被挂断（对方断开连接）      |
| `EPOLLET`      | 边缘触发（Edge Trigger）                        |
| `EPOLLONESHOT` | 事件只触发一次，触发后自动从 epoll 中移除       |

**返回值**：成功时，返回文件描述符；错误时，返回 -1，并设置 errno。

## epoll_wait

```c
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
```

**作用**：等待*文件描述符* **epfd** 指向的 epoll 实例上的事件 。**events** 指向的内存区域将包含调用者可用的事件。epoll_wait () 最多返回 **maxevents** 个事件。 maxevents 参数必须大于零。

**timeout** 参数指定 epoll_wait() 阻塞的最小毫秒数。将*超时*指定为 -1 会导致 epoll_wait() 无限期阻塞，同时指定等于零的*超时*会导致 **epoll_wait** () 立即返回，即使没有可用事件。

**返回值**：成功时，返回已为就绪的 I/O 文件描述符数量；如果在请求的*超时*毫秒数内没有文件描述符就绪，则返回0；错误时，返回 -1，并设置 errno。

# 核心数据结构

## eventpoll

**eventpoll 就是 epoll 的控制中心**，红黑树管着监听 fd，rdllist 是触发事件队列，wq 是 epoll_wait 阻塞队列，poll_wait 给 file->poll 用，锁保护多核并发。事件触发靠回调，回调唤醒 epoll_wait，事件放到 rdllist。

```cpp
/*
 * This structure is stored inside the "private_data" member of the file
 * structure and represents the main data structure for the eventpoll
 * interface.
 */
struct eventpoll {
    /*
     * This mutex is used to ensure that files are not removed
     * while epoll is using them. This is held during the event
     * collection loop, the file cleanup path, the epoll file exit
     * code and the ctl operations.
     */
    struct mutex mtx;

    /* Wait queue used by sys_epoll_wait() */
    wait_queue_head_t wq;

    /* Wait queue used by file->poll() */
    wait_queue_head_t poll_wait;

    /* List of ready file descriptors */
    struct list_head rdllist;

    /* Lock which protects rdllist and ovflist */
    rwlock_t lock;

    /* RB tree root used to store monitored fd structs */
    struct rb_root_cached rbr;

    /*
     * This is a single linked list that chains all the "struct epitem" that
     * happened while transferring ready events to userspace w/out
     * holding ->lock.
     */
    struct epitem *ovflist;

    /* wakeup_source used when ep_scan_ready_list is running */
    struct wakeup_source *ws;

    /* The user that created the eventpoll descriptor */
    struct user_struct *user;

    struct file *file;

    /* used to optimize loop detection check */
    int visited;
    struct list_head visited_list_link;

#ifdef CONFIG_NET_RX_BUSY_POLL
    /* used to track busy poll napi_id */
    unsigned int napi_id;
#endif
};
```

|   成员    | 描述                                                         |
| :-------: | :----------------------------------------------------------- |
|    mtx    | 全局互斥锁，保证在**事件收集、关闭 fd、epoll_ctl 操作**时对 epoll 的一致性。 |
|    wq     | **epoll_wait 阻塞队列**，当没有事件时，调用 `epoll_wait` 的进程挂在这里。 |
| poll_wait | 给 `file->poll()` 用的等待队列，内核中 poll 实现底层回调时用到。 |
|  rdllist  | **就绪事件链表**，当 fd 触发了事件，内核会把对应 `epitem` 放到这里，供 `epoll_wait` 返回给用户。 |
|  ovflist  | 单链表，当 rdllist 被锁定遍历，向用户空间发送数据时，rdllist 不允许被修改，新触发的就绪 epitem 被 ovflist 串联起来，等待 rdllist 被处理完了，重新将 ovflist 数据写入 rdllist。 详看 ep_scan_ready_list 逻辑。 |
|   user    | 拥有这个 epoll 的用户，做内核权限检查、资源限制等用途。      |
|   lock    | 锁，保护 rdllist 和 ovflist 。                               |
|    rbr    | **红黑树根节点**，用来管理所有注册到 epoll 的 fd（`epitem`）。 |
|   file    | eventpoll 对应的文件结构，Linux 一切皆文件，用 vfs 管理数据。 |
|  napi_id  | 应用于中断缓解技术。                                         |

## epitem

**epitem** 是 epoll 里**管理单个 fd 事件状态**的核心单元，红黑树挂 rbn，就绪事件挂 rdllist，等待队列挂 pwqlist，拷贝用户关心事件在 `event` 里，容器指针 ep，fd 信息 ffd，控制能高效组织+高效唤醒。

```cpp
/*
 * Each file descriptor added to the eventpoll interface will
 * have an entry of this type linked to the "rbr" RB tree.
 * Avoid increasing the size of this struct, there can be many thousands
 * of these on a server and we do not want this to take another cache line.
 */
struct epitem {
    union {
        /* RB tree node links this structure to the eventpoll RB tree */
        struct rb_node rbn;
        /* Used to free the struct epitem */
        struct rcu_head rcu;
    };

    /* List header used to link this structure to the eventpoll ready list */
    struct list_head rdllink;

    /*
     * Works together "struct eventpoll"->ovflist in keeping the
     * single linked chain of items.
     */
    struct epitem *next;

    /* The file descriptor information this item refers to */
    struct epoll_filefd ffd;

    /* Number of active wait queue attached to poll operations */
    int nwait;

    /* List containing poll wait queues */
    struct list_head pwqlist;

    /* The "container" of this item */
    struct eventpoll *ep;

    /* List header used to link this item to the "struct file" items list */
    struct list_head fllink;

    /* wakeup_source used when EPOLLWAKEUP is set */
    struct wakeup_source __rcu *ws;

    /* The structure that describe the interested events and the source fd */
    struct epoll_event event;
};
```

|  成员   | 描述                                                         |
| :-----: | :----------------------------------------------------------- |
|   rbn   | 挂在 `eventpoll->rbr` 红黑树上。                             |
|   rcu   | 当需要释放 epitem 时用 RCU 延迟删除。                        |
| rdllink | 链接到 `eventpoll->rdllist`，表示**就绪事件**。当 fd 有事件触发，内核就把它放到 `rdllist` 里。 |
|  next   | 用来维护 `eventpoll->ovflist` 单向链表的 next 指针，避免拷贝事件到用户空间时遗漏新的事件。 |
|   ffd   | 记录节点对应的 fd 和 file 文件信息。                         |
|  nwait  | 等待队列个数。                                               |
| pwqlist | 等待事件回调队列。当数据进入网卡，底层中断执行 ep_poll_callback。 |
|   ep    | eventpoll 指针，epitem 关联 eventpoll。                      |
| fllink  | epoll 文件链表结点，与 epoll 文件链表进行关联 file.f_ep_links。参考 fs.h, struct file 结构。 |
|   ws    | EPOLLWAKEUP 模式下使用。                                     |
|  event  | 用户关注的事件。                                             |

# epoll 的网络服务器流程图

![image.png](https://s2.loli.net/2025/05/10/LzueGlhxgo1wyiU.png)

## 初始化阶段

- `socket()`：创建 `server_fd`
- `bind()`：绑定 IP 和端口
- `fcntl()`：设置非阻塞模式（O_NONBLOCK）
- `listen()`：监听端口，准备接收连接

## epoll 创建 & 注册

```cpp
epoll_fd = epoll_create()
epoll_ctl(epoll_fd, EPOLL_CTL_ADD, server_fd, EPOLLIN)
```

- `epoll_create()`：创建 epoll 实例，得到 `epoll_fd`
- `epoll_ctl(EPOLL_CTL_ADD)`：将 `server_fd` 加入 `epoll_fd`，监听 `EPOLLIN`（监听是否有新连接）

## 事件循环

```cpp
epoll_wait(epoll_fd, events, ...)
```

- `epoll_wait()`：阻塞等待事件，如果有事件触发（新连接 / 客户端读写事件），返回事件数组

**处理新连接（server_fd 可读）**

```c
if (server_fd == events[i].data.fd) {
    client_fd = accept(server_fd);  // 非阻塞 accept
    fcntl(client_fd, O_NONBLOCK);  // 设置非阻塞
    epoll_ctl(EPOLL_CTL_ADD, client_fd, EPOLLIN);  // 监控 client_fd
}
```

- `accept()` 接收新连接，得到 `client_fd`
- 设置 `client_fd` 非阻塞
- `epoll_ctl(EPOLL_CTL_ADD)`，监听 `client_fd` 的 `EPOLLIN`

**处理客户端数据（client_fd 可读）**

```cpp
if (client_fd 有 EPOLLIN 事件) {
    ret = read(client_fd, buf, size);  // 非阻塞 read
    if (ret > 0) {
        // 正常读取数据，处理业务逻辑
    } else if (ret == 0 || (ret == -1 && errno != EAGAIN)) {
        // 客户端关闭连接或出错
        close(client_fd);
        epoll_ctl(EPOLL_CTL_DEL, client_fd);  // 移除监控
    }
    // EAGAIN 表示数据未就绪，继续等待
}
```

- 如果 `EPOLLIN`：说明 `client_fd` 有数据可读，调用 `read()`
  - `ret > 0`：正常读取数据，进行业务处理。
  - `ret == 0`：客户端关闭连接，关闭 `client_fd` 并从 epoll 移除。
  - `ret == -1 && errno == EAGAIN`：数据未就绪（非阻塞模式），继续等待。

**处理客户端写入（EPOLLOUT 事件）**

```cpp
if (需要向 client_fd 写入数据) {
    epoll_ctl(EPOLL_CTL_MOD, client_fd, EPOLLOUT);  // 关注可写事件
}

if (client_fd 有 EPOLLOUT 事件) {
    ret = write(client_fd, buf, size);  // 非阻塞 write
    if (ret == -1 && errno == EAGAIN) {
        // 缓冲区满，稍后重试
    } else if (ret >= 0) {
        // 写入成功，恢复监控 EPOLLIN
        epoll_ctl(EPOLL_CTL_MOD, client_fd, EPOLLIN);
    } else {
        // 写入失败，关闭连接
        close(client_fd);
        epoll_ctl(EPOLL_CTL_DEL, client_fd);
    }
}
```

- 如果 `EPOLLOUT`：说明 `client_fd` 可写，调用 `write()`
  - `ret >= 0`：写入成功，恢复监控 `EPOLLIN`。
  - `ret == -1 && errno == EAGAIN`：缓冲区满，稍后重试。

## 关闭连接

- `close()` 关闭 `server_fd` 和所有 `client_fd`，结束服务

# TCP + epoll 流程图

![image.png](https://s2.loli.net/2025/05/10/3gBCqtFOuU9epR7.png)

这是一张 Linux 5.0.1 内核下基于 epoll 的网络编程及 TCP 连接建立相关的流程图，涵盖了从客户端连接请求到服务器端处理的多个环节，下面我简单介绍一下流程：

## ① `epoll_create`

- 创建 `eventpoll` 结构体，初始化红黑树 `rbr` 和就绪队列 `rdlist`
- 返回 `epoll_fd`

## ② `epoll_ctl`

- `EPOLL_CTL_ADD`  把目标 `fd` 包装成 `epitem` 节点
- 挂到 `eventpoll->rbr` 红黑树里
- 建立 `sock->sk_wq` 的回调 `ep_poll_callback`

## ③ 监听事件队列

- 将 `epitem` 挂到 `socket->wq->wait_queue_head` 上
- 事件触发时由回调唤醒挂在 `wait_queue` 上的 `epitem`

## ④ `epoll_wait`

- 把当前线程挂到 `eventpoll->wq` 上，进入 `TASK_INTERRUPTIBLE`
- 调用 `schedule()` 让出 CPU 等待事件

## ⑤ 网络事件到达（TCP 三次握手）

- 驱动收到网卡中断，触发 `tcp_v4_rcv`
- 调用 `sock_def_wakeup` 唤醒 `wait_queue` 上的 `epitem`

## ⑥ `ep_poll_callback`

- 将就绪的 `epitem` 节点挂到 `eventpoll->rdlist`
- 设置 `wake_up_flag` 唤醒 `epoll_wait`

## ⑦ 唤醒 `epoll_wait`

- `wake_up_locked` 唤醒 epoll 阻塞的线程

## ⑧ `ep_send_events`

- 从 `eventpoll->rdlist` 中取出就绪事件
- 返回到用户空间的 `events` 数组

## ⑨ 用户态 accept 连接

- 获取 client_fd，再次 `epoll_ctl` 加入监听

# epoll 的应用场景

**高并发网络服务器**

例如：Web 服务器（如 Nginx、**C++ WebServer** 项目），Chat Server（即时通讯服务器），游戏服务器，HTTP Proxy。

**为什么用 epoll**：支持**成千上万的并发连接**，不像 `select/poll` 那样有 FD 数量上限（`select` 1024），事件通知机制效率高（`epoll_wait` 只返回活跃 FD）。

-----

**高并发客户端程序**

例如：爬虫系统（需要同时维护大量 HTTP/TCP 连接），高并发消息队列消费者，WebSocket 客户端集群。

**为什么用 epoll**：维护大量 socket，避免频繁阻塞或线程爆炸，支持非阻塞 IO，高效回调处理响应。

----

**长连接服务场景**

例如：IM 服务，实时推送服务，股票/期货交易系统行情推送，实时视频/音频流服务。

为什么用 epoll：长连接数量庞大，短时间内事件数量较少，`epoll` 的**事件回调机制**极其适合，配合 ET 模式，减少 epoll_wait 调用次数，提高效率。

----

**多 IO 设备场景**

例如：高性能日志系统，文件同步/备份系统（同时监听大量文件 IO 事件），异步文件下载器/上传器。

**为什么用 epoll**：能同时监听多设备 IO 事件，不必阻塞等待单个 IO 完成。

> 参考资料：
>
> [epoll(4) - Linux man page](https://linux.die.net/man/4/epoll)（Linux man 手册）
>
> [[内核源码] epoll 实现原理](https://wenfh2020.com/2020/04/23/epoll-code/)