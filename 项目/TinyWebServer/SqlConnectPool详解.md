# 数据库连接池是什么？

前面实现了线程池，而数据库连接池的思想与线程池类似，都是将数据库连接或线程并存储在一个 “池”（容器）中，便于后续使用。

数据库连接池是一种数据库连接管理技术，它预先创建一定数量的数据库连接并存储在一个 “池” 中。当应用程序需要与数据库进行交互时，它不是直接创建新的数据库连接，而是从连接池中获取一个已经建立好的连接；当应用程序使用完连接后，将连接归还给连接池，而不是直接关闭连接。这样，连接可以被重复使用，避免了频繁创建和销毁连接带来的开销。

**数据库连接池的作用**

1. **减少资源消耗**：创建和销毁数据库连接是一个相对昂贵的操作，涉及到网络通信、身份验证、资源分配等多个步骤。使用连接池可以复用已经创建好的连接，减少了频繁创建和销毁连接所带来的系统资源消耗，提高了系统的性能。
2. **提高响应速度**：由于连接池中的连接已经预先创建好，应用程序可以直接从连接池中获取连接，无需等待新连接的创建过程，从而显著提高了应用程序的响应速度。
3. **统一管理连接**：连接池可以对连接进行统一管理，可以有效地控制数据库连接的使用，避免因连接过多导致数据库服务器资源耗尽。
4. **增强系统的稳定性**：连接池可以监控连接的状态，当连接出现异常时，可以及时进行处理，如重新建立连接等。这有助于增强系统的稳定性，减少因连接问题导致的系统故障。

# Web Server 中为什么需要数据库连接池？

在 Web Server 中，通常会面临大量的并发请求，每个请求可能都需要与数据库进行交互。如果没有数据库连接池，每次请求都要创建一个新的数据库连接，会导致以下问题：

1. **性能瓶颈**：频繁创建和销毁数据库连接会消耗大量的系统资源，导致 Web Server 的性能下降，响应时间变长。
2. **资源耗尽**：如果并发请求过多，不断创建新的连接可能会耗尽数据库服务器和 Web Server 的资源，导致系统崩溃。
3. **管理困难**：大量的连接会增加管理的难度，例如难以控制连接的数量和状态，容易出现连接泄漏等问题。

# SqlConnectPool 成员变量

```C++
int max_connect_;
std::queue<MYSQL*> connect_queue_;
std::mutex mtx_;
sem_t sem_id_;
```

- `connect_queue_`：使用队列存储数据库连接。
- `sem_id_`：信号量，用于控制连接的获取和释放，确保连接的数量不超过最大连接数。

# 实现 Init() 函数

- 遍历指定的连接数量，使用 `mysql_init` 初始化 MySQL 连接对象，使用 `mysql_real_connect` 建立与数据库的连接。
- 如果连接成功，将连接对象存入连接队列，并增加最大连接数。
- 最后使用 `sem_init` 初始化信号量，信号量的初始值为最大连接数。

```C++
void SqlConnectPool::Init(const char* host, uint16_t port, 
                          const char* user, const char* pwd,
                          const char* db_name, int connect_size) {
    assert(connect_size > 0);
    for (int i = 0; i < connect_size; ++i) {
        MYSQL* connect = nullptr;
        connect = mysql_init(nullptr);
        if (!connect) {
            LOG_ERROR("MySql init fail");
            assert(connect);
        }
        if (!mysql_real_connect(connect, host, user, pwd, db_name, port, nullptr, 0)) {
            LOG_ERROR("MySql failed to connect to database: Error: %s", mysql_error(connect));
        } else {
            connect_queue_.emplace(connect);
            max_connect_++;
        }
    }
    assert(max_connect_ > 0);
    // 线程级信号量
    sem_init(&sem_id_, 0, max_connect_);
}
```

# 实现 ClosePool() 函数

- 使用 `std::lock_guard` 加锁，确保多线程环境下对连接队列的安全访问。
- 遍历连接队列，使用 `mysql_close` 关闭所有的数据库连接，并调用 `mysql_library_end` 结束 MySQL 库的使用。

```C++
void SqlConnectPool::ClosePool() {
    std::lock_guard<std::mutex> locker(mtx_);
    while (!connect_queue_.empty()) {
        MYSQL* connect = connect_queue_.front();
        connect_queue_.pop();
        mysql_close(connect);
    }
    mysql_library_end();
}

```

# SqlConnectRAII 类

在构造函数中从连接池获取一个数据库连接，在析构函数中自动将连接放回连接池，确保连接的正确释放，避免内存泄漏。

```C++
// 构造时初始化，析构时释放
class SqlConnectRAII {
public:
    SqlConnectRAII(MYSQL** sql, SqlConnectPool* pool) {
        assert(pool);
        *sql = pool->GetConnect();
        sql_ = *sql;
        pool_ = pool;
    }

    ~SqlConnectRAII() {
        if (sql_)
            pool_->FreeConnect(sql_);
    }

private:
    MYSQL* sql_;
    SqlConnectPool* pool_;
};
```

# SqlConnectPool 代码

**sql_connect_pool.h**

```C++
#ifndef SQL_CONNECT_POOL_H
#define SQL_CONNECT_POOL_H

#include <string>
#include <queue>
#include <mutex>
#include <thread>
#include <cassert>
#include <semaphore.h>
#include <mysql/mysql.h>
#include "../log/log.h"

class SqlConnectPool {
public:
    static SqlConnectPool* GetInstance();

    void Init(const char* host, uint16_t port, 
              const char* user, const char* pwd,
              const char* db_name, int connect_size = 10);
    void ClosePool();

    MYSQL* GetConnect();
    void FreeConnect(MYSQL* connect);
    int GetFreeConnectCount();

private:
    SqlConnectPool() = default;
    ~SqlConnectPool() { ClosePool(); }

    int max_connect_;
    std::queue<MYSQL*> connect_queue_;
    std::mutex mtx_;
    sem_t sem_id_;
};

// 构造时初始化，析构时释放
class SqlConnectRAII {
public:
    SqlConnectRAII(MYSQL** sql, SqlConnectPool* pool) {
        assert(pool);
        *sql = pool->GetConnect();
        sql_ = *sql;
        pool_ = pool;
    }

    ~SqlConnectRAII() {
        if (sql_)
            pool_->FreeConnect(sql_);
    }

private:
    MYSQL* sql_;
    SqlConnectPool* pool_;
};

#endif // SQL_CONNECT_POOL_H
```

**sql_connect_pool.cc**

```C++
#include "sql_connect_pool.h"

SqlConnectPool* SqlConnectPool::GetInstance() {
    static SqlConnectPool pool;
    return &pool;
}


void SqlConnectPool::Init(const char* host, uint16_t port, 
                          const char* user, const char* pwd,
                          const char* db_name, int connect_size) {
    assert(connect_size > 0);
    for (int i = 0; i < connect_size; ++i) {
        MYSQL* connect = nullptr;
        connect = mysql_init(nullptr);
        if (!connect) {
            LOG_ERROR("MySql init fail");
            assert(connect);
        }
        if (!mysql_real_connect(connect, host, user, pwd, db_name, port, nullptr, 0)) {
            LOG_ERROR("MySql failed to connect to database: Error: %s", mysql_error(connect));
        } else {
            connect_queue_.emplace(connect);
            max_connect_++;
        }
    }
    assert(max_connect_ > 0);
    // 线程级信号量
    sem_init(&sem_id_, 0, max_connect_);
}

void SqlConnectPool::ClosePool() {
    std::lock_guard<std::mutex> locker(mtx_);
    while (!connect_queue_.empty()) {
        MYSQL* connect = connect_queue_.front();
        connect_queue_.pop();
        mysql_close(connect);
    }
    mysql_library_end();
}

MYSQL* SqlConnectPool::GetConnect() {
    MYSQL* connect = nullptr;
    if (connect_queue_.empty()) {
        LOG_WARN("SqlConnectPool busy!");
        return nullptr;
    } 
    sem_wait(&sem_id_); // -1
    std::lock_guard<std::mutex> locker(mtx_);
    connect = connect_queue_.front();
    connect_queue_.pop();
    return connect;
}

// 存入连接池
void SqlConnectPool::FreeConnect(MYSQL* connect) {
    assert(connect);
    std::lock_guard<std::mutex> locker(mtx_);
    connect_queue_.push(connect);
    sem_post(&sem_id_); // +1
}

int SqlConnectPool::GetFreeConnectCount() {
    std::lock_guard<std::mutex> locker(mtx_);
    return connect_queue_.size();
}
```

# SqlConnectPool 测试

**测试SqlConnectPool中MYSQL的初始化，连接，以及查询语句，以及SqlConnRAII 类**

```C++
#include "../code/pool/sql_connect_pool.h"
#include <iostream>

// 测试 SqlConnectPool 类的功能
void TestSqlConnectPool() {
    // 初始化日志系统
    Log* logger = Log::GetInstance();
    logger->Init(0, "./logs/", ".log", 1024);

    // 获取 SqlConnectPool 的单例实例
    SqlConnectPool* pool = SqlConnectPool::GetInstance();

    // 初始化连接池
    const char* host = "localhost";
    uint16_t port = 3306;
    const char* user = "Tian";
    const char* pwd = "123456";
    const char* db_name = "web_server";
    int connect_size = 10;
    pool->Init(host, port, user, pwd, db_name, connect_size);

    // 测试获取连接
    MYSQL* conn = pool->GetConnect();
    assert(conn != nullptr);  // 确保获取到的连接不为空

    // 执行简单的 SQL 查询
    if (mysql_query(conn, "select 1")) {
        std::cerr << "query failed: " << mysql_error(conn) << std::endl;
    } else {
        MYSQL_RES* result = mysql_store_result(conn);
        if (result) {
            MYSQL_ROW row = mysql_fetch_row(result);
            if (row) {
                std::cout << "Query result: " << row[0] << std::endl;
            }
            mysql_free_result(result);
        }
    }

    // 测试释放连接
    pool->FreeConnect(conn);

    // 测试获取空闲连接数量
    int freeCount = pool->GetFreeConnectCount();
    std::cout << "Free connection count: " << freeCount << std::endl;

    // 测试 SqlConnRAII 类
    {
        MYSQL* sql = nullptr;
        SqlConnectRAII raii(&sql, pool);
        assert(sql != nullptr);  // 确保通过 RAII 获取到的连接不为空
    }  // 离开作用域，自动释放连接

    // 关闭连接池
    pool->ClosePool();
}

int main() {
    TestSqlConnectPool();
    return 0;
}
```

**CMakeLists.txt**

```cmake
cmake_minimum_required(VERSION 3.10)
project(buffer_unit_test)

# 设置 C++ 标准和编译器选项
# set(CMAKE_BUILD_TYPE Debug)
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra")

# 定义公共源文件和特定文件
set(COMMON_SOURCES ../code/buffer/buffer.cc ../code/log/log.cc)
set(POOL_SOURCE ../code/pool/sql_connect_pool.cc)

# 查找 MySQL 库
find_package(PkgConfig REQUIRED)
pkg_check_modules(MYSQL REQUIRED mysqlclient)
# 包含 MySQL 头文件目录
include_directories(${MYSQL_INCLUDE_DIR})

# 添加可执行文件
add_executable(sql_connect_pool_test sql_connect_pool_test.cc ${COMMON_SOURCES} ${POOL_SOURCE})

# 链接库
target_link_libraries(sql_connect_pool_test ${MYSQL_LIBRARIES})
```

