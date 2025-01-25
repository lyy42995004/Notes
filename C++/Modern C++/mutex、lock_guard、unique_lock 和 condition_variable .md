# mutex

`std::mutex` 是 C++ 标准库中提供的一种互斥量，用于在多线程环境下同步线程对共享资源的访问。通过互斥量，可以确保某一时刻只有一个线程能够访问或修改共享资源，从而避免数据竞争。

## 核心成员函数

- **`lock()`**：用于获取互斥量的锁。若互斥量当前未被锁定，调用线程会立即获得锁并继续执行；若互斥量已被其他线程锁定，调用线程会被阻塞，直到锁被释放并成功获取。

- **`try_lock()`**：尝试获取互斥量的锁，但不会阻塞线程。若互斥量当前未被锁定，调用线程会获得锁并返回`true`；若互斥量已被其他线程锁定，函数会立即返回`false`。

- **`unlock()`**：释放互斥量的锁，允许其他线程获取该锁。调用此函数的线程必须是当前持有锁的线程，否则会导致未定义行为。

## 使用场景

- **保护共享资源**：当多个线程需要访问和修改同一个共享资源时，使用`std::mutex`可以确保在任何时刻只有一个线程能对资源进行操作，避免数据不一致的问题。例如，多个线程同时对一个全局变量进行递增操作，就需要用`std::mutex`保护该变量。

```C++
std::mutex mtx;
int counter = 0;

void incrementCounter() {
    for (int i = 0; i < 1000; ++i) {
        // 获取互斥锁
        mtx.lock();
        // 对共享计数器进行递增
        ++counter;
        // 释放互斥锁
        mtx.unlock();
    }
}
```

# lock_guard

`std::lock_guard` 是 C++ 标准库中的一个 **RAII**（Resource Acquisition Is Initialization）锁管理器，用于简化互斥量的加锁和解锁操作。它在构造时自动加锁互斥量，在作用域结束时自动解锁，从而确保互斥量能够正确释放，避免因为异常或逻辑错误导致的死锁或资源泄漏问题。

## 功能和特性

- **自动管理锁的生命周期**：`std::lock_guard` 在构造时会自动锁定互斥量，在其作用域结束时（即对象被销毁时），会自动调用互斥量的 `unlock` 方法解锁，无需手动调用 `lock` 和 `unlock`，避免了因忘记解锁而导致的死锁问题。
- **独占所有权**：同一时间只有一个 `std::lock_guard` 对象可以拥有互斥量的所有权，确保对共享资源的独占访问，防止多线程并发访问时的数据竞争。
- **不可复制**：`std::lock_guard` 的拷贝构造函数和拷贝赋值运算符被删除，不允许复制，因为复制可能会导致多个对象同时管理同一个互斥量，引发未定义行为。

## 构造函数

- **`explicit lock_guard(mutex_type& m);`**：创建一个 `std::lock_guard` 对象，并立即锁定互斥量 `m`。`explicit` 关键字用于防止隐式类型转换，确保只能显式调用该构造函数。
- **`lock_guard(mutex_type& m, adopt_lock_t t);`**：假设当前线程已经锁定了互斥量 `m`，创建 `std::lock_guard` 对象来管理该互斥量，不会再次尝试锁定。`adopt_lock_t` 是一个空的标记类型，用于区分不同的构造方式。

## 使用场景

- **简单的临界区保护**：当需要保护一个简单的代码块，确保同一时间只有一个线程可以执行该代码块时，`std::lock_guard` 是一个很好的选择。例如，对共享变量的读写操作、对共享数据结构的修改等。

```C++
std::mutex mtx;
int counter = 0;

void incrementCounter() {
    for (int i = 0; i < 1000; ++i) {
        // 创建 std::lock_guard 对象，构造时自动锁定互斥量
        std::lock_guard<std::mutex> lock(mtx);
        // 对共享计数器进行递增
        ++counter;
        // 离开此作用域，std::lock_guard 对象析构，自动解锁互斥量
    }
}
```

# unique_lock

`std::unique_lock` 是 C++ 标准库中提供的一种线程同步工具，位于 `<mutex>` 头文件中。它是一个智能锁，提供了更灵活的互斥锁管理方式，与 `std::lock_guard` 相比，功能更强大。

## 功能和特性

- **独占所有权**：同一时间内，`std::unique_lock`保证只有一个实例能拥有互斥量，这避免了多线程环境下对共享资源的并发访问问题，确保线程安全。
- **灵活锁定**：在创建时不强制立即锁定互斥量，也可以在生命周期内随时锁定或解锁，比`std::lock_guard`更具灵活性。
- **可移动性**：`std::unique_lock`支持移动语义，允许在不同作用域和函数间转移互斥量的所有权。

## 构造函数

- **`unique_lock(mutex_type& m)`**：创建对象并立即锁定互斥量`m`，若互斥量被其他线程占用，当前线程会阻塞。
- **`unique_lock(mutex_type& m, std::defer_lock_t t)`**：创建对象但不锁定互斥量，后续需手动调用`lock()`锁定。

## 核心成员函数

- **`lock()`**：锁定关联的互斥量，若被其他线程占用则阻塞。

- **`unlock()`**：解锁关联的互斥量。
- **`try_lock()`**：尝试锁定互斥量，成功返回`true`，失败返回`false`，不阻塞。
- **`owns_lock()`**：检查对象是否拥有互斥量的所有权。
- **`release()`**：释放互斥量的所有权，返回指向互斥量的指针。
- **`mutex()`**：返回管理的互斥量指针。

## 使用场景

- **需要灵活锁定和解锁的场景**：当在临界区中需要根据特定条件锁定或解锁互斥量时，`std::unique_lock`非常有用。

```C++
std::mutex mtx;
int sharedData = 0;

void complexTask() {
    // 创建 std::unique_lock 对象，但不立即锁定互斥锁
    std::unique_lock<std::mutex> lock(mtx, std::defer_lock);
    // ...

    // 根据条件决定是否锁定
    if (true) {
        lock.lock();
        // 临界区
        sharedData++;
        std::cout << "Shared resource value: " << shareData << std::endl;
        // 提前解锁
        lock.unlock();
    }

    // ...
}
```

- **与条件变量配合使用**：`std::condition_variable`的`wait`系列函数要求使用`std::unique_lock`，因为这些函数在等待期间需要解锁和重新锁定互斥量。

```C++
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

std::mutex mtx;
std::condition_variable cv;
bool ready = false;

// 等待特定条件达成，继续工作
void worker() {
    std::unique_lock<std::mutex> lock(mtx);
    // 等待条件变量通知
    cv.wait(lock, [] { return ready; });
    std::cout << "Worker thread is working..." << std::endl;
}

// 设置条件变量并通知等待的线程
void setReady() {
    {
        std::lock_guard<std::mutex> lock(mtx);
        ready = true;
    }
    // 通知等待的线程
    cv.notify_one();
}

int main() {
    std::thread t1(worker);
    std::thread t2(setReady);
    t1.join();
    t2.join();
    return 0;
}
```

# lock_guard对比unique_lock

| **特性** | **std::lock_guard** | **std::unique_lock** |
| ----- | ---- | ---- |
| 自动加锁 | 是   | 是   |
| 延迟锁定 | 否   | 是   |
| 尝试锁定 | 否   | 是   |
| 手动解锁 | 否   | 是   |
| 接管已锁定的互斥量 | 支持 | 支持 |
| 开销 | 较低 | 较高 |

- `std::lock_guard` 是 C++ 提供的简单、高效的锁管理工具，非常适合短时间的互斥量保护场景。通过 RAII 机制，`std::lock_guard` 大大减少了因手动管理锁而可能导致的错误。
- `std::unique_lock`可以应对更复杂的锁管理需求（如延迟锁定或多次解锁/加锁）。

# condition_variable

条件变量 (`std::condition_variable`) 是 C++ 标准库中提供的一种线程同步工具，用于线程之间的高效通信和协调。它允许一个线程等待某个条件满足时再继续执行，同时其他线程可以通知（signal）它条件已经满足。

## 核心成员函数

- **`wait`**：该函数用于使当前线程阻塞，直到收到通知且满足特定条件。它有两种重载形式：
  - `void wait(std::unique_lock<std::mutex>& lock);`：该形式会释放当前线程持有的互斥锁`lock`，并使线程进入阻塞状态，直到收到其他线程的通知。当线程被唤醒时，它会重新获取互斥锁。
  - `template< class Predicate > void wait(std::unique_lock<std::mutex>& lock, Predicate pred);`：这种形式会在每次被唤醒时检查`pred`条件，如果条件不满足，则继续等待；只有当条件满足时，线程才会继续执行。
- **`wait_for`**：与`wait`类似，但它允许指定一个超时时间。如果在超时时间内没有收到通知或条件未满足，线程会自动唤醒。
- **`notify_one`**：唤醒一个等待在该条件变量上的线程。如果有多个线程在等待，只会唤醒其中一个，具体唤醒哪个线程是不确定的。
- **`notify_all`**：唤醒所有等待在该条件变量上的线程。

## 使用场景

- **生产者-消费者模型**：生产者线程负责生产数据并将其放入缓冲区，消费者线程则从缓冲区取出数据进行处理。当缓冲区为空时，消费者线程需要等待；当缓冲区满时，生产者线程需要等待。

```C++
std::queue<int> buffer;
std::mutex mtx;
std::condition_variable not_full, not_empty;
const int MAX_SIZE = 5;

void producer() {
    for (int i = 0; i < 10; ++i) {
        std::unique_lock<std::mutex> lock(mtx);
        // 等待缓冲区不满
        not_full.wait(lock, [] { return buffer.size() < MAX_SIZE; });
        buffer.push(i);
        std::cout << "Produced: " << i << std::endl;
        // 通知消费者缓冲区非空
        not_empty.notify_one();
    }
}

void consumer() {
    for (int i = 0; i < 10; ++i) {
        std::unique_lock<std::mutex> lock(mtx);
        // 等待缓冲区非空
        not_empty.wait(lock, [] { return !buffer.empty(); });
        int item = buffer.front();
        buffer.pop();
        std::cout << "Consumed: " << item << std::endl;
        // 通知生产者缓冲区非满
        not_full.notify_one();
    }
}
```

**多线程任务同步**：在某些场景中，一个或多个线程需要等待其他线程完成特定任务后才能继续执行。条件变量可以用来实现这种同步机制。

**线程池任务调度**：在线程池中，工作线程需要等待新任务的到来。当有新任务提交时，主线程可以通知一个或多个工作线程开始执行任务。

**资源分配与使用**：当多个线程竞争有限的资源时，条件变量可以用于协调资源的分配和使用。例如，当资源可用时，等待的线程可以被唤醒使用资源；当资源被占用时，线程需要等待。
