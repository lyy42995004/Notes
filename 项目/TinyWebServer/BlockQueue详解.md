# 阻塞队列是什么？

![](https://s2.loli.net/2025/01/24/rRUVvHSlsQAak2Z.jpg)

**阻塞队列是一种线程安全的数据结构，支持多线程环境中的生产者-消费者模型**。其核心特点在于，当队列为空时，消费者线程会进入阻塞状态，直到有新的数据可供消费；而当队列已满时，生产者线程会被阻塞，直至队列中有空闲空间可供使用。

- **线程安全**：借助同步机制，有效避免了多个线程同时操作队列时可能出现的数据竞争问题，确保数据的一致性和完整性。

- **容量限制**：可以根据实际需求灵活设置队列的容量上限，当队列达到最大容量时，生产者线程会被阻塞，避免数据溢出。
- **阻塞操作**：当队列为空时，消费者线程会自动等待；当队列满时，生产者线程也会进入等待状态，实现了线程间的协调与同步。


# 异步日志系统为什么需要阻塞队列？

- **解耦生产消费**：将日志信息的产生和存储过程进行分离，应用程序只需将日志快速放入队列，无需等待写入操作完成，从而提高了系统的可维护性。

- **平衡速度差异**：有效应对日志生产速度不稳定和消费速度受存储设备性能限制的问题，队列能够缓存多余的日志信息，实现动态平衡，确保系统的稳定运行。
- **提升并发性能**：支持多线程协作，生产者和消费者线程可以同时工作，充分利用多核处理器的性能优势，同时队列的同步机制避免了线程竞争，提高了系统的并发处理能力。
- **保证日志顺序**：阻塞队列的先进先出特性确保日志按产生顺序进行存储，便于后续的问题排查和分析。

# BlockQueue 成员变量

```C++
bool is_close;          // 是否关闭
size_t capacity_;       // 容量
std::deque<T> deque_;   // 双向队列 

std::mutex mtx_;      // 锁
std::condition_variable condition_producer; // 生产者条件变量
std::condition_variable condition_consumer; // 消费者条件变量
```

# 实现 push() 函数

获取互斥锁`mtx_`保证线程安全；检查队列是否已满，满了就等待；向队列添加元素；通知一个等待的消费者线程。

```C++
void push_back(const T& item);
```

向阻塞队列的尾部添加一个元素。

```C++
template<class T>
void BlockQueue<T>::push_back(const T& item) {
    std::unique_lock<std::mutex> locker(mtx_);
    // 队列满了，暂停生产
    while (deque_.size() >= capacity_)
        condition_producer.wait(locker); // 防止虚假唤醒
    deque_.push_back(item);
    condition_consumer.notify_one();
}
```

向阻塞队列的头部添加一个元素。

```C++
template<class T>
void BlockQueue<T>::push_front(const T& item) {
    std::unique_lock<std::mutex> locker(mtx_);
    while (deque_.size() >= capacity_)
        condition_producer.wait(locker);
    deque_.push_front(item);
    condition_consumer.notify_one();
}
```

# 实现 pop() 函数

获取互斥锁`mtx_`；检查队列是否为空，如果为空且队列未关闭，就等待；如果队列关闭，返回`false`；取出队列头部元素，存储到`item`中，并移除队列头部元素；通知一个等待的生产者线程，队列中有空间了。

```C++
bool pop(T& item);
```

从阻塞队列的头部取出一个元素。

```C++
template<class T>
bool BlockQueue<T>::pop(T& item) {
    std::unique_lock<std::mutex> locker(mtx_);
    // 队列空了，暂停消费
    while (deque_.empty()) {
        if (is_close)
            return false;
        condition_consumer.wait(locker);
    }
    item = deque_.front();
    deque_.pop_front();
    condition_producer.notify_one();
    return true;
}
```

```C++
bool pop(T& item, int timeout);
```

从阻塞队列的头部取出一个元素，但是有超时机制。

```C++
template<class T>
bool BlockQueue<T>::pop(T& item, int timeout) {
    std::unique_lock<std::mutex> locker(mtx_);
    const std::cv_status TIMEOUT_STATUS = std::cv_status::timeout;
    while (deque_.empty()) {
        if (is_close)
            return false;
        if (condition_consumer.wait_for(locker, 
            std::chrono::seconds(timeout)) == TIMEOUT_STATUS)
            return false;
    }
    item = deque_.front();
    deque_.pop_front();
    condition_producer.notify_one();
    return true;
}
```

# 实现 close() 函数

获取互斥锁`mtx_`，清空队列，设置关闭标志`is_close`为`true`；通知所有等待的生产者和消费者线程。

```C++
void close();
```

关闭阻塞队列，释放所有等待的生产者和消费者线程。

```C++
template<class T>
void BlockQueue<T>::close() {
    {
        std::lock_guard<std::mutex> locker(mtx_);
        deque_.clear();
        is_close = true;
    }
    condition_producer.notify_all();
    condition_consumer.notify_all();
}
```

# BlockQueue 代码

模板的定义和实现要放在同一个头文件中，因为模板的代码需要在编译时实例化。

**block**

```C++
#ifndef BLOCKQUEUE_H
#define BLOCKQUEUE_H

#include <iostream>
#include <deque>
#include <mutex>
#include <condition_variable>
#include <chrono>
#include <assert.h>

template<class T>
class BlockQueue {
public:
    BlockQueue(size_t max_size = 1000);
    ~BlockQueue();

    bool empty();
    bool full();
    void clear();
    size_t size();
    size_t capacity();

    void push_front(const T& item);
    void push_back(const T& item);
    bool pop(T& item);
    bool pop(T& item, int timeout);

    T front();
    T back();
    void flush();
    void close();

private:
    bool is_close;          // 是否关闭
    size_t capacity_;       // 容量
    std::deque<T> deque_;   // 双向队列 

    std::mutex mtx_;      // 锁
    std::condition_variable condition_producer; // 生产者条件变量
    std::condition_variable condition_consumer; // 消费者条件变量
};

// 模板的定义和实现要放在同一个头文件中
// 因为模板的代码需要在编译时实例化

template<class T>
BlockQueue<T>::BlockQueue(size_t max_size) : capacity_(max_size) {
    assert(max_size > 0);
    is_close = false;
}

template<class T>
BlockQueue<T>::~BlockQueue() {
    close();
}   

template<class T>
void BlockQueue<T>::push_back(const T& item) {
    std::unique_lock<std::mutex> locker(mtx_);
    // 队列满了，暂停生产
    while (deque_.size() >= capacity_)
        condition_producer.wait(locker); // 防止虚假唤醒
    deque_.push_back(item);
    condition_consumer.notify_one();
}

template<class T>
void BlockQueue<T>::push_front(const T& item) {
    std::unique_lock<std::mutex> locker(mtx_);
    while (deque_.size() >= capacity_)
        condition_producer.wait(locker);
    deque_.push_front(item);
    condition_consumer.notify_one();
}

template<class T>
bool BlockQueue<T>::pop(T& item) {
    std::unique_lock<std::mutex> locker(mtx_);
    // 队列空了，暂停消费
    while (deque_.empty()) {
        if (is_close)
            return false;
        condition_consumer.wait(locker);
    }
    item = deque_.front();
    deque_.pop_front();
    condition_producer.notify_one();
    return true;
}

template<class T>
bool BlockQueue<T>::pop(T& item, int timeout) {
    std::unique_lock<std::mutex> locker(mtx_);
    const std::cv_status TIMEOUT_STATUS = std::cv_status::timeout;
    while (deque_.empty()) {
        if (is_close)
            return false;
        if (condition_consumer.wait_for(locker, 
            std::chrono::seconds(timeout)) == TIMEOUT_STATUS)
            return false;
    }
    item = deque_.front();
    deque_.pop_front();
    condition_producer.notify_one();
    return true;
}

// 关闭阻塞队列，唤醒所有生产者和消费者
template<class T>
void BlockQueue<T>::close() {
    {
        std::lock_guard<std::mutex> locker(mtx_);
        deque_.clear();
        is_close = true;
    }
    condition_producer.notify_all();
    condition_consumer.notify_all();
}

// 唤醒消费者
template<class T>
void BlockQueue<T>::flush() {
    condition_consumer.notify_one();
}

template<class T>
T BlockQueue<T>::front() {
    std::lock_guard<std::mutex> locker(mtx_);
    return deque_.front();
}

template<class T>
T BlockQueue<T>::back() {
    std::lock_guard<std::mutex> locker(mtx_);
    return deque_.back();
}

template<class T>
bool BlockQueue<T>::empty() {
    std::lock_guard<std::mutex> locker(mtx_);
    return deque_.empty();
}

template<class T>
bool BlockQueue<T>::full() {
    std::lock_guard<std::mutex> locker(mtx_);
    return deque_.size() >= capacity_;
}

template<class T>
void BlockQueue<T>::clear() {
    std::lock_guard<std::mutex> locker(mtx_);
    deque_.clear();
}

template<class T>
size_t BlockQueue<T>::size() {
    std::lock_guard<std::mutex> locker(mtx_);
    return deque_.size();
}

template<class T>
size_t BlockQueue<T>::capacity() {
    std::lock_guard<std::mutex> locker(mtx_);
    return capacity_;
}

#endif // BLOCKQUEUE_H
```

# BlockQueue 测试

利用***Google Test***对`BlockQueue`类进行单元测试，**测试对`Buffer`的基本功能，延时出队，多线程下的生产者-消费者模型，关闭操作。**

```C++
#include "../code/log/blockqueue.h"
#include <thread>
#include <chrono>
#include <gtest/gtest.h>

// 测试 BlockQueue 的基本功能
TEST(BlockQueueTest, TestBasicFunctionality) {
    BlockQueue<int> queue(5);
    EXPECT_TRUE(queue.empty());
    EXPECT_EQ(queue.capacity(), 5);
    
    for (int i = 0; i < 5; ++i)
        queue.push_back(i);
    EXPECT_TRUE(queue.full());
    EXPECT_EQ(queue.size(), 5);

    int item;
    EXPECT_TRUE(queue.pop(item));
    EXPECT_EQ(item, 0);
    EXPECT_EQ(queue.front(), 1);
    EXPECT_EQ(queue.back(), 4);

    queue.push_front(5);
    EXPECT_EQ(queue.front(), 5);

    queue.clear();
    EXPECT_TRUE(queue.empty());
}

// 测试带有超时的 pop 操作
TEST(BlockQueueTest, TestPopWithTimeout) {
    BlockQueue<int> queue(5);
    int item;
    // 等待失败，花费1s
    EXPECT_FALSE(queue.pop(item, 1));

    queue.push_back(2);
    // 等待成功，不耗时
    EXPECT_TRUE(queue.pop(item, 1));
    EXPECT_EQ(item, 2);
}

// 测试多线程下的 生产者-消费者模式
TEST(BlockQueueTest, TestProducerConsumer) {
    BlockQueue<int> queue(5);

    std::thread producer([&queue]() {
        for (int i = 0; i < 10; ++i) {
            queue.push_back(i);
            // 模拟生产过程
            std::this_thread::sleep_for(std::chrono::milliseconds(200));
        }
    });

    std::thread consumer([&queue]() {
        int item;
        for (int i = 0; i < 10; ++i) {
            if (queue.pop(item))
                EXPECT_EQ(item, i);
            else
                FAIL() << "Failed to consume an item.";
            // 模拟消费过程
            std::this_thread::sleep_for(std::chrono::milliseconds(100));
        }
    });

    producer.join();
    consumer.join();
}

// 测试关闭操作
TEST(BlockQueueTest, TestClose) {
    BlockQueue<int> queue(5);

    std::thread producer([&queue]() {
        for (int i = 0; i < 10; ++i) {
            queue.push_back(i);
            std::this_thread::sleep_for(std::chrono::milliseconds(100));
        }
        queue.close();
    });

    std::thread consumer([&queue]() {
        int item;
        while (queue.pop(item))
            continue;
        EXPECT_TRUE(queue.empty());
    });

    producer.join();
    consumer.join();
}

int main(int argc, char* argv[]) {
    ::testing::InitGoogleTest(&argc, argv);
    return RUN_ALL_TESTS(); 
}
```

**CMakeLists.txt**

```cmake
cmake_minimum_required(VERSION 3.10)
project(tests)

# 设置 C++ 标准和编译器选项
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra")

# 查找 Google Test 包
find_package(GTest REQUIRED)
# 包含 Google Test 头文件目录
include_directories(${GTEST_INCLUDE_DIRS})
# 添加可执行文件
add_executable(blockqueue_unit_test blockqueue_unit_test.cc)
# 链接 Google Test 库
target_link_libraries(blockqueue_unit_test ${GTEST_LIBRARIES} pthread)
# 启用测试
enable_testing()
# 添加测试
add_test(NAME blockqueue_unit_test COMMAND blockqueue_unit_test)
```
