# 池化思想

在计算机科学里，**资源高效管理**至关重要，池化思想便是应对此挑战的有效策略。

池化的核心是预先创建资源并集中管理复用。就像一个资源 “池子”，提前备好资源实例，有需求时从中取用，用完归还，避免反复创建销毁带来的高开销。比如创建线程涉及内存分配等操作，频繁进行会耗费大量 CPU 时间，池化能有效规避。同时，资源池还可统一管理监控资源，防止过度使用。

池化在软件系统中应用广泛。**数据库连接池**是典型，应用程序频繁交互数据库，每次新建连接低效，连接池预先创建连接供取用，提升效率且防服务器过载。**线程池**也是基于此思想，多线程编程里，线程创建销毁开销大，线程池创建一定数量线程，任务分配执行后线程再等新任务，实现高效利用。此外，还有**对象池**，对创建成本高的对象如 Socket 对象等，预先缓存按需分配回收。

池化思想凭借高效可控的资源管理，在多领域发挥关键作用。其中，线程池作为其在多线程编程的应用，独具魅力与价值，下面我们就深入探究。

# 线程池

线程池（Thread Pool）是一种多线程处理方式，用于管理和复用多个线程，以提高程序的并发性能并避免频繁创建和销毁线程所带来的开销。

![](https://s2.loli.net/2025/05/07/ZxrYBDf9kyd1zgV.png)

**基本概念**：

线程池维护着若干个**已创建好的线程**，当有任务需要执行时，就从线程池中取出空闲线程来执行任务，任务执行完成后线程并不被销毁，而是返回线程池中继续等待执行下一个任务。

**主要优点**：

1. **减少资源消耗**：重复利用已创建的线程，避免频繁创建和销毁线程的开销。
2. **提高响应速度**：任务到达时可以立即执行（如果有空闲线程），而不必等待创建新线程。
3. **提高线程可管理性**：线程池可以设置线程的最大并发数，防止系统因为创建过多线程而耗尽资源。
4. **任务排队机制**：可以统一调度和管理任务，比如按顺序执行、设置优先级、延迟执行等。

**使用场景**：

1. **Web 服务器**：处理高并发的客户端请求，复用线程，提升效率。
2. **任务并行处理**：同时处理多个独立任务，减少总执行时间。
3. **后台定时任务**：定时执行任务或后台处理，避免阻塞主线程。
4. **IO密集型任务**：提高系统资源利用率，如网络请求或磁盘读写。

# 实现

## 构造函数

创建指定数量的工作线程，并让这些线程等待执行任务。

```cpp
ThreadPool() = default;
ThreadPool(ThreadPool&&) = default;

explicit ThreadPool(size_t thread_count = 4) : stop_(false) {
    for (size_t i = 0; i < thread_count; ++i) {
        workers_.emplace_back([this] {
            while (true) {
                std::function<void()> task;
                {
                    std::unique_lock<std::mutex> lock(mtx_);
                    cond_.wait(lock, [this] {
                        return stop_ || !tasks_.empty();
                    });

                    if (stop_ && tasks_.empty()) {
                        return;
                    }

                    task = std::move(tasks_.front());
                    tasks_.pop();
                }
                task();  // 执行任务
            }
        });
    }
}
```

- **循环创建线程**：使用 `for` 循环创建 `thread_count` 个工作线程。
- **无限循环**：每个工作线程进入一个无限循环，持续等待任务。
- **任务变量**：`std::function<void()>` 类型的变量 `task` 用于存储从任务队列中取出的任务。
- **加锁操作**：使用 `std::unique_lock<std::mutex>` 对互斥锁 `mtx_` 进行加锁，确保在访问任务队列时的线程安全。
- **条件变量等待**：`cond_.wait()` 让线程进入等待状态，直到满足 `stop_` 为 `true` 或者任务队列 `tasks_` 不为空的条件。当条件满足时，线程被唤醒并继续执行。
- **停止条件判断**：如果 `stop_` 为 `true` 且任务队列为空，说明线程池已经停止工作，工作线程跳出循环并结束执行。
- **取出任务**：使用 `std::move` 将任务队列的第一个任务移动到 `task` 变量中，并从队列中移除该任务。
- **执行任务**：`task()` 调用取出的任务，执行具体的工作。

> **`emplace_back`**：`workers_` 是一个 `std::vector<std::thread>` 类型的容器，用于存储所有的工作线程。`emplace_back` 直接在容器尾部构造一个新的线程对象，避免了不必要的拷贝或移动操作。
>
> **lambda 表达式**：传递给线程构造函数的是一个 lambda 表达式，这个 lambda 表达式定义了每个工作线程的执行逻辑。`[this]` 表示该 lambda 表达式可以访问当前对象的成员变量和成员函数。
>
> **`std::move`**：`tasks_` 是存储任务的 `std::queue` 容器，`tasks_.front()` 为队首任务。`std::move` 把任务从队列转移到 `task` 变量，借助 “转移语义” 避免拷贝，转移所有权，降低资源开销、提升效率，保障多线程下取任务的安全性。

## 析构函数

在对象生命周期结束时，安全地停止线程池并释放相关资源。

```cpp
~ThreadPool() {
    {
        std::lock_guard<std::mutex> lock(mtx_);
        stop_ = true;
    }
    cond_.notify_all();  // 唤醒所有线程
    for (std::thread &worker : workers_) {
        worker.join();  // 等待线程结束
    }
}
```

- **加锁与设置停止标志**：利用 `std::lock_guard` 对互斥锁 `mtx_` 加锁，保证多线程环境下对成员变量 `stop_` 的安全访问。将成员变量 `stop_` 设置为 `true`，告知工作线程线程池要停止运行。
- **唤醒所有线程**：`cond_.notify_all()`通过条件变量 `cond_` 唤醒所有因等待条件而处于阻塞状态的工作线程，让它们得知线程池要停止运行并进行相应处理。
- **等待线程结束**：遍历线程池中的所有工作线程，检查线程是否处于可连接状态，只有可连接的线程才能调用 `join` 方法。主线程调用 `join` 方法等待每个工作线程执行完毕，确保在析构函数结束时所有工作线程都已完成任务，避免资源泄漏。

## 添加任务

将可执行任务添加到线程池的任务队列中，等待工作线程执行。

```cpp
void addTask(const std::function<void()>& task) {
    {
        std::lock_guard<std::mutex> lock(mtx_);
        tasks_.push(task);
    }
    cond_.notify_one();  // 唤醒一个线程
}
```

- **加锁操作**：通过 `std::lock_guard` 对互斥锁 `mtx_` 进行加锁，保证在多线程环境下向任务队列 `tasks_` 添加任务时的线程安全，避免数据竞争。
- **任务入队**：`tasks_.push(task);` 将传入的任务（`std::function<void()>` 类型的 `task` ）添加到任务队列 `tasks_` 中，等待工作线程后续处理。
- **唤醒线程**：`cond_.notify_one();` 调用条件变量 `cond_` 的 `notify_one` 方法，唤醒一个因等待任务而处于阻塞状态的工作线程，使其能够从任务队列中获取并执行新任务。

## 代码

```cpp
#pragma once
#include <iostream>
#include <thread>
#include <vector>
#include <functional>
#include <queue>
#include <mutex>
#include <condition_variable>
#include <atomic>

class ThreadPool {
public:
    explicit ThreadPool(size_t thread_count) : stop_(false) {
        for (size_t i = 0; i < thread_count; ++i) {
            workers_.emplace_back([this] {
                while (true) {
                    std::function<void()> task;
                    {
                        std::unique_lock<std::mutex> lock(mtx_);
                        cond_.wait(lock, [this] {
                            return stop_ || !tasks_.empty();
                        });

                        if (stop_ && tasks_.empty()) {
                            return;
                        }

                        task = std::move(tasks_.front());
                        tasks_.pop();
                    }
                    task();  // 执行任务
                }
            });
        }
    }

    ~ThreadPool() {
        {
            std::lock_guard<std::mutex> lock(mtx_);
            stop_ = true;
        }
        cond_.notify_all();  // 唤醒所有线程
        for (std::thread &worker : workers_) {
            worker.join();  // 等待线程结束
        }
    }

    void addTask(const std::function<void()>& task) {
        {
            std::lock_guard<std::mutex> lock(mtx_);
            tasks_.push(task);
        }
        cond_.notify_one();  // 唤醒一个线程
    }

private:
    std::vector<std::thread> workers_;
    std::queue<std::function<void()>> tasks_;
    std::mutex mtx_;
    std::condition_variable cond_;
    std::atomic<bool> stop_;
};
```

# 更高级的实现addTask()

>  这是我在学习线程池时，在GitHub上搜索发现的一个高star的线程池库，下面我来简单讲解一下，这个库在`addTask()`上的升级，[A simple C++11 Thread Pool implementation](https://github.com/progschj/ThreadPool/blob/master/ThreadPool.h)。

## 原来的 `addTask()`

**功能**：

- 接收一个 `std::function<void()>` 形式的可调用对象
- 加入任务队列
- 唤醒一个工作线程去执行

 **限制**：

- 只能加入无返回值、无参数的任务（`std::function<void()>`）
- 不能异步获取任务的执行结果（没有 `future`）
- 没有任务参数的转发和封装功能（比如带参数、lambda 带捕获、右值引用参数都不太方便）

##  `addTask`（高级版）

 **功能**：

-  **支持任意返回值的任务**
-  **支持传递任意参数**（可变模板 + 完美转发）
-  **异步获取任务返回值**（通过 `std::future`）
-  **任务执行异常安全**（异常可以通过 `future.get()` 捕获）
-  **任务加入时检查线程池状态**（防止池关闭后加入任务）

```cpp
template<class F, class... Args>
auto addTask(F&& f, Args&&... args) 
    -> std::future<typename std::result_of<F(Args...)>::type> {
    using return_type = typename std::result_of<F(Args...)>::type;

    auto task = std::make_shared<std::packaged_task<return_type()>>(
        std::bind(std::forward<F>(f), std::forward<Args>(args)...)
    );

    std::future<return_type> res = task->get_future();
    {
        std::unique_lock<std::mutex> lock(mtx_);

        if (stop_) {
            throw std::runtime_error("addTask on stopped ThreadPool");
        }

        tasks_.emplace([task](){ (*task)(); });
    }
    cond_.notify_one();
    return res;
}
```

> ### `template<class F, class... Args>`
>
> **可变模板参数（Variadic Templates）**
>
> - `F` 表示一个可调用对象类型（比如函数、lambda、函数对象等）
> - `Args...` 表示可变数量的参数类型，配合 `args...` 代表具体参数。
> - 可变模板参数在 C++11 引入，允许一个模板接受任意数量和类型的模板参数。
>
> ------
>
> ###  `auto addTask(F&& f, Args&&... args) -> std::future<typename std::result_of<F(Args...)>::type>`
>
>  **返回类型后置（Trailing Return Type）+ 类型萃取**
>
> - `auto ... -> xxx` 是 C++11 的后置返回类型声明方式，方便当返回值依赖于模板参数时先写参数，再推导返回值。
> - `std::result_of<F(Args...)>::type` 作用是：
>   - 推导出 `f` 这个可调用对象用 `args...` 调用后的返回值类型。
>   - 比如 `int f(double, string)`，那么 `std::result_of<decltype(f)(double, string)>::type` 就是 `int`。
> - **警告**：`std::result_of` 在 C++17 起被弃用，推荐用 `std::invoke_result`。
>
> ------
>
> ###  `using return_type = typename std::result_of<F(Args...)>::type;`
>
> **类型别名（Type Alias）**
>
> - 这里给 `std::result_of<F(Args...)>::type` 起了个别名叫 `return_type`，方便后续多处用。
>
> ------
>
> ###  `std::make_shared<std::packaged_task<return_type()>>(...)`
>
> **智能指针 + packaged_task**
>
> - `std::packaged_task<return_type()>`：
>   - 包装一个可调用对象（比如函数、lambda）
>   - 执行后可以获取执行结果，和 `std::future` 联动。
> - `std::make_shared`：
>   - 智能指针，创建对象同时管理其生命周期，避免裸指针泄漏。
>   - `std::make_shared` 比 `new` 更高效、安全。
>
> ------
>
> ### `std::bind(std::forward<F>(f), std::forward<Args>(args)...)`
>
>  **`std::bind` 和 完美转发（Perfect Forwarding）**
>
> - `std::bind`：
>   - 将一个函数或可调用对象和其参数绑定，生成一个可调用对象，稍后执行。
> - `std::forward`：
>   - 保持参数的左值/右值属性，避免不必要的拷贝。
>   - 这是完美转发的核心，避免右值被当作左值引用传递。
>
> ------
>
> ### `std::future<return_type> res = task->get_future();`
>
>  **future/promise机制**
>
> - `packaged_task` 内部关联了 `std::promise`
> - `get_future()` 可以拿到与这个 task 绑定的 `future` 对象，执行 `task()` 后，`future.get()` 就能取到执行结果。
>
> ------
>
> ### `tasks_.emplace([task](){ (*task)(); });`
>
>  **Lambda捕获 + emplace**
>
> - `emplace`：
>   - 直接在队列尾部原地构造对象，效率高于 `push`。
> - `[task]() { (*task)(); }`：
>   - 捕获 `task` 智能指针，生成一个无参 lambda
>   - 执行 lambda 就等于执行 `task()`，触发真正的调用。

# 使用

```cpp
#include <iostream>
#include "threadpool.h"

int main() {
    ThreadPool pool(4);
    for (int i = 0; i < 10; ++i) {
        pool.addTask([i] {
            std::cout << "Task " << i << " running in thread " 
                      << std::this_thread::get_id() << std::endl;
        });
    }
    std::this_thread::sleep_for(std::chrono::seconds(1));  // 简单等一下
}
```

更到的`addTask()`函数的使用

```cpp
int main() {
    ThreadPool pool(4);
    std::vector< std::future<int> > results;

    for(int i = 0; i < 8; ++i) {
        results.emplace_back(
            pool.addTask([i] {
                std::cout << "hello " << i << std::endl;
                std::this_thread::sleep_for(std::chrono::seconds(1));
                std::cout << "world " << i << std::endl;
                return i*i;
            })
        );
    }

    for(auto && result: results)
        std::cout << result.get() << ' ';
    std::cout << std::endl;
    
    return 0;
}
```

