对于一个服务器而言，不论是在调试中还是在运行中，都需要通过打日志的方式来记录程序的运行情况。

- **同步日志**：日志写入函数与工作线程串行执行，由于涉及到I/O操作，当单条日志比较大的时候，同步模式会阻塞整个处理流程，服务器所能处理的并发能力将有所下降，尤其是在峰值的时候，写日志可能成为系统的瓶颈。
- **异步日志**：将所写的日志内容先存入阻塞队列中，写线程从阻塞队列中取出内容，写入日志。

![](https://s2.loli.net/2025/01/24/9JXW5qw4jxvziZI.png)

# 日志的运行流程

1. **获取日志实例**：
   使用单例模式（具体采用局部静态变量的实现方法）来获取 `Log` 类的唯一实例，调用方式为 `Log::GetInstance()`。单例模式确保了在整个程序运行期间，`Log` 类只有一个实例存在，避免了多个实例可能带来的资源浪费和数据不一致问题。
2. **初始化日志系统**：
   通过获取的 `Log` 实例调用 `init()` 函数来完成日志系统的初始化工作。在初始化过程中，会根据设置的阻塞队列大小来决定采用同步日志还是异步日志。如果阻塞队列的大小大于 0，就会选择异步日志模式；如果等于 0，则选择同步日志模式。同时，会更新 `is_async` 变量来标记当前的日志模式。
3. **写入日志**：
   当需要记录日志时，通过 `Log` 实例调用 `Write()` 函数。在写入日志之前，会根据当前的时间信息创建一个新的日志文件，日志文件的命名规则是：前缀为当前时间，后缀为 `.log`。同时，会更新记录当前日期的 `today_` 变量和记录当前日志行数的 `line_count` 变量。
4. **判断日志写入方式**：
   在 `Write()` 函数内部，会根据 `is_async` 变量的值来决定具体的日志写入方式。如果 `is_async` 为 `true`，表示当前处于异步日志模式，工作线程会将需要写入的日志内容放入阻塞队列中，然后由专门的写线程从阻塞队列中取出数据并写入日志文件；如果 `is_async` 为 `false`，则表示处于同步日志模式，日志内容会直接写入到日志文件中。

# Log 成员变量

```C++
static const int Log_NAME_LENGTH = 256; // 最长文件名
static const int MAX_LINES = 50000;     // 最长日志条数

const char* path_;      // 路径名
const char* suffix_;    // 后缀名

bool is_open_;  // 是否开启
int level_;     // 日志等级
bool is_async_; // 是否开启异步日志

int today_;     // 当天日期
int line_count_;// 日志行数

Buffer buff_;   // 输出缓冲区
FILE* fp_;      // 文件指针

std::mutex mtx_;
std::unique_ptr<BlockQueue<string>> deque_; // 阻塞队列
std::unique_ptr<thread> write_thread_;		// 写线程
```

# 实现构造和析构

**构造函数**：初始化成员变量，默认不启用异步日志记录。

```C++
Log::Log() 
    : is_async_(false), today_(0),
      line_count_(0), fp_(nullptr),
      deque_(nullptr), write_thread_(nullptr) {}
```

**析构函数**：在对象销毁时，确保所有待写入的日志信息都被处理，关闭阻塞队列，等待写线程退出，最后关闭文件指针。

```C++
Log::~Log() {
    while (!deque_->empty())
        deque_->flush(); // 唤醒消费者，处理剩下数据
    deque_->close();
    write_thread_->join(); // 等待线程退出
    if (fp_) {
        std::lock_guard<std::mutex> locker(mtx_);
        Flush();
        fclose(fp_);
    }
}
```

# 实现 Init() 函数

- 初始化日志系统，阻塞队列，写线程，设置日志级别、日志文件路径和后缀等。
- 根据 `max_capacity` 的值决定是否启用异步日志记录。
- 生成日志文件名，并打开日志文件。如果文件打开失败，尝试创建目录并再次打开。

```C++
// 初始化
void Log::Init(int level, const char* path,
              const char* suffix, 
              int max_capacity) {
    is_open_ = true;
    level_ = level; 
    path_ = path;
    suffix_ = suffix;

    if (max_capacity) { // 异步
        is_async_ = true;
        if (!deque_) {
            std::unique_ptr<BlockQueue<string>> new_deque(new BlockQueue<string>);
            deque_ = std::move(new_deque); // 所有权转移
            std::unique_ptr<thread> new_thread(new thread(FLushLogThread));
            write_thread_ = std::move(new_thread);
        }
    } else { // 同步
        is_async_ = false;
    }

    line_count_ = 0;

    path_ = path;
    suffix_ = suffix;
    time_t timer = time(nullptr);
    struct tm* sys_time = localtime(&timer);
    struct tm t = *sys_time;
    char filename[Log_NAME_LENGTH] = {0};
    snprintf(filename, Log_NAME_LENGTH - 1, "%s%04d_%02d_%0d%s",
             path_, t.tm_year + 1900, t.tm_mon + 1, t.tm_mday, suffix_);
    today_ = t.tm_mday;

    {
        std::lock_guard<std::mutex> locker(mtx_);
        if (fp_) {
            Flush(); 
            fclose(fp_);
        }
        fp_ = fopen(filename, "a");
        if (fp_ == nullptr) {
            mkdir(path_, 0777); // 777最大权限
            fp_ = fopen(filename, "a");
        }
        assert(fp_ != nullptr);
    }
}
```

# 实现 Write() 函数

- 获取当前时间，判断是否需要创建新的日志文件（日期改变或日志行数达到最大值）。
- 格式化日志信息，包括时间戳、日志级别等。
- 根据是否启用异步日志记录，将日志信息写入阻塞队列或直接写入文件。

```C++
void Log::Write(int level, const char* format, ...) {
    struct timeval now = {0, 0};
    gettimeofday(&now, nullptr);
    time_t time_second =  now.tv_sec;
    struct tm* sys_time = localtime(&time_second);
    struct tm t = *sys_time;

    // 日期不对或行数满了
    if (today_ != t.tm_mday || (line_count_ && (line_count_ % MAX_LINES == 0))) {
        std::unique_lock<std::mutex> locker(mtx_);
        locker.unlock();

        char new_file[Log_NAME_LENGTH];
        char tail[36];
        snprintf(tail, 36, "%04d_%02d_%02d", t.tm_year + 1900, t.tm_mon + 1, t.tm_mday);

        if (today_ != t.tm_mday) {
            snprintf(new_file, Log_NAME_LENGTH - 72, "%s%s%s", path_, tail, suffix_);
            today_ = t.tm_mday;
        } else {
            int num = line_count_ / MAX_LINES;
            snprintf(new_file, Log_NAME_LENGTH, "%s%s-%d%s", path_, tail, num, suffix_);
        }

        {
            std::lock_guard<std::mutex> locker(mtx_);
            Flush();
            fclose(fp_);
            fp_ = fopen(new_file, "a");
            assert(fp_ != nullptr);
        }
    }

    {
        std::lock_guard<std::mutex> locker(mtx_);
        line_count_++;

        int n = snprintf(buff_.WriteBegin(), 128, "%d-%02d-%02d %02d:%02d:%02d.%06ld ",     
                         t.tm_year + 1900, t.tm_mon + 1, t.tm_mday, 
                         t.tm_hour, t.tm_min, t.tm_sec, now.tv_usec);
        buff_.HasWritten(n);
        AppendLogLevel(level);

        va_list vaList;
        va_start(vaList, format);
        int m = vsnprintf(buff_.WriteBegin(), buff_.WritableBytes(), format, vaList);
        va_end(vaList);
        buff_.HasWritten(m);
        buff_.Append("\n\0", 2);

        if (is_async_ && deque_) // 异步模式-生产者
            deque_->push_back(buff_.RetrieveAllAsString());
        else // 同步模式-直接写入
            fputs(buff_.ReadBegin(), fp_);    
        buff_.RetrieveAll(); 
    }
}
```

# 实现宏定义函数

- **简化日志类的使用**，方便日志的写入操作，根据日志级别判断是否需要写入日志。
- 如果日志级别小于等于当前的日志级别，则调用`Log::Write`函数写入日志，并调用`Log::Flush`函数刷新缓冲区。

```C++
// 小于等于当前level才输出
#define LOG_BASE(level, format, ...) \
    do { \
        Log* log = Log::GetInstance(); \
        if (log->IsOpen() && log->GetLevel() <= level) { \
            log->Write(level, format, ##__VA_ARGS__); \
            log->Flush(); \
        } \
    }while(0);

#define LOG_DEBUG(format, ...) do {LOG_BASE(0, format, ##__VA_ARGS__)} while(0);
#define LOG_INFO(format, ...) do {LOG_BASE(1, format, ##__VA_ARGS__)} while(0);
#define LOG_WARN(format, ...) do {LOG_BASE(2, format, ##__VA_ARGS__)} while(0);
#define LOG_ERROR(format, ...) do {LOG_BASE(3, format, ##__VA_ARGS__)} while(0);
#define LOG_FATAL(format, ...) do {LOG_BASE(4, format, ##__VA_ARGS__)} while(0);
```

# Log 代码

**log.h**

```C++
#ifndef Log_H
#define Log_H

#include <sys/stat.h> // mkdir
#include <sys/time.h> // gettimeofday
#include <cstdio>   // FILE
#include <cstdarg>  // va_start
#include <ctime>
#include <cassert>
#include <string>
#include <utility>  // move
#include <memory>   // unique_ptr
#include <thread>

#include "blockqueue.h"
#include "../buffer/buffer.h"

using std::string;
using std::thread;

class Log {
public:
    void Init(int level, const char* path = "./log",
              const char* suffix = ".log", 
              int max_capacity = 1024);    

    static Log* GetInstance();
    static void FLushLogThread();

    void Flush();
    void Write(int level, const char* format, ...);

    int GetLevel();
    void SetLevel(int level);
    bool IsOpen();

private:
    Log();
    ~Log();

    void AppendLogLevel(int level);
    void AsyncWrite();

    static const int Log_NAME_LENGTH = 256; // 最长文件名
    static const int MAX_LINES = 50000;     // 最长日志条数

    const char* path_;      // 路径名
    const char* suffix_;    // 后缀名

    bool is_open_;  // 是否开启
    int level_;     // 日志等级
    bool is_async_; // 是否开启异步日志

    int today_;     // 当天日期
    int line_count_;// 日志行数

    Buffer buff_;   // 输出缓冲区
    FILE* fp_;      // 文件指针

    std::mutex mtx_;
    std::unique_ptr<BlockQueue<string>> deque_;
    std::unique_ptr<thread> write_thread_;
};

// 小于等于当前level才输出
#define LOG_BASE(level, format, ...) \
    do { \
        Log* log = Log::GetInstance(); \
        if (log->IsOpen() && log->GetLevel() <= level) { \
            log->Write(level, format, ##__VA_ARGS__); \
            log->Flush(); \
        } \
    }while(0);

#define LOG_DEBUG(format, ...) do {LOG_BASE(0, format, ##__VA_ARGS__)} while(0);
#define LOG_INFO(format, ...) do {LOG_BASE(1, format, ##__VA_ARGS__)} while(0);
#define LOG_WARN(format, ...) do {LOG_BASE(2, format, ##__VA_ARGS__)} while(0);
#define LOG_ERROR(format, ...) do {LOG_BASE(3, format, ##__VA_ARGS__)} while(0);
#define LOG_FATAL(format, ...) do {LOG_BASE(4, format, ##__VA_ARGS__)} while(0);

#endif // Log_H
```

**log.cc**

```C++
#include "log.h"

Log::Log() 
    : is_async_(false), today_(0),
      line_count_(0), fp_(nullptr),
      deque_(nullptr), write_thread_(nullptr) {}

Log::~Log() {
    while (!deque_->empty())
        deque_->flush(); // 唤醒消费者，处理剩下数据
    deque_->close();
    write_thread_->join(); // 等待线程退出
    if (fp_) {
        std::lock_guard<std::mutex> locker(mtx_);
        Flush();
        fclose(fp_);
    }
}

// 初始化
void Log::Init(int level, const char* path,
              const char* suffix, 
              int max_capacity) {
    is_open_ = true;
    level_ = level; 
    path_ = path;
    suffix_ = suffix;

    if (max_capacity) { // 异步
        is_async_ = true;
        if (!deque_) {
            std::unique_ptr<BlockQueue<string>> new_deque(new BlockQueue<string>);
            deque_ = std::move(new_deque); // 所有权转移
            std::unique_ptr<thread> new_thread(new thread(FLushLogThread));
            write_thread_ = std::move(new_thread);
        }
    } else { // 同步
        is_async_ = false;
    }

    line_count_ = 0;

    path_ = path;
    suffix_ = suffix;
    time_t timer = time(nullptr);
    struct tm* sys_time = localtime(&timer);
    struct tm t = *sys_time;
    char filename[Log_NAME_LENGTH] = {0};
    snprintf(filename, Log_NAME_LENGTH - 1, "%s%04d_%02d_%0d%s",
             path_, t.tm_year + 1900, t.tm_mon + 1, t.tm_mday, suffix_);
    today_ = t.tm_mday;

    {
        std::lock_guard<std::mutex> locker(mtx_);
        if (fp_) {
            Flush(); 
            fclose(fp_);
        }
        fp_ = fopen(filename, "a");
        if (fp_ == nullptr) {
            mkdir(path_, 0777); // 777最大权限
            fp_ = fopen(filename, "a");
        }
        assert(fp_ != nullptr);
    }
}

void Log::AppendLogLevel(int level) {
    const char* level_title[] = {"[DEBUG]: ", "[INFO] : ", "[WARN] : ",
                                "[ERROR]: ", "[FATAL]: "};
    int valid_level = (level >= 0 && level <= 4) ? level : 1;
    buff_.Append(level_title[valid_level], 9);
}

void Log::Write(int level, const char* format, ...) {
    struct timeval now = {0, 0};
    gettimeofday(&now, nullptr);
    time_t time_second =  now.tv_sec;
    struct tm* sys_time = localtime(&time_second);
    struct tm t = *sys_time;

    // 日期不对或行数满了
    if (today_ != t.tm_mday || (line_count_ && (line_count_ % MAX_LINES == 0))) {
        std::unique_lock<std::mutex> locker(mtx_);
        locker.unlock();

        char new_file[Log_NAME_LENGTH];
        char tail[36];
        snprintf(tail, 36, "%04d_%02d_%02d", t.tm_year + 1900, t.tm_mon + 1, t.tm_mday);

        if (today_ != t.tm_mday) {
            snprintf(new_file, Log_NAME_LENGTH - 72, "%s%s%s", path_, tail, suffix_);
            today_ = t.tm_mday;
        } else {
            int num = line_count_ / MAX_LINES;
            snprintf(new_file, Log_NAME_LENGTH, "%s%s-%d%s", path_, tail, num, suffix_);
        }

        {
            std::lock_guard<std::mutex> locker(mtx_);
            Flush();
            fclose(fp_);
            fp_ = fopen(new_file, "a");
            assert(fp_ != nullptr);
        }
    }

    {
        std::lock_guard<std::mutex> locker(mtx_);
        line_count_++;

        int n = snprintf(buff_.WriteBegin(), 128, "%d-%02d-%02d %02d:%02d:%02d.%06ld ",     
                         t.tm_year + 1900, t.tm_mon + 1, t.tm_mday, 
                         t.tm_hour, t.tm_min, t.tm_sec, now.tv_usec);
        buff_.HasWritten(n);
        AppendLogLevel(level);

        va_list vaList;
        va_start(vaList, format);
        int m = vsnprintf(buff_.WriteBegin(), buff_.WritableBytes(), format, vaList);
        va_end(vaList);
        buff_.HasWritten(m);
        buff_.Append("\n\0", 2);

        if (is_async_ && deque_) // 异步模式-生产者
            deque_->push_back(buff_.RetrieveAllAsString());
        else // 同步模式-直接写入
            fputs(buff_.ReadBegin(), fp_);    
        buff_.RetrieveAll(); 
    }
}

// 单例模式之饿汉模式
Log* Log::GetInstance() {
    // 静态局部变量的初始化是线程安全的
    static Log log;
    return &log;
}

// 异步日志的写线程函数
void Log::FLushLogThread() {
    Log::GetInstance()->AsyncWrite();
}

// 写线程真正的执行函数
void Log::AsyncWrite() {
    string str = "";
    while (deque_->pop(str)) { // 异步模式-消费者
        std::lock_guard<std::mutex> locker(mtx_);
        fputs(str.c_str(), fp_);
    }
}

// 唤醒消费者，开始写日志
void Log::Flush() {
    if (is_async_)
        deque_->flush();
    fflush(fp_);
}

int Log::GetLevel() {
    std::lock_guard<std::mutex> lock(mtx_);
    return level_;
}

void Log::SetLevel(int level) {
    std::lock_guard<std::mutex> lock(mtx_);
    level_ = level;
}

bool Log::IsOpen() {
    return is_open_;
}
```

# Log 测试

```
#include "../code/log/log.h"
#include <iostream>

int main() {
    // 获取 Log 类的单例实例
    Log* logger = Log::GetInstance();

    // 初始化日志系统
    logger->Init(0, "./logs/", ".log", 1024);

    // 输出不同级别的日志信息
    LOG_DEBUG("This is a debug message.");
    LOG_INFO("This is an info message.");
    LOG_WARN("This is a warning message.");
    LOG_ERROR("This is an error message.");
    LOG_FATAL("This is anfatal message.");

    // 输出日志级别
    std::cout << "Current log level: " << logger->GetLevel() << std::endl;

    // 修改日志级别
    logger->SetLevel(2);
    std::cout << "New log level: " << logger->GetLevel() << std::endl;

    // 再次输出不同级别的日志信息
    LOG_DEBUG("This debug message should not be logged.");
    LOG_INFO("This info message should not be logged.");
    LOG_WARN("This is a new warning message.");
    LOG_ERROR("This is a new error message.");
    LOG_FATAL("This is a new fatal message.");


    // 可变参数
    logger->SetLevel(0);
    LOG_DEBUG("%s %d %d", "log info", 123, 666);

    // 大量日志
    for (int i = 0; i < 100000; ++i)
        LOG_DEBUG("This is the %d-th error message.", i);

    return 0;
}
```

**CMakeLists.txt**

```cmake
cmake_minimum_required(VERSION 3.10)
project(buffer_unit_test)

# 设置 C++ 标准和编译器选项
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra")


# 添加可执行文件
add_executable(log_unit_test log_unit_test.cc ../code/log/log.cc ../code/buffer/buffer.cc)
```