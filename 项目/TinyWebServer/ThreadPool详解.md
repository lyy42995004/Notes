# 线程池是什么？

线程池是一种多线程处理形式，它预先创建一定数量的线程并将它们存储在一个 “池” 中。当有任务提交时，线程池会从池中取出一个空闲线程来执行该任务。如果池中没有空闲线程，任务可能会被放入一个队列中等待，直到有线程完成当前任务并变为空闲状态。当所有任务都完成后，线程池中的线程可以继续保持活跃，等待新的任务到来，或者根据配置进行销毁。

**线程池的作用**

1. **减少线程创建和销毁的开销**：线程的创建和销毁是比较昂贵的操作，涉及到操作系统的资源分配和回收。使用线程池可以避免频繁地创建和销毁线程，复用已经创建好的线程，从而提高系统的性能和效率。
2. **控制并发线程数量**：通过线程池可以限制系统中同时运行的线程数量，避免因创建过多线程导致系统资源耗尽，如 CPU 过度使用、内存溢出等问题。这有助于保持系统的稳定性和可靠性。
3. **提高响应速度**：由于线程已经预先创建好，当有任务到来时，可以立即分配线程执行，减少了线程创建的时间开销，从而提高了系统的响应速度。

# Web Server为什么需要线程池？

1. **高并发处理能力**：Web Server 通常需要处理大量的并发请求。如果为每个请求都创建一个新的线程，会导致线程创建和销毁的开销非常大，并且可能会耗尽系统资源。使用线程池可以复用线程，高效地处理大量并发请求，提高 Web Server 的并发处理能力。
2. **资源管理**：线程池可以控制同时运行的线程数量，避免因过多线程占用过多系统资源而导致服务器性能下降甚至崩溃。通过合理配置线程池的大小，可以确保 Web Server 在高负载情况下仍能稳定运行。
3. **提高响应性能**：线程池中的线程已经预先创建好，当有新的请求到来时，可以立即分配线程进行处理，减少了线程创建的时间开销，从而提高了 Web Server 的响应性能，给用户更好的体验。

# ThreadPool 成员变量

```C++
 // 用一个结构体封装起来，方便调用
struct Pool {
    bool is_closed;
    std::mutex mtx_;
    std::condition_variable cond_;
    std::queue<std::function<void()>> tasks_; // 任务队列
};
std::shared_ptr<Pool> pool_; 
```

- `mtx_`：互斥锁，用于保护任务队列和线程池状态的访问，确保线程安全。
- `pool_`：`std::shared_ptr<Pool>` 类型的智能指针，用于管理 `Pool` 对象的生命周期。

# 构造函数

- `std::thread`：创建一个新的线程，并使用 lambda 表达式作为线程的执行体。
  - `std::unique_lock<std::mutex> locker(pool_->mtx_)`：获取互斥锁，确保线程安全。
- 线程无限循环，不断检查任务队列的状态：
  - 如果任务队列不为空，取出队首任务，释放锁，执行任务，然后重新获取锁。
  - 如果任务队列为空且线程池未关闭，调用 `pool_->cond_.wait(locker)` 使线程进入等待状态，直到有新任务到来或线程池关闭。
  - 如果线程池已关闭，跳出循环，线程退出

- `detach()`：将线程分离，使其在后台独立运行。

```C++
ThreadPool() = default;
ThreadPool(ThreadPool&&) = default;
explicit ThreadPool(int thread_count = 8) : pool_(std::make_shared<Pool>()) {
    assert(thread_count > 0);
    for (int i = 0; i < thread_count; ++i) {
        std::thread([this]() {
            std::unique_lock<std::mutex> locker(pool_->mtx_);
            while (true) {
                if (!pool_->tasks_.empty()) {   // 有任务，取任务执行
                    auto task = std::move(pool_->tasks_.front());
                    pool_->tasks_.pop();
                    locker.unlock();
                    task();
                    locker.lock();
                } else if (!pool_->is_closed) { // 未关闭，等待
                    pool_->cond_.wait(locker);
                } else {                        // 已关闭，退出 
                    break;
                }
            }
        }).detach();
    }
}
```

# 析构函数

- `pool_->cond_.notify_all()` ：通知所有等待的线程，使其检查线程池状态并退出循环。

```C++
~ThreadPool() {
    if (pool_) {
        std::lock_guard<std::mutex> locker(pool_->mtx_);
        pool_->is_closed = true;
    }
    pool_->cond_.notify_all();
}
```

**添加任务方法**

- `std::forward<T>(task)`：使用 `std::forward` 进行完美转发，保留任务的左值或右值属性。

```C++
template<class T>
void AddTask(T&& task) {
    std::lock_guard<std::mutex> locker(pool_->mtx_);
    pool_->tasks_.push(std::forward<T>(task));
    pool_->cond_.notify_one();
}
```

# TheadPool 代码

**theadpool.h**

```C++
class ThreadPool {
public:
    ThreadPool() = default;
    ThreadPool(ThreadPool&&) = default;
    explicit ThreadPool(int thread_count = 8) : pool_(std::make_shared<Pool>()) {
        assert(thread_count > 0);
        for (int i = 0; i < thread_count; ++i) {
            std::thread([this]() {
                std::unique_lock<std::mutex> locker(pool_->mtx_);
                while (true) {
                    if (!pool_->tasks_.empty()) {   // 有任务，取任务执行
                        auto task = std::move(pool_->tasks_.front());
                        pool_->tasks_.pop();
                        locker.unlock();
                        task();
                        locker.lock();
                    } else if (!pool_->is_closed) { // 未关闭，等待
                        pool_->cond_.wait(locker);
                    } else {                        // 已关闭，退出 
                        break;
                    }
                }
            }).detach();
        }
    }

    ~ThreadPool() {
        if (pool_) {
            std::lock_guard<std::mutex> locker(pool_->mtx_);
            pool_->is_closed = true;
        }
        pool_->cond_.notify_all();
    }

    template<class T>
    void AddTask(T&& task) {
        std::lock_guard<std::mutex> locker(pool_->mtx_);
        pool_->tasks_.push(std::forward<T>(task));
        pool_->cond_.notify_one();
    }

private:
    struct Pool {
        bool is_closed;
        std::mutex mtx_;
        std::condition_variable cond_;
        std::queue<std::function<void()>> tasks_; // 任务队列
    };
    std::shared_ptr<Pool> pool_; 
};
```

