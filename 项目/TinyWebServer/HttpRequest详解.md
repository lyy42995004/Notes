# HTTP 请求报文

![ 2025-02-27 200519.png](https://s2.loli.net/2025/02/27/xorUpuKgmtZceMs.png)

![ 2025-02-27 200654.png](https://s2.loli.net/2025/02/27/ybKgAozr2G1f5Dd.png)

一个典型的 HTTP 请求包含请求行、请求头和请求体（可选）。

- **请求行**：包含请求方法（如 `GET`、`POST` 等）、请求路径（如 `/index.html`）和 HTTP 版本（如 `HTTP/1.1`）。
- **请求头**：包含多个键值对，用于传递额外的信息，如 `Host`、`User-Agent` 等。
- **请求体**：通常在 `POST` 请求中使用，用于传递数据。

# HttpRequest 类

在 Web 服务器中，`HttpRequest`类起着至关重要的作用，主要负责处理客户端发送的 HTTP 请求。具体来说，它承担了以下几个关键任务：

1. **请求解析**：接收客户端发送的 HTTP 请求报文，并对其进行解析，将请求报文中的各个部分（如请求行、请求头、请求体等）提取出来，以便后续处理。
2. **状态管理**：在解析过程中，通过状态机来管理解析的状态，确保解析过程的正确性和顺序性。
3. **请求处理**：根据解析结果，进行相应的处理，例如处理 GET 请求、POST 请求等，并根据请求的内容和服务器的配置，决定如何响应客户端。

```C++
class HttpRequest {
public:
    enum PARSE_STATE {
        REQUEST_LINE,
        HEADERS,
        BODY,
        FINISH
    };

    HttpRequest() { Init(); }
    ~HttpRequest() = default;

    void Init();
    bool Parse(Buffer& buff);

    std::string Method() const;
    std::string Path() const;
    std::string& Path();
    std::string Version() const;
    std::string GetPost(const std::string& key) const;
    std::string GetPost(const char* key) const;

    bool IsKeepAlive() const;

private:
    static int ConverHex(char ch);	// 16进制转10进制
    static bool UserVerify(const std::string& name, const std::string& pwd, bool is_login);	// 验证用户

    bool ParseRequestLine(const std::string& line);	// 解析请求行
    void ParseHeader(const std::string& line);		// 解析请求头
    void ParseBody(const std::string& line);		// 解析请求体

    void ParsePath();	// 解析请求路径
    void ParsePost();	// 解析Post事件
    void ParseFromUrlEncoded();	// 解析url

    static const std::unordered_set<std::string> DEFAULT_HTML;
    static const std::unordered_map<std::string, int> DEFAULT_HTML_TAG;

    PARSE_STATE state_;
    std::string method_;
    std::string path_;
    std::string version_;
    std::string body_;
    std::unordered_map<std::string, std::string> header_; // HTTP 请求的头部信息
    std::unordered_map<std::string, std::string> post_;   // POST 请求的数据
};
```

# 实现 Init() 函数

```C++
void HttpRequest::Init() {
    state_ = REQUEST_LINE;
    method_ = path_ = version_ = body_ = "";
    header_.clear();
    post_.clear();
}
```

# 实现 ParseRequestLine() 函数

- 使用正则表达式`^([^ ]*) ([^ ]*) HTTP/([^ ]*)$`以及`std::regex_match`来匹配请求行。
- 若匹配成功，将匹配结果分别存储到`method_`、`path_`和`version_`中，并将解析状态设置为`HEADERS`，表示接下来要解析请求头，返回`true`。

```C++
bool HttpRequest::ParseRequestLine(const std::string& line) {
    // GET /index.html HTTP/1.1
    std::regex patten("^([^ ]*) ([^ ]*) HTTP/([^ ]*)$");
    std::smatch match;
    // 匹配指定字符串整体是否符合
    if (std::regex_match(line, match, patten)) { 
        method_ = match[1];
        path_ = match[2];
        version_ = match[3];
        state_ = HEADERS;
        return true;
    } 
    LOG_ERROR("RequestLine Error"); 
    return false;
}
```

# 实现 ParseHeader() 函数

- 如果路径为`/`，则将其修改为`/index.html`。
- 否则，检查路径是否在`DEFAULT_HTML`中，如果存在则在路径后面添加`.html`后缀。

```C++
void HttpRequest::ParseHeader(const std::string& line) {
    std::regex patten("^([^:]*): ?(.*)$");
    std::smatch match;
    if (std::regex_match(line, match, patten)) {
        header_[match[1]] = match[2];
    } else {
        state_ = BODY;
    }
}
```

# 实现 ParsePath() 函数

- 如果路径为`/`，则将其修改为`/index.html`。
- 否则，检查路径是否在`DEFAULT_HTML`中，如果存在则在路径后面添加`.html`后缀

```C++
void HttpRequest::ParsePath() {
    if (path_ == "/") {
        path_ = "/index.html";
    } else {
        if (DEFAULT_HTML.find(path_) != DEFAULT_HTML.end())
            path_ += ".html";
    }
}
```

# 实现 ParseBody() 函数

- 将请求体内容存储到`body_`中。
- 调用`ParsePost`函数处理 POST 请求数据。
- 将解析状态设置为`FINISH`，表示解析完成。

```C++
void HttpRequest::ParseBody(const std::string& line) {
    body_ = line;
    ParsePost();    
    state_ = FINISH;
    LOG_DEBUG("Body: %s, len: %d", line.c_str(), line.size());
}
```

# 实现 ParsePost() 函数

- 检查请求方法是否为`POST`，并且请求头中的`Content-Type`是否为`application/x-www-form-urlencoded`。
- 如果满足条件，调用`ParseFromUrlencoded`函数解析 URL 编码的请求体。
- 检查路径是否为登录或注册相关路径，如果是则调用`UserVerify`函数，进行用户验证，根据验证结果设置相应的跳转路径。

```C++
void HttpRequest::ParsePost() {
    if (method_ == "POST" && header_["Content-Type"] == "application/x-www-form-urlencoded") {
        ParseFromUrlEncoded();
        if (DEFAULT_HTML_TAG.count(path_)) { // 登录/注册
            int tag = DEFAULT_HTML_TAG.find(path_)->second;
            LOG_DEBUG("Tag: %d", tag);
            if (tag == 0 || tag == 1) {
                bool is_login = (tag == 1);
                if (UserVerify(post_["username"], post_["password"], is_login))
                    path_ = "/welcome.html";
                else
                    path_ = "/error.html";
            }
        }
    }
}
```

# 实现 ParseFromUrlEncoded() 函数

- 首先检查请求体是否为空，如果为空则直接返回。
- 遍历请求体字符串，根据不同的字符进行不同的处理：
  - 遇到`=`，将之前的字符作为键存储到`key`中，并更新起始位置`j`。
  - 遇到`&`，将之前的字符作为值存储到`value`中，并将键值对存储到`post_`容器中，同时更新起始位置`j`。
  - 遇到`+`，将其替换为空格。
  - 遇到`%`，将其后的两个十六进制字符转换为十进制整数，并更新请求体内容。
- 最后处理剩余的键值对。

```C++
void HttpRequest::ParseFromUrlEncoded() {
    if (body_.size() == 0)
        return;
    
    std::string key, value;
    int num = 0;
    int n = body_.size();
    int i = 0, j = 0;

    while (i < n) {
        char ch = body_[i];
        // username=john%20doe&password=123456
        switch (ch) {
        case '=': // 获取键值对
            key = body_.substr(j, i - j); 
            j = i + 1;
            break;
        case '&': // 获取键值对
            value = body_.substr(j, i - j);
            j = i + 1;
            post_[key] = value;
            LOG_DEBUG("%s = %s", key.c_str(), value.c_str());
            break;
        case '+': // 替换为空格
            body_[i] = ' ';
            break;
        case '%':
            num = ConverHex(body_[i + 1]) * 16 + ConverHex(body_[i + 2]);
            body_[i + 2] = num % 10 + '0';
            body_[i + 1] = num / 10 + '0';
            i += 2;
            break;
        default:
            break;
        }
        i++;
    }
    assert(j <= i);
    if (post_.count(key) == 0 && j < i) {
        value = body_.substr(j, i - j);
        post_[key] = value;
    }
}
```

# 实现 UserVerify() 函数

- 首先检查用户名和密码是否为空，如果为空则返回`false`。
- 使用`SqlConnRAII`管理数据库连接，构造查询语句，查询数据库中是否存在该用户名。
- 如果是登录行为，比较输入的密码和数据库中的密码是否一致，一致则验证成功。
- 如果是注册行为，检查用户名是否已被使用，若未被使用则将用户信息插入数据库。

```C++
// 验证用户的用户名和密码
bool HttpRequest::UserVerify(const std::string& name, const std::string& pwd, bool is_login) {
    if (name == "" || pwd == "")
        return false;

    LOG_INFO("Verify name: %s pwd: %s", name.c_str(), pwd.c_str());
    MYSQL* sql;
    SqlConnectRAII(&sql, SqlConnectPool::GetInstance());
    assert(sql);

    bool flag = false;
    char order[256] = {0};
    MYSQL_RES* res = nullptr;      // 查询结果集

    if (!is_login) // 注册
        flag = true;
    snprintf(order, 256, "SELECT username, password FROM User WHERE username='%s' LIMIT 1", name.c_str());
    LOG_DEBUG("%s", order);

    // 查询用户信息
    if (mysql_query(sql, order)) {
        if (res)
            mysql_free_result(res); // 查询失败，释放结果集
        return false;
    }
    res = mysql_store_result(sql);   // 存储查询结果到res中

    // 处理查询结果
    while (MYSQL_ROW row = mysql_fetch_row(res)) {
        LOG_DEBUG("MYSQL ROW: %s %s", row[0], row[1]);
        std::string password = row[1];
        if (is_login) {
            if (pwd == password) {
                flag = true;
            } else {
                flag = false;
                LOG_INFO("pwd error!");
            }
        } else {
            flag = false;
            LOG_INFO("user used!");
        }
    }
    mysql_free_result(res);

    // 注册（用户名未被使用）
    if (!is_login && flag) {
        LOG_DEBUG("register");
        bzero(order, 256);
        snprintf(order, 256, "INSERT INTO User(username, password) VALUES('%s', '%s')", name.c_str(), pwd.c_str());
        LOG_DEBUG("%s", order);
        if (mysql_query(sql, order)) {
            LOG_ERROR("MySQL insert fail: Error: %s", mysql_error(sql));
            flag = false;
        } else {
            LOG_DEBUG("UserVerify success!");
            flag = true;
        }
    }
    return flag;
}
```

# 实现 Parse() 函数

- 首先检查`Buffer`中是否有可读数据，如果没有则返回`false`。

- 循环读取`Buffer`中的数据，直到没有可读数据或者解析状态为`FINISH`。

- 利用`search`函数查找`\r\n`来确定每行数据的结束位置，将每行数据提取出来存为`line`。

- 根据当前解析状态`state_`的不同，调用不同的解析函数进行处理：

  - 若为`REQUEST_LINE`状态，调用`ParseRequestLine_`解析请求行，若解析失败则返回`false`，成功则继续解析路径。
  - 若为`HEADERS`状态，调用`ParseHeader_`解析请求头，若`Buffer`中剩余可读字节小于等于 2，则认为是 GET 请求，将状态设置为`FINISH`。
  - 若为`BODY`状态，调用`ParseBody_`解析请求体。

- 如果`lineend`到达`Buffer`的写指针位置，说明数据读完，调用`buff.RetrieveAll()`清空`Buffer`，并跳出循环；否则跳过`\r\n`继续解析。

- 最后记录解析出的请求方法、路径和版本信息，并返回`true`表示解析成功。

```C++
bool HttpRequest::Parse(Buffer& buff) {
    const char* END = "\r\n";
    if (buff.ReadableBytes() == 0)
        return false;
    
    while (buff.ReadableBytes() && state_ != FINISH) {
        // 找到buff中，首次出现"\r\n"的位置
        const char* line_end = std::search(buff.ReadBegin(), buff.WriteBeginConst(), END, END + 2);
        string line(buff.ReadBegin(), line_end);
        switch (state_) {
        case REQUEST_LINE:
            if (ParseRequestLine(line) == false)
                return false;
            ParsePath();
            break;
        case HEADERS:
            ParseHeader(line);
            if (buff.ReadableBytes() <= 2) // get请求，提前结束
                state_ = FINISH;
            break;
        case BODY:
            ParseBody(line);
            break;
        default:
            break;
        }
        if (line_end == buff.WriteBegin()) { // 读完了
            buff.RetrieveAll();
            break;
        }
        buff.RetrieveUntil(line_end + 2); // 跳过回车换行
    }
    LOG_DEBUG("[%s] [%s] [%s]", method_ .c_str(), path_.c_str(), version_.c_str());
    return true;
}
```

# HttpRequest 代码

**http_request.h**

```C++
#ifndef HTTP_REQUEST_H
#define HTTP_REQUEST_H

#include <cstring>
#include <string>
#include <unordered_set>
#include <unordered_map>
#include <algorithm>
#include <regex> // 正则表达式
#include <mysql/mysql.h>

#include "../buffer/buffer.h"
#include "../log/log.h"
#include "../pool/sql_connect_pool.h"

class HttpRequest {
public:
    enum PARSE_STATE {
        REQUEST_LINE,
        HEADERS,
        BODY,
        FINISH
    };

    HttpRequest() { Init(); }
    ~HttpRequest() = default;

    void Init();
    bool Parse(Buffer& buff);

    std::string Method() const;
    std::string Path() const;
    std::string& Path();
    std::string Version() const;
    std::string GetPost(const std::string& key) const;
    std::string GetPost(const char* key) const;

    bool IsKeepAlive() const;

private:
    static int ConverHex(char ch);
    static bool UserVerify(const std::string& name, const std::string& pwd, bool is_login);

    bool ParseRequestLine(const std::string& line);
    void ParseHeader(const std::string& line);
    void ParseBody(const std::string& line);

    void ParsePath();
    void ParsePost();
    void ParseFromUrlEncoded();

    static const std::unordered_set<std::string> DEFAULT_HTML;
    static const std::unordered_map<std::string, int> DEFAULT_HTML_TAG;

    PARSE_STATE state_;
    std::string method_;
    std::string path_;
    std::string version_;
    std::string body_;
    std::unordered_map<std::string, std::string> header_; // HTTP 请求的头部信息
    std::unordered_map<std::string, std::string> post_;   // POST 请求的数据
};

#endif // HTTP_REQUEST_H
```

**http_request.cc**

```C++
#include "http_request.h"

const std::unordered_set<std::string> HttpRequest::DEFAULT_HTML {
    "/index", "/register", "/login", "/welcome", "/video", "/picture"
};

// 登录/注册
const std::unordered_map<std::string, int> HttpRequest::DEFAULT_HTML_TAG {
    {"/login.html", 1}, {"/register.html", 0}
};

void HttpRequest::Init() {
    state_ = REQUEST_LINE;
    method_ = path_ = version_ = body_ = "";
    header_.clear();
    post_.clear();
}

bool HttpRequest::ParseRequestLine(const std::string& line) {
    // GET /index.html HTTP/1.1
    std::regex patten("^([^ ]*) ([^ ]*) HTTP/([^ ]*)$");
    std::smatch match;
    if (std::regex_match(line, match, patten)) {
        method_ = match[1];
        path_ = match[2];
        version_ = match[3];
        state_ = HEADERS;
        return true;
    } 
    LOG_ERROR("RequestLine Error"); 
    return false;
}

void HttpRequest::ParseHeader(const std::string& line) {
    std::regex patten("^([^:]*): ?(.*)$");
    std::smatch match;
    if (std::regex_match(line, match, patten)) {
        header_[match[1]] = match[2];
    } else {
        state_ = BODY;
    }
}

void HttpRequest::ParseBody(const std::string& line) {
    body_ = line;
    ParsePost();    
    state_ = FINISH;
    LOG_DEBUG("Body: %s, len: %d", line.c_str(), line.size());
}

void HttpRequest::ParsePost() {
    if (method_ == "POST" && header_["Content-Type"] == "application/x-www-form-urlencoded") {
        ParseFromUrlEncoded();
        if (DEFAULT_HTML_TAG.count(path_)) { // 登录/注册
            int tag = DEFAULT_HTML_TAG.find(path_)->second;
            LOG_DEBUG("Tag: %d", tag);
            if (tag == 0 || tag == 1) {
                bool is_login = (tag == 1);
                if (UserVerify(post_["username"], post_["password"], is_login))
                    path_ = "/welcome.html";
                else
                    path_ = "/error.html";
            }
        }
    }
}

// 16进制转10进制
int HttpRequest::ConverHex(char ch) {
    if (ch >= 'A' && ch <= 'F')
        return ch - 'A' + 10;
    if (ch >= 'a' && ch <= 'f')
        return ch - 'a' + 10;
    return ch;
}

void HttpRequest::ParseFromUrlEncoded() {
    if (body_.size() == 0)
        return;
    
    std::string key, value;
    int num = 0;
    int n = body_.size();
    int i = 0, j = 0;

    while (i < n) {
        char ch = body_[i];
        // username=john%20doe&password=123456
        switch (ch) {
        case '=': // 获取键值对
            key = body_.substr(j, i - j); 
            j = i + 1;
            break;
        case '&': // 获取键值对
            value = body_.substr(j, i - j);
            j = i + 1;
            post_[key] = value;
            LOG_DEBUG("%s = %s", key.c_str(), value.c_str());
            break;
        case '+': // 替换为空格
            body_[i] = ' ';
            break;
        case '%':
            num = ConverHex(body_[i + 1]) * 16 + ConverHex(body_[i + 2]);
            body_[i + 2] = num % 10 + '0';
            body_[i + 1] = num / 10 + '0';
            i += 2;
            break;
        default:
            break;
        }
        i++;
    }
    assert(j <= i);
    if (post_.count(key) == 0 && j < i) {
        value = body_.substr(j, i - j);
        post_[key] = value;
    }
}

void HttpRequest::ParsePath() {
    if (path_ == "/") {
        path_ = "/index.html";
    } else {
        if (DEFAULT_HTML.find(path_) != DEFAULT_HTML.end())
            path_ += ".html";
    }
}

// 验证用户的用户名和密码
bool HttpRequest::UserVerify(const std::string& name, const std::string& pwd, bool is_login) {
    if (name == "" || pwd == "")
        return false;

    LOG_INFO("Verify name: %s pwd: %s", name.c_str(), pwd.c_str());
    MYSQL* sql;
    SqlConnectRAII(&sql, SqlConnectPool::GetInstance());
    assert(sql);

    bool flag = false;
    char order[256] = {0};
    MYSQL_RES* res = nullptr;      // 查询结果集

    if (!is_login) // 注册
        flag = true;
    snprintf(order, 256, "SELECT username, password FROM User WHERE username='%s' LIMIT 1", name.c_str());
    LOG_DEBUG("%s", order);

    // 查询用户信息
    if (mysql_query(sql, order)) {
        if (res)
            mysql_free_result(res); // 查询失败，释放结果集
        return false;
    }
    res = mysql_store_result(sql);   // 存储查询结果到res中

    // 处理查询结果
    while (MYSQL_ROW row = mysql_fetch_row(res)) {
        LOG_DEBUG("MYSQL ROW: %s %s", row[0], row[1]);
        std::string password = row[1];
        if (is_login) {
            if (pwd == password) {
                flag = true;
            } else {
                flag = false;
                LOG_INFO("pwd error!");
            }
        } else {
            flag = false;
            LOG_INFO("user used!");
        }
    }
    mysql_free_result(res);

    // 注册（用户名未被使用）
    if (!is_login && flag) {
        LOG_DEBUG("register");
        bzero(order, 256);
        snprintf(order, 256, "INSERT INTO User(username, password) VALUES('%s', '%s')", name.c_str(), pwd.c_str());
        LOG_DEBUG("%s", order);
        if (mysql_query(sql, order)) {
            LOG_ERROR("MySQL insert fail: Error: %s", mysql_error(sql));
            flag = false;
        } else {
            LOG_DEBUG("UserVerify success!");
            flag = true;
        }
    }
    return flag;
}

bool HttpRequest::Parse(Buffer& buff) {
    const char* END = "\r\n";
    if (buff.ReadableBytes() == 0)
        return false;
    
    while (buff.ReadableBytes() && state_ != FINISH) {
        // 找到buff中，首次出现"\r\n"的位置
        const char* line_end = std::search(buff.ReadBegin(), buff.WriteBeginConst(), END, END + 2);
        string line(buff.ReadBegin(), line_end);
        switch (state_) {
        case REQUEST_LINE:
            if (ParseRequestLine(line) == false)
                return false;
            ParsePath();
            break;
        case HEADERS:
            ParseHeader(line);
            if (buff.ReadableBytes() <= 2) // get请求，提前结束
                state_ = FINISH;
            break;
        case BODY:
            ParseBody(line);
            break;
        default:
            break;
        }
        if (line_end == buff.WriteBegin()) { // 读完了
            buff.RetrieveAll();
            break;
        }
        buff.RetrieveUntil(line_end + 2); // 跳过回车换行
    }
    LOG_DEBUG("[%s] [%s] [%s]", method_ .c_str(), path_.c_str(), version_.c_str());
    return true;
}

std::string HttpRequest::Method() const {
    return method_;
}

std::string HttpRequest::Path() const {
    return path_;
}

std::string& HttpRequest::Path() {
    return path_;
}

std::string HttpRequest::Version() const {
    return version_;
}

std::string HttpRequest::GetPost(const std::string& key) const {
    if (post_.count(key) == 1)
        // return post_[key]; post_有const属性，所以不能用[]
        return post_.find(key)->second;
    return "";
}

std::string HttpRequest::GetPost(const char* key) const {
    assert(key != nullptr);
    if (post_.count(key) == 1)
        return post_.find(key)->second;
    return "";
}

bool HttpRequest::IsKeepAlive() const {
    if (header_.count("Connect") == 1) 
        return header_.find("Connect")->second == "keep-alive" && version_ == "1.1";
    return false;
}
```

# HttpRequest 测试

**测试HttpRequest的，解析，注册，登录功能**

```C++
#include "../code/http/http_request.h"
#include <iostream>

void TestHttpRequest() {
    // 初始化日志系统
    Log* logger = Log::GetInstance();
    logger->Init(0, "./logs/", ".log", 1024);

    // 初始化测试
    HttpRequest request;

    // 初始化数据库连接池
    SqlConnectPool* conn_pool = SqlConnectPool::GetInstance();
    conn_pool->Init("localhost", 3306, "Tian", "123456", "web_server", 10);


    // 解析请求测试
    std::string http_request = "GET /index.html HTTP/1.1\r\n"
                               "Host: example.com\r\n"
                               "Connection: keep-alive\r\n"
                               "\r\n";
    Buffer buff;
    buff.Append(http_request);
    bool parseResult = request.Parse(buff);
    assert(parseResult);

    // 访问器方法测试
    assert(request.Method() == "GET");
    assert(request.Path() == "/index.html");
    assert(request.Version() == "1.1");

    // 模拟注册 POST 请求
    std::string register_request = "POST /register.html HTTP/1.1\r\n"
                                   "Host: example.com\r\n"
                                   "Content-Type: application/x-www-form-urlencoded\r\n"
                                   "Content-Length: 23\r\n"
                                   "\r\n"
                                   "username=test&password=123456";
    request.Init();
    buff.Append(register_request);
    parseResult = request.Parse(buff);
    assert(parseResult);

    assert(request.Method() == "POST");
    assert(request.Path() == "/welcome.html");
    assert(request.Version() == "1.1");
    assert(request.GetPost("username") == "test");
    assert(request.GetPost("password") == "123456");

    // Post 数据获取测试
    // 模拟登录 POST 请求
    std::string login_request = "POST /login HTTP/1.1\r\n"
                               "Content-Type: application/x-www-form-urlencoded\r\n"
                               "Content-Length: 19\r\n"
                               "\r\n"
                               "username=test&password=123456"
                               "\r\n";
    buff.Append(login_request);
    request.Init();

    parseResult = request.Parse(buff);
    assert(parseResult);

    assert(request.Method() == "POST");
    assert(request.Path() == "/welcome.html");
    assert(request.Version() == "1.1");
    assert(request.GetPost("username") == "test");
    assert(request.GetPost("password") == "123456");

    std::cout << "All tests passed!" << std::endl;
}

int main() {
    TestHttpRequest();
    return 0;
}
```

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
set(COMMON_SOURCES ../code/buffer/buffer.cc ../code/log/log.cc)
set(HTTP_SOURCE ../code/http/http_request.cc)
set(POOL_SOURCE ../code/pool/sql_connect_pool.cc)

# 查找 MySQL 库
find_package(PkgConfig REQUIRED)
pkg_check_modules(MYSQL REQUIRED mysqlclient)
# 包含 MySQL 头文件目录
include_directories(${MYSQL_INCLUDE_DIR})

# 添加可执行文件
add_executable(http_request_test http_request_test.cc ${COMMON_SOURCES} ${HTTP_SOURCE} ${POOL_SOURCE})
# 链接库
target_link_libraries(http_request_test ${MYSQL_LIBRARIES})
```

