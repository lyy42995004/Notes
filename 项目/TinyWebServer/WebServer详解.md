# WebServer 运行流程

```cpp
#include "server/web_server.h"

int main() {
    WebServer server(8080, 3, 60000, 3306,          // 端口 ET模式 timeoutMs 
                    "User", "123456", "web_server", // Mysql配置
                    12, 8, true, 1, 1024);          // 连接池数量 线程池数量 日志开关 日志等级 日志异步队列容量
    server.start();
    return 0;

```

## 1. 服务器初始化

在创建 `WebServer` 对象时，构造函数会进行一系列的初始化操作：

- **日志系统初始化**：若开启日志功能，会初始化日志系统，设置日志级别、存储路径和队列大小等参数。
- **资源目录设置**：获取当前工作目录，并拼接上 `/resources` 作为资源文件的根目录，同时将其赋值给 `HttpConnect::src_dir`。
- **数据库连接池初始化**：调用 `SqlConnectPool::GetInstance()->Init` 初始化数据库连接池，设置数据库的地址、端口、用户名、密码、数据库名和连接池大小。
- **事件模式设置**：调用 `InitEventMode` 函数，依据传入的 `trigger_mode` 参数设置监听套接字和客户端连接套接字的事件模式，可选择水平触发（LT）或边沿触发（ET）。
- **监听套接字初始化**：调用 `InitSocker` 函数创建监听套接字，设置地址重用选项，绑定地址和端口，开始监听客户端连接请求。将监听套接字添加到 `epoll` 实例中，并设置为非阻塞模式。

## 2. 服务器启动

调用 `start` 函数启动服务器，进入主循环：

- **获取下一个定时器超时时间**：若设置了超时时间 `timeout_ms_`，调用 `timer_->GetNextTick()` 获取下一个定时器的超时时间，用于 `epoll` 的 `Wait` 函数。
- **等待事件发生**：调用 `epoller_->Wait(timeout_ms)` 等待事件发生，此函数会阻塞，直到有事件触发或超时。
- 处理事件：当有事件触发时，遍历所有事件，依据事件类型调用相应的处理函数：
  - **新连接事件**：若事件的文件描述符为监听套接字 `listen_fd_`，调用 `DealListen` 函数处理新的客户端连接请求。
  - **连接关闭或错误事件**：若事件包含 `EPOLLRDHUP`、`EPOLLHUP` 或 `EPOLLERR` 标志，调用 `CloseConn` 函数关闭客户端连接。
  - **读事件**：若事件包含 `EPOLLIN` 标志，调用 `DealRead` 函数处理读事件。
  - **写事件**：若事件包含 `EPOLLOUT` 标志，调用 `DealWrite` 函数处理写事件。

## 3. 处理新连接

当有新的客户端连接请求到达时，`DealListen` 函数会被调用：

- **接受连接**：调用 `accept` 函数接受新的客户端连接，获取客户端的套接字文件描述符和地址信息。
- **检查连接数**：检查当前连接数是否达到上限 `MAX_FD`，若达到上限，调用 `SendError` 函数向客户端发送错误信息并关闭连接。
- **添加客户端**：调用 `AddClient` 函数处理新连接，初始化客户端的 `HttpConnect` 对象，将其添加到 `users_` 映射中。若设置了超时时间，将该客户端的超时处理函数添加到定时器中。将客户端套接字添加到 `epoll` 实例中，并设置为非阻塞模式。

## 4. 处理读事件

当客户端有数据发送过来时，会触发读事件，`DealRead` 函数会被调用：

- **延长超时时间**：调用 `ExtendTime` 函数延长客户端连接的超时时间。
- **添加读任务**：将读处理任务添加到线程池中，调用 `thread_pool_->AddTask(std::bind(&WebServer::OnRead, this, client))`。
- **读取数据**：在线程池中执行 `OnRead` 函数，从客户端读取数据。若读取失败且错误码不是 `EAGAIN`，调用 `CloseConn` 函数关闭连接。
- **处理请求**：调用 `OnProcess` 函数处理客户端请求，根据处理结果修改客户端套接字的事件类型，若处理成功则监听写事件，否则监听读事件。

## 5. 处理写事件

当服务器准备好向客户端发送响应数据时，会触发写事件，`DealWrite` 函数会被调用：

- **延长超时时间**：调用 `ExtendTime` 函数延长客户端连接的超时时间。
- **添加写任务**：将写处理任务添加到线程池中，调用 `thread_pool_->AddTask(std::bind(&WebServer::OnWrite, this, client))`。
- **发送数据**：在线程池中执行 `OnWrite` 函数，向客户端发送响应数据。若数据发送完毕且客户端希望保持连接，将客户端套接字的事件类型修改为监听读事件。若发送失败且错误码是 `EAGAIN`，继续监听写事件；否则关闭连接。

## 6. 连接超时处理

若设置了超时时间，`timer_` 会管理连接的超时。当某个客户端连接超时，定时器会触发相应的回调函数，即 `CloseConn` 函数，关闭该客户端连接。

## 7. 服务器关闭

当 `is_close_` 标志被设置为 `true` 时，主循环会退出，服务器停止运行。在析构函数中，会关闭监听套接字，释放资源目录的内存，关闭数据库连接池。

# WebServer 类

**成员变量**

```cpp
private:
    static const int MAX_FD = 65536;

    int port_;
    bool open_linger_;
    int timeout_ms_;
    bool is_close_;
    int listen_fd_;
    char *src_dir_;

    uint32_t listen_event_;
    uint32_t conn_event_;

    std::unique_ptr<HeapTimer> timer_;
    std::unique_ptr<ThreadPool> thread_pool_;
    std::unique_ptr<Epoller> epoller_;
    std::unordered_map<int, HttpConnect> users_;
```

# 构造函数

- 初始化服务器的各项参数，包括端口号、超时时间、日志系统、资源目录、数据库连接池等。
- 调用 `InitEventMode` 函数设置事件模式。
- 调用 `InitSocker` 函数初始化监听套接字，如果失败则将 `is_close_` 设为 `true`。

```cpp
WebServer::WebServer(int port, int trigger_mode, int timeout_ms,
                     int sql_port, const char* sql_user, const char* sql_pwd,
                     const char* db_name, int conn_pool_num, int thread_num, 
                     bool open_log, int log_level, int log_que_size) 
                     : port_(port), timeout_ms_(timeout_ms), is_close_(false), 
                       timer_(new HeapTimer()), thread_pool_(new ThreadPool(thread_num)), 
                       epoller_(new Epoller()) {
    if (open_log) {
        Log::GetInstance()->Init(log_level, "./log/", ".log", log_que_size);
        if (is_close_) { LOG_ERROR("============== Server Init Error! =============="); } 
        else { 
            LOG_INFO("============== Server Init! ==============");
            LOG_INFO("Listen Mode: %s, OpenConn Mode: %s",
                    (listen_event_ & EPOLLET) ? "ET" : "LT",
                    (conn_event_ & EPOLLET) ? "ET" : "LT");
            LOG_INFO("LogSys level: %d", log_level);
            LOG_INFO("src_dir: %s", HttpConnect::src_dir);
            LOG_INFO("SqlConnectPool num: %d, ThreadPool num: %d", conn_pool_num, thread_num);
        }
    }

    HttpConnect::use_count = 0;
    src_dir_ = getcwd(nullptr, 256);
    assert(src_dir_);
    strcat(src_dir_, "/resources");
    HttpConnect::src_dir = src_dir_;

    SqlConnectPool::GetInstance()->Init("localhost", sql_port, sql_user, sql_pwd, db_name, conn_pool_num);
    InitEventMode(trigger_mode);
    if (!InitSocker()) { is_close_ = true; }
}
```

# 析构函数

- 关闭监听套接字，释放资源目录的内存，关闭数据库连接池。

```cpp
WebServer::~WebServer() {
    close(listen_fd_);
    is_close_ = true;
    free(src_dir_);
    SqlConnectPool::GetInstance()->ClosePool();
}
```

# 实现 InitSocker() 函数

- 创建监听套接字，设置 `SO_REUSEADDR` 选项以允许地址重用。
- 绑定地址和端口，开始监听。
- 将监听套接字添加到 `epoll` 实例中，并设置为非阻塞模式。

```cpp
bool WebServer::InitSocker() {
    int ret;
    struct sockaddr_in addr;
    addr.sin_family = AF_INET;
    addr.sin_addr.s_addr = htonl(INADDR_ANY);
    addr.sin_port = htons(port_);

    listen_fd_ = socket(AF_INET, SOCK_STREAM, 0);
    if (listen_fd_ < 0) {
        LOG_ERROR("Create socket error!");
        return false;
    }

    int optval = 1;
    ret = setsockopt(listen_fd_, SOL_SOCKET, SO_REUSEADDR, (const void*)&optval, sizeof(int));
    if (ret == -1) {
        LOG_ERROR("Set socket setsockopt error!");
        close(listen_fd_);
        return false;   
    }

    ret = bind(listen_fd_, (struct sockaddr*)&addr, sizeof(addr));
    if (ret < 0) {
        LOG_ERROR("Bind port: %d error!", port_);
        close(listen_fd_);
        return false;   
    }

    ret = listen(listen_fd_, 8);
    if (ret < 0) {
        LOG_ERROR("Listen port: %d error!", port_);
        close(listen_fd_);
        return false;   
    }
    ret = epoller_->AddFd(listen_fd_, listen_event_ | EPOLLIN);
    if (ret == 0) {
        LOG_ERROR("Add listen error!");
        close(listen_fd_);
        return false;
    }
    SetFdNonBlock(listen_fd_);
    LOG_INFO("Server port:%d", port_);
    return true;
}
```

# 实现 SetFdNonBlock() 函数

- 将指定的文件描述符设置为非阻塞模式。

```cpp
int WebServer::SetFdNonBlock(int fd) {
    assert(fd > 0);
    return fcntl(fd, F_SETFL, fcntl(fd, F_GETFD, 0) | O_NONBLOCK);
}
```

# 实现 InitEventMode() 函数

- 根据传入的 `trigger_mode` 设置监听套接字和客户端连接套接字的事件模式（水平触发或边沿触发）。
- `EPOLLRDHUP` 用于检测连接关闭，`EPOLLONESHOT` 确保一个事件只由一个线程处理。

```cpp
void WebServer::InitEventMode(int trigger_mode) {
    listen_event_ = EPOLLRDHUP; // 检测socket关闭
    conn_event_ = EPOLLONESHOT | EPOLLRDHUP; // EPOLLONESHOT由一个线程处理
    switch (trigger_mode) {
    case 0:
        break;
    case 1:
        conn_event_ |= EPOLLET;
        break;
    case 2:
        listen_event_ |= EPOLLET;
        break;
    case 3:
        listen_event_ |= EPOLLET;
        conn_event_ |= EPOLLET;
        break;
    default: 
        listen_event_ |= EPOLLET;
        conn_event_ |= EPOLLET;
        break;
    }
    HttpConnect::is_ET = (conn_event_ & EPOLLET);
}
```

# 实现 start() 函数

- 服务器的主循环，不断调用 `epoller_->Wait` 等待事件发生。
- 根据事件类型调用相应的处理函数，如 `DealListen` 处理新连接，`CloseConn` 关闭连接，`DealRead` 处理读事件，`DealWrite` 处理写事件。

```cpp
void WebServer::start() {
    int timeout_ms = -1; // 无事件阻塞
    if (!is_close_) { LOG_INFO("============== Server Start! =============="); }
    while (!is_close_) {
        if (timeout_ms_ > 0)
            timeout_ms = timer_->GetNextTick();
        int event_cnt = epoller_->Wait(timeout_ms);
        for (int i = 0; i < event_cnt; ++i) {
            int fd = epoller_->GetEventsFd(i);
            uint32_t events = epoller_->GetEvents(i);
            if (fd == listen_fd_) {
                DealListen();
            } else if (events & (EPOLLRDHUP | EPOLLHUP | EPOLLERR)) {
                assert(users_.count(fd) > 0);
                CloseConn(&users_[fd]);
            } else if (events & EPOLLIN) {
                assert(users_.count(fd) > 0);
                DealRead(&users_[fd]);
            } else if (events & EPOLLOUT) {
                assert(users_.count(fd) > 0);
                DealWrite(&users_[fd]);
            } else {
                LOG_ERROR("Unexpected event!");
            }
        }
    }
}
```

# 实现 AddClient() 函数

- 初始化新客户端的 `HttpConnect` 对象。
- 如果设置了超时时间，将该客户端的超时处理函数添加到定时器中。
- 将客户端套接字添加到 `epoll` 实例中，并设置为非阻塞模式。

```cpp
void WebServer::AddClient(int fd, sockaddr_in addr) {
    assert(fd > 0);
    users_[fd].Init(fd, addr);
    if (timeout_ms_ > 0) {
        timer_->Add(fd, timeout_ms_, std::bind(&WebServer::CloseConn, this, &users_[fd]));
    }
    epoller_->AddFd(fd, EPOLLIN | conn_event_);
    SetFdNonBlock(fd);
    LOG_INFO("Client[%d] in!", users_[fd].GetFd());
}
```

# 实现DealListen() 函数

- 处理新的客户端连接请求。
- 如果连接数达到上限，向客户端发送错误信息并关闭连接。
- 调用 `AddClient` 函数处理新连接。

```cpp
void WebServer::DealListen() {
    struct sockaddr_in addr;
    socklen_t len = sizeof(addr);
    do {
        int fd = accept(listen_fd_, (sockaddr*)&addr, &len);
        if (fd <= 0) { return; }
        else if (HttpConnect::use_count >= MAX_FD) {
            SendError(fd, "Server busy!");
            LOG_WARN("Clients is full!");
            return;
        }
        AddClient(fd, addr);
    } while (listen_event_ & EPOLLET);
}
```

# 实现 DealRead() 函数

- 延长客户端连接的超时时间。
- 将读处理任务添加到线程池中。

```cpp
void WebServer::DealRead(HttpConnect* client) {
    assert(client);
    ExtendTime(client);
    thread_pool_->AddTask(std::bind(&WebServer::OnRead, this, client));
}
```

# 实现  DealWrite() 函数

- 延长客户端连接的超时时间。
- 将写处理任务添加到线程池中。

```cpp
void WebServer::DealWrite(HttpConnect* client) {
    assert(client);
    ExtendTime(client);
    thread_pool_->AddTask(std::bind(&WebServer::OnWrite, this, client));
}
```

# 实现 OnRead() 函数

- 从客户端读取数据。
- 如果读取失败且错误码不是 `EAGAIN`，则关闭连接。
- 调用 `OnProcess` 函数处理请求。

```cpp
void WebServer::OnRead(HttpConnect* client) {
    assert(client);
    int ret = -1;
    int read_errno = 0;
    ret = client->Read(&read_errno);
    if (ret <= 0 && read_errno != EAGAIN) {
        CloseConn(client);
        return;
    }
    OnProcess(client);
}
```

# 实现 OnProcess() 函数

- 处理客户端请求。
- 如果处理成功，将客户端套接字的事件类型修改为监听写事件；否则监听读事件

```cpp
void WebServer::OnProcess(HttpConnect* client) {
    if (client->Process()) {
        epoller_->ModFd(client->GetFd(), conn_event_ | EPOLLOUT); // 监听写
    } else {
        epoller_->ModFd(client->GetFd(), conn_event_ | EPOLLIN); // 监听读
    }
}
```

# 实现 OnWrite() 函数

- 向客户端发送响应数据。
- 如果数据发送完毕且客户端希望保持连接，将客户端套接字的事件类型修改为监听读事件。
- 如果发送失败且错误码是 `EAGAIN`，继续监听写事件；否则关闭连接。

```cpp
void WebServer::OnWrite(HttpConnect* client) {
    assert(client);
    int ret = 0;
    int write_errno = 0;
    ret = client->Write(&write_errno);
    if (client->ToWriteBytes() == 0) {
        if (client->IsKeepAlive()) {
            epoller_->ModFd(client->GetFd(), conn_event_ | EPOLLIN); // 监听读
        }
    } else if (ret < 0) {
        if (write_errno == EAGAIN) {
            epoller_->ModFd(client->GetFd(), conn_event_ | EPOLLOUT); // 监听写
            return;
        }
    }
    CloseConn(client);
}
```

# 实现 ExtendTime() 函数

- 延长客户端连接的超时时间。

```cpp
void WebServer::ExtendTime(HttpConnect* client) {
    assert(client);
    if (timeout_ms_ > 0) {
        timer_->Adjust(client->GetFd(), timeout_ms_);
    }
}
```

# web_server 代码

**web_server.h**

```cpp
#ifndef WEB_SERVER_H
#define WEB_SERVER_H

#include <memory>
#include <unordered_map>
#include <functional>

#include "../log/log.h"
#include "../pool/threadpool.h"
#include "../pool/sql_connect_pool.h"
#include "../http/http_connect.h"
#include "../heap_timer/heap_timer.h"
#include "epoller.h"

class WebServer
{
public:
    WebServer(int port, int trigger_mode, int timeout_ms,
              int sql_port, const char *sql_user, const char *sql_pwd,
              const char *db_name, int conn_pool_num, int thread_num,
              bool open_log, int log_level, int log_que_size);
    ~WebServer();
    void start();

private:
    static int SetFdNonBlock(int fd);

    bool InitSocker();
    void InitEventMode(int trigger_mode);

    void DealListen();
    void DealRead(HttpConnect *client);
    void DealWrite(HttpConnect *client);

    void AddClient(int fd, sockaddr_in addr);
    void SendError(int fd, const char *info);
    void ExtendTime(HttpConnect *client);
    void CloseConn(HttpConnect *client);

    void OnRead(HttpConnect *client);
    void OnProcess(HttpConnect *client);
    void OnWrite(HttpConnect *client);

    static const int MAX_FD = 65536;

    int port_;
    bool open_linger_;
    int timeout_ms_;
    bool is_close_;
    int listen_fd_;
    char *src_dir_;

    uint32_t listen_event_;
    uint32_t conn_event_;

    std::unique_ptr<HeapTimer> timer_;
    std::unique_ptr<ThreadPool> thread_pool_;
    std::unique_ptr<Epoller> epoller_;
    std::unordered_map<int, HttpConnect> users_;
};

#endif // WEB_SERVER_H
```

**web_server.cc**

```cpp
#include "web_server.h"

WebServer::WebServer(int port, int trigger_mode, int timeout_ms,
                     int sql_port, const char* sql_user, const char* sql_pwd,
                     const char* db_name, int conn_pool_num, int thread_num, 
                     bool open_log, int log_level, int log_que_size) 
                     : port_(port), timeout_ms_(timeout_ms), is_close_(false), 
                       timer_(new HeapTimer()), thread_pool_(new ThreadPool(thread_num)), 
                       epoller_(new Epoller()) {
    if (open_log) {
        Log::GetInstance()->Init(log_level, "./log/", ".log", log_que_size);
        if (is_close_) { LOG_ERROR("============== Server Init Error! =============="); } 
        else { 
            LOG_INFO("============== Server Init! ==============");
            LOG_INFO("Listen Mode: %s, OpenConn Mode: %s",
                    (listen_event_ & EPOLLET) ? "ET" : "LT",
                    (conn_event_ & EPOLLET) ? "ET" : "LT");
            LOG_INFO("LogSys level: %d", log_level);
            LOG_INFO("src_dir: %s", HttpConnect::src_dir);
            LOG_INFO("SqlConnectPool num: %d, ThreadPool num: %d", conn_pool_num, thread_num);
        }
    }

    HttpConnect::use_count = 0;
    src_dir_ = getcwd(nullptr, 256);
    assert(src_dir_);
    strcat(src_dir_, "/resources");
    HttpConnect::src_dir = src_dir_;

    SqlConnectPool::GetInstance()->Init("localhost", sql_port, sql_user, sql_pwd, db_name, conn_pool_num);
    InitEventMode(trigger_mode);
    if (!InitSocker()) { is_close_ = true; }
}

WebServer::~WebServer() {
    close(listen_fd_);
    is_close_ = true;
    free(src_dir_);
    SqlConnectPool::GetInstance()->ClosePool();
}

bool WebServer::InitSocker() {
    int ret;
    struct sockaddr_in addr;
    addr.sin_family = AF_INET;
    addr.sin_addr.s_addr = htonl(INADDR_ANY);
    addr.sin_port = htons(port_);

    listen_fd_ = socket(AF_INET, SOCK_STREAM, 0);
    if (listen_fd_ < 0) {
        LOG_ERROR("Create socket error!");
        return false;
    }

    int optval = 1;
    ret = setsockopt(listen_fd_, SOL_SOCKET, SO_REUSEADDR, (const void*)&optval, sizeof(int));
    if (ret == -1) {
        LOG_ERROR("Set socket setsockopt error!");
        close(listen_fd_);
        return false;   
    }

    ret = bind(listen_fd_, (struct sockaddr*)&addr, sizeof(addr));
    if (ret < 0) {
        LOG_ERROR("Bind port: %d error!", port_);
        close(listen_fd_);
        return false;   
    }

    ret = listen(listen_fd_, 8);
    if (ret < 0) {
        LOG_ERROR("Listen port: %d error!", port_);
        close(listen_fd_);
        return false;   
    }
    ret = epoller_->AddFd(listen_fd_, listen_event_ | EPOLLIN);
    if (ret == 0) {
        LOG_ERROR("Add listen error!");
        close(listen_fd_);
        return false;
    }
    SetFdNonBlock(listen_fd_);
    LOG_INFO("Server port:%d", port_);
    return true;
}

int WebServer::SetFdNonBlock(int fd) {
    assert(fd > 0);
    return fcntl(fd, F_SETFL, fcntl(fd, F_GETFD, 0) | O_NONBLOCK);
}

void WebServer::InitEventMode(int trigger_mode) {
    listen_event_ = EPOLLRDHUP; // 检测socket关闭
    conn_event_ = EPOLLONESHOT | EPOLLRDHUP; // EPOLLONESHOT由一个线程处理
    switch (trigger_mode) {
    case 0:
        break;
    case 1:
        conn_event_ |= EPOLLET;
        break;
    case 2:
        listen_event_ |= EPOLLET;
        break;
    case 3:
        listen_event_ |= EPOLLET;
        conn_event_ |= EPOLLET;
        break;
    default: 
        listen_event_ |= EPOLLET;
        conn_event_ |= EPOLLET;
        break;
    }
    HttpConnect::is_ET = (conn_event_ & EPOLLET);
}

void WebServer::start() {
    int timeout_ms = -1; // 无事件阻塞
    if (!is_close_) { LOG_INFO("============== Server Start! =============="); }
    while (!is_close_) {
        if (timeout_ms_ > 0)
            timeout_ms = timer_->GetNextTick();
        int event_cnt = epoller_->Wait(timeout_ms);
        for (int i = 0; i < event_cnt; ++i) {
            int fd = epoller_->GetEventsFd(i);
            uint32_t events = epoller_->GetEvents(i);
            if (fd == listen_fd_) {
                DealListen();
            } else if (events & (EPOLLRDHUP | EPOLLHUP | EPOLLERR)) {
                assert(users_.count(fd) > 0);
                CloseConn(&users_[fd]);
            } else if (events & EPOLLIN) {
                assert(users_.count(fd) > 0);
                DealRead(&users_[fd]);
            } else if (events & EPOLLOUT) {
                assert(users_.count(fd) > 0);
                DealWrite(&users_[fd]);
            } else {
                LOG_ERROR("Unexpected event!");
            }
        }
    }
}

void WebServer::AddClient(int fd, sockaddr_in addr) {
    assert(fd > 0);
    users_[fd].Init(fd, addr);
    if (timeout_ms_ > 0) {
        timer_->Add(fd, timeout_ms_, std::bind(&WebServer::CloseConn, this, &users_[fd]));
    }
    epoller_->AddFd(fd, EPOLLIN | conn_event_);
    SetFdNonBlock(fd);
    LOG_INFO("Client[%d] in!", users_[fd].GetFd());
}

void WebServer::DealListen() {
    struct sockaddr_in addr;
    socklen_t len = sizeof(addr);
    do {
        int fd = accept(listen_fd_, (sockaddr*)&addr, &len);
        if (fd <= 0) { return; }
        else if (HttpConnect::use_count >= MAX_FD) {
            SendError(fd, "Server busy!");
            LOG_WARN("Clients is full!");
            return;
        }
        AddClient(fd, addr);
    } while (listen_event_ & EPOLLET);
}

void WebServer::DealRead(HttpConnect* client) {
    assert(client);
    ExtendTime(client);
    thread_pool_->AddTask(std::bind(&WebServer::OnRead, this, client));
}

void WebServer::DealWrite(HttpConnect* client) {
    assert(client);
    ExtendTime(client);
    thread_pool_->AddTask(std::bind(&WebServer::OnWrite, this, client));
}

void WebServer::OnRead(HttpConnect* client) {
    assert(client);
    int ret = -1;
    int read_errno = 0;
    ret = client->Read(&read_errno);
    if (ret <= 0 && read_errno != EAGAIN) {
        CloseConn(client);
        return;
    }
    OnProcess(client);
}

void WebServer::OnProcess(HttpConnect* client) {
    if (client->Process()) {
        epoller_->ModFd(client->GetFd(), conn_event_ | EPOLLOUT); // 监听写
    } else {
        epoller_->ModFd(client->GetFd(), conn_event_ | EPOLLIN); // 监听读
    }
}

void WebServer::OnWrite(HttpConnect* client) {
    assert(client);
    int ret = 0;
    int write_errno = 0;
    ret = client->Write(&write_errno);
    if (client->ToWriteBytes() == 0) {
        if (client->IsKeepAlive()) {
            epoller_->ModFd(client->GetFd(), conn_event_ | EPOLLIN); // 监听读
        }
    } else if (ret < 0) {
        if (write_errno == EAGAIN) {
            epoller_->ModFd(client->GetFd(), conn_event_ | EPOLLOUT); // 监听写
            return;
        }
    }
    CloseConn(client);
}

void WebServer::CloseConn(HttpConnect* client) {
    assert(client);
    LOG_INFO("Client[%d] quit!", client->GetFd());
    epoller_->DelFd(client->GetFd());
    client->Close();
}

void WebServer::SendError(int fd, const char* info) {
    assert(fd > 0);
    int ret = send(fd, info, strlen(info), 0);
    if (ret < 0) {
        LOG_WARN("Send error to client[%d] error!", fd);
    }
    close(fd);
}

void WebServer::ExtendTime(HttpConnect* client) {
    assert(client);
    if (timeout_ms_ > 0) {
        timer_->Adjust(client->GetFd(), timeout_ms_);
    }
}
```

