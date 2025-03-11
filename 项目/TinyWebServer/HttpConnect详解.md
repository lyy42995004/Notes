# HttpConnect 类

`HttpConnect` 类在 Web 服务器中充当了客户端连接的管理和处理单元。它负责处理客户端与服务器之间的网络连接，包括数据的读取、解析请求、生成响应以及数据的写入等操作，是连接客户端和服务器业务逻辑的桥梁。

```C++
class HttpConnect {
public:
    static bool is_ET;
    static const char* src_dir;
    static std::atomic<int> use_count;

    HttpConnect();
    ~HttpConnect();

    void Init(int socket_fd, const sockaddr_in& addr);
    void Close();

    ssize_t Read(int* save_errno);
    ssize_t Write(int* save_errno);
    bool Process();

    // 写的总长度
    int ToWriteBytes() const { return iov_[0].iov_len + iov_[1].iov_len; }
    bool IsKeepAlive() const { return request_.IsKeepAlive(); }

    int GetFd() const { return fd_; }
    int GetPort() const { return addr_.sin_port; }
    const char* GetIP() const { return inet_ntoa(addr_.sin_addr); }
    sockaddr_in GetAddr() const { return addr_; }

private:
    int fd_;
    bool is_close_;
    struct sockaddr_in addr_;

    int iov_cnt_;
    struct iovec iov_[2];

    Buffer read_buff_;
    Buffer write_buff_;    
    
    HttpRequest request_;
    HttpResponse response_;
};
```

- `fd_`：客户端连接的套接字文件描述符，用于进行网络数据的读写操作。
- `addr_`：客户端的地址信息，包括 IP 地址和端口号。
- `iov_cnt_`：`iovec` 数组的有效元素数量，用于 `writev` 函数进行分散写操作。
- `iov_[2]`：`iovec` 结构体数组，用于存储待写入的数据块信息。

# 实现 Init() 函数

```C++
void HttpConnect::Init(int fd, const sockaddr_in& addr) {
    assert(fd > 0);
    ++use_count;
    fd_ = fd;
    is_close_ = false;
    addr_ = addr;
    read_buff_.RetrieveAll();
    write_buff_.RetrieveAll();
    LOG_INFO("Client[%d][%s:%d] in, user count: %d", fd_, GetIP(), GetPort(), (int)use_count);
}
```

# 实现 Close() 函数

```C++
void HttpConnect::Close() {
    response_.UnmapFile();
    if (is_close_ == false) {
        is_close_ = true;
        --use_count;
        close(fd_);
        LOG_INFO("Client[%d][%s:%d] quit, user count: %d", fd_, GetIP(), GetPort(), (int)use_count);
    }
}
```

# 实现 Read() 函数

- 从客户端连接的套接字文件描述符 `fd_` 中读取数据，并将其存储到 `read_buff_` 缓冲区中。
- 在边沿触发（ET）模式下，会循环读取数据，直到读取失败或没有更多数据可读；
- 在水平触发（LT）模式下，只会读取一次。

```C++
ssize_t HttpConnect::Read(int* save_errno) {
    ssize_t len = -1;
    do {
        len = read_buff_.ReadFD(fd_, save_errno);
        if (len <= 0)
            break;
    } while (is_ET); // EF: 边沿触发，循环读取数据将数据处理完
    return len;
}
```

# 实现 Write() 函数

- 将 `iov_` 数组中存储的数据通过 `writev` 函数写入到客户端连接的套接字文件描述符 `fd_` 中。
- 在边沿触发（ET）模式下，会循环写入数据直到全部写入；
- 在水平触发（LT）模式下，或需要写入的数据量较大时，也会分多次写入。

```C++
ssize_t HttpConnect::Write(int* save_errno) {
    ssize_t len = -1;
    do {
        len = writev(fd_, iov_, iov_cnt_);
        if (len <= 0) {
            *save_errno = errno;
            break;
        }

        if (iov_[0].iov_len + iov_[1].iov_len == 0) { // 读完
            break;
        } else if (static_cast<size_t>(len) > iov_[0].iov_len) { // iov[0]读完
            // 更新指针和长度
            size_t read_len = len - iov_[0].iov_len;
            iov_[1].iov_base = (uint8_t*)iov_[1].iov_base + read_len;
            iov_[1].iov_len -= read_len;
            if (iov_[0].iov_len) {
                write_buff_.RetrieveAll();
                iov_[0].iov_len = 0;
            }
        } else { // iov[0]未读完
            iov_[0].iov_base = (uint8_t*)iov_[0].iov_base + len;
            iov_[0].iov_len -= len;
            write_buff_.Retrieve(len);
        }
    } while (is_ET || ToWriteBytes() > 10240);
    // 边沿触发模式下数据被全部写入
    // 在非边沿触发模式下，或需要写入的数据量较大时，也能分多次写入
    return len;
}
```

# 实现 Process() 函数

- 初始化 `request_` 对象，调用 `request_.Parse` 函数**解析客户端的 HTTP 请求**，如果解析成功，则初始化 `response_` 对象，状态码设为 200；如果解析失败，则状态码设为 400。

- 调用 `response_.MakeResponse` 函数**生成 HTTP 响应**，并将响应数据存储到 `write_buff_` 缓冲区中。
- 设置 `iov_` 数组的信息，根据是否有文件内容决定 `iov_cnt_` 的值。

```C++
bool HttpConnect::Process() {
    request_.Init();
    if (read_buff_.ReadableBytes() <= 0)
        return false;
    if (request_.Parse(read_buff_)) {
        LOG_DEBUG("path: %s", request_.Path().c_str());
        response_.Init(request_.Path(), src_dir, 200, request_.IsKeepAlive());
    } else {
        response_.Init(request_.Path(), src_dir, 400, false);
    }

    response_.MakeResponse(write_buff_);
    iov_[0].iov_base = const_cast<char*>(write_buff_.ReadBegin());
    iov_[0].iov_len = response_.FileLen();
    iov_cnt_ = 1;

    // 报文主体文件
    if (response_.File()) {
        iov_[1].iov_base = response_.File();
        iov_[1].iov_len = response_.FileLen();
        iov_cnt_ = 2;
    }
    LOG_DEBUG("filesize: %d, %d to %d", response_.FileLen(), iov_cnt_, ToWriteBytes());
    return true;
}
```

# HttpConnect 代码

**http_connect.h**

```C++
#ifndef HTTP_CONNECT_H
#define HTTP_CONNECT_H

#include <arpa/inet.h> // sockaddr_in
#include <sys/uio.h>   // readv/writev
#include <cassert>
#include <atomic>

#include "../buffer/buffer.h"
#include "../log/log.h"
#include "http_request.h"
#include "http_response.h"

class HttpConnect {
public:
    static bool is_ET;
    static const char* src_dir;
    static std::atomic<int> use_count;

    HttpConnect();
    ~HttpConnect();

    void Init(int socket_fd, const sockaddr_in& addr);
    void Close();

    ssize_t Read(int* save_errno);
    ssize_t Write(int* save_errno);
    bool Process();

    // 写的总长度
    int ToWriteBytes() const { return iov_[0].iov_len + iov_[1].iov_len; }
    bool IsKeepAlive() const { return request_.IsKeepAlive(); }

    int GetFd() const { return fd_; }
    int GetPort() const { return addr_.sin_port; }
    const char* GetIP() const { return inet_ntoa(addr_.sin_addr); }
    sockaddr_in GetAddr() const { return addr_; }

private:
    int fd_;
    bool is_close_;
    struct sockaddr_in addr_;

    int iov_cnt_;
    struct iovec iov_[2];

    Buffer read_buff_;
    Buffer write_buff_;    
    
    HttpRequest request_;
    HttpResponse response_;
};

#endif // HTTP_CONNECT_H
```

**http_connect.cc**

```C++
#include "http_connect.h"

bool HttpConnect::is_ET;
const char* HttpConnect::src_dir;
std::atomic<int> HttpConnect::use_count;

HttpConnect::HttpConnect()
    : fd_(-1), is_close_(true) {
    memset(&addr_, 0, sizeof(addr_));
}

HttpConnect::~HttpConnect() {
    Close();
}

void HttpConnect::Init(int fd, const sockaddr_in& addr) {
    assert(fd > 0);
    ++use_count;
    fd_ = fd;
    is_close_ = false;
    addr_ = addr;
    read_buff_.RetrieveAll();
    write_buff_.RetrieveAll();
    LOG_INFO("Client[%d][%s:%d] in, user count: %d", fd_, GetIP(), GetPort(), (int)use_count);
}

void HttpConnect::Close() {
    response_.UnmapFile();
    if (is_close_ == false) {
        is_close_ = true;
        --use_count;
        close(fd_);
        LOG_INFO("Client[%d][%s:%d] quit, user count: %d", fd_, GetIP(), GetPort(), (int)use_count);
    }
}

ssize_t HttpConnect::Read(int* save_errno) {
    ssize_t len = -1;
    do {
        len = read_buff_.ReadFD(fd_, save_errno);
        if (len <= 0)
            break;
    } while (is_ET); // EF: 边沿触发一次性全部读出
    return len;
}

ssize_t HttpConnect::Write(int* save_errno) {
    ssize_t len = -1;
    do {
        len = writev(fd_, iov_, iov_cnt_);
        if (len <= 0) {
            *save_errno = errno;
            break;
        }

        if (iov_[0].iov_len + iov_[1].iov_len == 0) { // 读完
            break;
        } else if (static_cast<size_t>(len) > iov_[0].iov_len) { // iov[0]读完
            // 更新指针和长度
            size_t read_len = len - iov_[0].iov_len;
            iov_[1].iov_base = (uint8_t*)iov_[1].iov_base + read_len;
            iov_[1].iov_len -= read_len;
            if (iov_[0].iov_len) {
                write_buff_.RetrieveAll();
                iov_[0].iov_len = 0;
            }
        } else { // iov[0]未读完
            iov_[0].iov_base = (uint8_t*)iov_[0].iov_base + len;
            iov_[0].iov_len -= len;
            write_buff_.Retrieve(len);
        }
    } while (is_ET || ToWriteBytes() > 10240);
    // 边沿触发模式下数据被全部写入
    // 在非边沿触发模式下，当需要写入的数据量较大时，也能分多次写入
    return len;
}

// 解析HTTP响应，生成HTTP报文
bool HttpConnect::process() {
    request_.Init();
    if (read_buff_.ReadableBytes() <= 0)
        return false;
    if (request_.Parse(read_buff_)) {
        LOG_DEBUG("path: %s", request_.Path().c_str());
        response_.Init(request_.Path(), src_dir, 200, request_.IsKeepAlive());
    } else {
        response_.Init(request_.Path(), src_dir, 400, false);
    }

    response_.MakeResponse(write_buff_);
    iov_[0].iov_base = const_cast<char*>(write_buff_.ReadBegin());
    iov_[0].iov_len = response_.FileLen();
    iov_cnt_ = 1;

    // 报文主体文件
    if (response_.File()) {
        iov_[1].iov_base = response_.File();
        iov_[1].iov_len = response_.FileLen();
        iov_cnt_ = 2;
    }
    LOG_DEBUG("filesize: %d, %d to %d", response_.FileLen(), iov_cnt_, ToWriteBytes());
    return true;
}
```

# HttpConnect 测试

**测试Httpconnect的`Read`，`Write`，`Process`函数，通过调试观察，跟踪变量和函数流程。**

```C++
#include "../code/http/http_connect.h"
#include <iostream>
#include <sys/socket.h>
#include <unistd.h>

// 创建一对本地套接字
void CreateSocketPair(int& sock1, int& sock2) {
    int socks[2];
    assert(socketpair(AF_UNIX, SOCK_STREAM, 0, socks) == 0);
    sock1 = socks[0];
    sock2 = socks[1];
}

void TestHttpConnect() {
    Log* logger = Log::GetInstance();
    logger->Init(0, "./logs/", ".log", 1024);

    int client_sock, server_sock;
    CreateSocketPair(client_sock, server_sock);

    sockaddr_in addr;
    addr.sin_family = AF_INET;
    addr.sin_addr.s_addr = INADDR_ANY;
    addr.sin_port = htons(8080);

    HttpConnect conn;
    conn.Init(server_sock, addr);

    // 测试Read函数
    {
        int save_errno;
        std::string http_request = "GET /index.html HTTP/1.1\r\n"
                                   "Host: example.com\r\n"
                                   "Connection: keep-alive\r\n"
                                   "\r\n";
        // client_sock 和 server_sock 是相互连接的
        // 以写入到 client_sock 的数据会被自动传输到 server_sock 的接收缓冲区中。
        write(client_sock, http_request.c_str(), http_request.size());
        ssize_t len = conn.Read(&save_errno);
        assert(len == (ssize_t)http_request.size());
    }

    // 测试Process函数
    {
        HttpConnect::src_dir = "/home/Tian/tiny_web_server/resources";
        bool result = conn.Process();
        assert(result);
    }

    // 测试Write函数
    {
        int saveErrno;
        ssize_t len = conn.Write(&saveErrno);
        assert(len >= -1);

        char buffer[10240];
        ssize_t recv_len = recv(client_sock, buffer, sizeof(buffer), -1);
        assert(recv_len == len);
    }
    close(client_sock);
    close(server_sock);
}

int main() {
    TestHttpConnect();
    return 0;
}
```

**CMakeList.txt**

```cmake
cmake_minimum_required(VERSION 3.10)
project(tests)

# 设置 C++ 标准和编译器选项
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra")

# 定义公共源文件和特定文件
set(COMMON ../code/buffer/buffer.cc ../code/log/log.cc)
set(SQL_POOL ../code/pool/sql_connect_pool.cc)
set(HTTP_REQUEST ../code/http/http_request.cc)
set(HTTP_RESPONSE ../code/http/http_response.cc)
set(HTTP_CONNECT ../code/http/http_connect.cc)
set(HTTP ../code/pool/sql_connect_pool.cc ../code/http/http_request.cc 
         ../code/http/http_response.cc ../code/http/http_connect.cc)

# 查找 MySQL 库
find_package(PkgConfig REQUIRED)
pkg_check_modules(MYSQL REQUIRED mysqlclient)
# 包含 MySQL 头文件目录
include_directories(${MYSQL_INCLUDE_DIR})

add_executable(http_connect_test http_connect_test.cc ${COMMON} ${HTTP})
target_link_libraries(http_connect_test ${MYSQL_LIBRARIES})
```

