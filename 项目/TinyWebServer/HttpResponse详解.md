# HTTP 请求报文
![image.png](https://s2.loli.net/2025/02/28/ip2KSecV1sUAQ8l.png)

![image.png](https://s2.loli.net/2025/02/28/9iXlHq1ac2gnuF6.png)

一个典型的 HTTP 响应包含以下部分：

- **状态行**：例如 `HTTP/1.1 200 OK`。
- **响应头**：包含各种元数据，如`Content-Type`、`Content-Length`、`Connection`等。
- **响应体**：实际传输的数据，如 HTML 文件内容。

# HttpResponse 类

在 Web 服务器中，`HttpResponse` 类起着至关重要的作用，主要负责处理服务器向客户端发送的 HTTP 响应。具体来说，它承担了以下几个关键任务：

1. **响应构建与内容处理**：依据服务器对客户端请求的处理结果，生成符合 HTTP 协议规范的响应报文。该过程涵盖设置合适的 HTTP 状态码及对应的状态描述，添加必要的响应头信息（如 `Connection`、`Content-type`、`Content-length` 等），同时处理响应体内容。若请求的是文件资源，会将文件内容映射到内存并作为响应体；若出现错误，则生成包含错误信息的 HTML 页面作为响应体。
2. **资源管理**：负责管理与响应相关的资源，如文件映射的内存。在不需要这些资源时，会及时释放，避免资源泄漏，确保服务器的稳定运行。
3. **与客户端交互**：将生成好的完整 HTTP 响应发送给客户端，使得客户端能够根据响应内容进行相应的处理，如显示网页、下载文件等，从而实现服务器与客户端之间的有效通信。

```C++
class HttpResponse {
public:
    HttpResponse();
    ~HttpResponse();

    void Init(const std::string& path, const std::string& src_dir, int code = -1, bool is_keep_alive = false);
    void UnmapFile();

    void MakeResponse(Buffer& buff);
    void ErrorContent(Buffer& buff, const std::string& message);

    char* File() { return mmfile_; }
    size_t FileLen() const { return mmfile_stat_.st_size; }
    int Code() const { return code_; }

private:
    void AddStateLine(Buffer& buff);
    void AddHeader(Buffer& buff);
    void AddContent(Buffer& buff);

    void ErrorHtml();
    std::string GetFileType();

    static const std::unordered_map<int, std::string> CODE_STATUS;          // 编码状态集
    static const std::unordered_map<int, std::string> CODE_PATH;            // 编码路径集
    static const std::unordered_map<std::string, std::string> SUFFIX_TYPE;  // 后缀状态集

    int code_; // 响应状态码
    bool is_keep_alive_;

    std::string path_;
    std::string src_dir_;

    char* mmfile_;
    struct stat mmfile_stat_;
};
```

# 实现构造和析构函数

```C++
HttpResponse::HttpResponse() 
    : code_(-1), is_keep_alive_(false),
      path_(""), src_dir_(""), mmfile_(nullptr) {
        memset(&mmfile_stat_, 0, sizeof(mmfile_stat_));
    }

HttpResponse::~HttpResponse() {
    UnmapFile();
}
```

# 实现 Init() 函数

```C++
void HttpResponse::Init(const std::string& path, const std::string& src_dir, 
                        int code, bool is_keep_alive) {
    assert(src_dir != "");
    code_ = code;
    is_keep_alive_ = is_keep_alive;
    path_ = path;
    src_dir_ = src_dir;
    mmfile_ = nullptr;
    memset(&mmfile_stat_, 0, sizeof(mmfile_stat_));
}
```

# 实现 UnmapFile() 函数

```C++
// 释放文件映射的内存
void HttpResponse::UnmapFile() {
    if (mmfile_) {
        munmap(mmfile_, mmfile_stat_.st_size); // 解除内存映射
        mmfile_ = nullptr;
    }
}
```

# 实现 ErrorHtml() 函数

- 当 HTTP 状态码表示出现错误时，根据状态码更新 `path_` 。
- 使用 `stat` 函数获取错误页面文件的状态信息，存储在 `mmfile_stat_` 中。

```C++
// 状态出错，更新路径，以及文件状态
void HttpResponse::ErrorHtml() {
    if (CODE_PATH.count(code_) == 1) {
        path_ = CODE_PATH.find(code_)->second;
        stat((src_dir_ + path_).c_str(), &mmfile_stat_);
    }
}
```

# 实现 AddStateLine() 函数

- 根据 `code_`添加 HTTP 响应的状态行到`buff`中。

```C++
void HttpResponse::AddStateLine(Buffer& buff) {
    std::string status;
    if (CODE_STATUS.count(code_) == 1) {
        status = CODE_STATUS.find(code_)->second;
    } else {
        code_ = 400;
        status = CODE_STATUS.find(code_)->second;
    }
    buff.Append("HTTP/1.1 " + std::to_string(code_) + " " + status + "\r\n");
}
```

# 实现 AddHeader() 函数

- 添加 HTTP 响应的状态行到`buff`中。

```C++
void HttpResponse::AddHeader(Buffer& buff) {
    buff.Append("Connection: ");
    if (is_keep_alive_) {
        buff.Append("keep-alive\r\n");
        buff.Append("keep-alive: max=6 timeout=120\r\n");
    } else {
        buff.Append("close\r\n");
    }
    buff.Append("Content-type: " + GetFileType() + "\r\n");
}
```

# 实现 GetFileType() 函数

- 查找文件路径中最后一个 `.` 的位置，若未找到则返回默认的 MIME 类型 `text/plain`。
- 提取后缀后在 `SUFFIX_TYPE` 中查找对应的 MIME 类型，若找到则返回该类型；否则返回默认的 `text/plain`。

```C++
std::string HttpResponse::GetFileType() {
    std::string::size_type idx = path_.find_last_of(".");
    if (idx == std::string::npos)
        return "text/plain"; // 文本类型
    std::string suffix = path_.substr(idx);
    if (SUFFIX_TYPE.count(suffix) == 1)
        return SUFFIX_TYPE.find(suffix)->second;
    return "text/plain";
}
```

# 实现 AddContent() 函数

- 拼接完整的文件路径，使用 `open` 函数以只读模式打开文件。若打开失败，则调用 `ErrorContent` 函数生成错误响应内容。
- 使用 `mmap` 函数将文件映射到内存中，若映射失败则关闭文件描述符，并调用 `ErrorContent` 函数。
- 将 `mmfile_` 指向映射的内存区域，关闭文件描述符，并添加 `Content-length` 头部。

```C++
void HttpResponse::AddContent(Buffer& buff) {
    std::string path = src_dir_ + path_;
    int src_fd = open(path.c_str(), O_RDONLY);  
    if (src_fd < 0) {
        ErrorContent(buff, "File Not Found!");
        return;
    }

    LOG_DEBUG("file path %s", path.c_str());
    void* mmret = mmap(0, mmfile_stat_.st_size, PROT_READ, MAP_PRIVATE, src_fd, 0);
    if (mmret == MAP_FAILED) {
        close(src_fd);
        ErrorContent(buff, "mmap fail!");
        return;
    }
    mmfile_ = (char*)mmret;
    close(src_fd);
    buff.Append("Content-length: " + std::to_string(mmfile_stat_.st_size) + "\r\n\r\n");
}
```

# 实现 ErrorContent() 函数

- 构建一个 HTML 格式的错误页面，包含状态码、状态描述和错误消息。
- 添加 `Content-length` 头部，后面跟着两个换行符表示头部结束，最后将错误页面响应报文追加到 `buff` 中。

```C++
void HttpResponse::ErrorContent(Buffer& buff, const std::string& message) {
    std::string body;
    std::string status;
    body += "<html><title>Error</title>";
    body += "<body bgcolor=\"fffff\">";
    if (CODE_STATUS.count(code_) == 1)
        status = CODE_STATUS.find(code_)->second;
    else
        status = "Bad Request";
    body += std::to_string(code_) + " : " + status + "\n";
    body += "<p>" + message + "</p>";
    body += "<hr><em>TinyWebServer</em><body></html>";

    buff.Append("Content-length: " + std::to_string(body.size()) + "\r\n\r\n");
    buff.Append(body);
}
```

# 实现 MakeResponse() 函数

- 使用 `stat` 函数获取文件的状态信息，根据不同情况设置状态码。
- 调用 `ErrorHtml` 函数处理错误页面，依次调用 `AddStateLine`、`AddHeader` 和 `AddContent` 函数，分别添加状态行、头部信息和响应内容到`buff`中。

```C++
void HttpResponse::MakeResponse(Buffer& buff) {
    if (stat((src_dir_ + path_).c_str(), &mmfile_stat_) < 0) {
        LOG_WARN("stat fail: error: %s", strerror(errno));
        code_ = 404;
    } else if (S_ISDIR(mmfile_stat_.st_mode)) {
        code_ = 404;
    } else if (!(mmfile_stat_.st_mode & S_IROTH)) {
        code_ = 403;
    } else if (code_ == -1) {
        code_ = 200;
    }
        
    ErrorHtml();
    AddStateLine(buff);
    AddHeader(buff);
    AddContent(buff);
}
```

# HttpResponse 代码

**http_response.h**

```C++
#ifndef HTTP_RESPONSE_H
#define HTTP_RESPONSE_H

#include <sys/stat.h> // stat
#include <sys/mman.h> // mmap, munmap
#include <fcntl.h>    // open
#include <unistd.h>   // close

#include <cstring>    // memset
#include <cassert>
#include <string>
#include <unordered_map>

#include "../buffer/buffer.h"
#include "../log/log.h"

class HttpResponse {
public:
    HttpResponse();
    ~HttpResponse();

    void Init(const std::string& path, const std::string& src_dir, int code = -1, bool is_keep_alive = false);
    void UnmapFile();

    void MakeResponse(Buffer& buff);
    void ErrorContent(Buffer& buff, const std::string& message);

    char* File() { return mmfile_; }
    size_t FileLen() const { return mmfile_stat_.st_size; }
    int Code() const { return code_; }

private:
    void AddStateLine(Buffer& buff);
    void AddHeader(Buffer& buff);
    void AddContent(Buffer& buff);

    void ErrorHtml();
    std::string GetFileType();

    static const std::unordered_map<int, std::string> CODE_STATUS;          // 编码状态集
    static const std::unordered_map<int, std::string> CODE_PATH;            // 编码路径集
    static const std::unordered_map<std::string, std::string> SUFFIX_TYPE;  // 后缀状态集

    int code_; // 响应状态码
    bool is_keep_alive_;

    std::string path_;
    std::string src_dir_;

    char* mmfile_;
    struct stat mmfile_stat_;
};

#endif // HTTP_RESPONSE_H
```

**http_response.cc**

```C++
#include "http_response.h"

const std::unordered_map<int, std::string> HttpResponse::CODE_STATUS = {
    {200, "OK"},
    {400, "Bad Requeset"},
    {403, "Forbidden"},
    {404, "Not Found"},
};

const std::unordered_map<int, std::string> HttpResponse::CODE_PATH = {
    {400, "/400.html"},
    {403, "/403.html"},
    {404, "/404.html"},
};

const std::unordered_map<std::string, std::string> HttpResponse::SUFFIX_TYPE = {
    {".html",  "text/html"},
    {".xml",   "text/xml"},
    {".xhtml", "application/xhtml+xml"},
    {".txt",   "text/plain"},
    {".rtf",   "application/rtf"},
    {".pdf",   "application/pdf"},
    {".word",  "application/nsword"},
    {".png",   "image/png"},
    {".gif",   "image/gif"},
    {".jpg",   "image/jpeg"},
    {".jpeg",  "image/jpeg"},
    {".au",    "audio/basic"},
    {".mpeg",  "video/mpeg"},
    {".mpg",   "video/mpeg"},
    {".avi",   "video/x-msvideo"},
    {".gz",    "application/x-gzip"},
    {".tar",   "application/x-tar"},
    {".css",   "text/css"},
    {".js",    "text/javascript"},
};

HttpResponse::HttpResponse() 
    : code_(-1), is_keep_alive_(false),
      path_(""), src_dir_(""), mmfile_(nullptr) {
        memset(&mmfile_stat_, 0, sizeof(mmfile_stat_));
    }

HttpResponse::~HttpResponse() {
    UnmapFile();
}

void HttpResponse::Init(const std::string& path, const std::string& src_dir, 
                        int code, bool is_keep_alive) {
    assert(src_dir != "");
    code_ = code;
    is_keep_alive_ = is_keep_alive;
    path_ = path;
    src_dir_ = src_dir;
    mmfile_ = nullptr;
    memset(&mmfile_stat_, 0, sizeof(mmfile_stat_));
}

// 释放文件映射的内存
void HttpResponse::UnmapFile() {
    if (mmfile_) {
        munmap(mmfile_, mmfile_stat_.st_size); // 解除内存映射
        mmfile_ = nullptr;
    }
}

// 状态出错，更新路径，以及文件状态
void HttpResponse::ErrorHtml() {
    if (CODE_PATH.count(code_) == 1) {
        path_ = CODE_PATH.find(code_)->second;
        stat((src_dir_ + path_).c_str(), &mmfile_stat_);
    }
}

void HttpResponse::AddStateLine(Buffer& buff) {
    std::string status;
    if (CODE_STATUS.count(code_) == 1) {
        status = CODE_STATUS.find(code_)->second;
    } else {
        code_ = 400;
        status = CODE_STATUS.find(code_)->second;
    }
    buff.Append("HTTP/1.1 " + std::to_string(code_) + " " + status + "\r\n");
}

void HttpResponse::AddHeader(Buffer& buff) {
    buff.Append("Connection: ");
    if (is_keep_alive_) {
        buff.Append("keep-alive\r\n");
        buff.Append("keep-alive: max=6 timeout=120\r\n");
    } else {
        buff.Append("close\r\n");
    }
    buff.Append("Content-type: " + GetFileType() + "\r\n");
}

std::string HttpResponse::GetFileType() {
    std::string::size_type idx = path_.find_last_of(".");
    if (idx == std::string::npos)
        return "text/plain"; // 文本类型
    std::string suffix = path_.substr(idx);
    if (SUFFIX_TYPE.count(suffix) == 1)
        return SUFFIX_TYPE.find(suffix)->second;
    return "text/plain";
}

void HttpResponse::AddContent(Buffer& buff) {
    std::string path = src_dir_ + path_;
    int src_fd = open(path.c_str(), O_RDONLY);  
    if (src_fd < 0) {
        ErrorContent(buff, "File Not Found!");
        return;
    }

    LOG_DEBUG("file path %s", path.c_str());
    void* mmret = mmap(0, mmfile_stat_.st_size, PROT_READ, MAP_PRIVATE, src_fd, 0);
    if (mmret == MAP_FAILED) {
        close(src_fd);
        ErrorContent(buff, "mmap fail!");
        return;
    }
    mmfile_ = (char*)mmret;
    close(src_fd);
    buff.Append("Content-length: " + std::to_string(mmfile_stat_.st_size) + "\r\n\r\n");
}

void HttpResponse::ErrorContent(Buffer& buff, const std::string& message) {
    std::string body;
    std::string status;
    body += "<html><title>Error</title>";
    body += "<body bgcolor=\"fffff\">";
    if (CODE_STATUS.count(code_) == 1)
        status = CODE_STATUS.find(code_)->second;
    else
        status = "Bad Request";
    body += std::to_string(code_) + " : " + status + "\n";
    body += "<p>" + message + "</p>";
    body += "<hr><em>TinyWebServer</em><body></html>";

    buff.Append("Content-length: " + std::to_string(body.size()) + "\r\n\r\n");
    buff.Append(body);
}

void HttpResponse::MakeResponse(Buffer& buff) {
    if (stat((src_dir_ + path_).c_str(), &mmfile_stat_) < 0) {
        LOG_WARN("stat fail: error: %s", strerror(errno));
        code_ = 404;
    } else if (S_ISDIR(mmfile_stat_.st_mode)) {
        code_ = 404;
    } else if (!(mmfile_stat_.st_mode & S_IROTH)) {
        code_ = 403;
    } else if (code_ == -1) {
        code_ = 200;
    }
        
    ErrorHtml();
    AddStateLine(buff);
    AddHeader(buff);
    AddContent(buff);
}
```

# HttpResponse 测试

**测试请求报文的构建**

```C++

```

![image.png](https://s2.loli.net/2025/03/01/rchgpEvURSq41FV.png)

**CMakeLists.txt**

```cmake
cmake_minimum_required(VERSION 3.10)
project(tests)

# 设置 C++ 标准和编译器选项
set(CMAKE_BUILD_TYPE Debug)
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra")

# 定义公共源文件和特定文件
set(COMMON ../code/buffer/buffer.cc ../code/log/log.cc)
set(HTTP_REQUEST ../code/http/http_request.cc)
set(HTTP_RESPONSE ../code/http/http_response.cc)
set(SQL_POOL ../code/pool/sql_connect_pool.cc)

add_executable(http_response_test http_response_test.cc ${COMMON} ${HTTP_RESPONSE})
```
