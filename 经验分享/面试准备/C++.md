# 智能指针

**auto_ptr**

```cpp
template <typename T>
class MyAutoPtr {
public:
    explicit MyAutoPtr(T* ptr = nullptr) : ptr_(ptr) {}

    MyAutoPtr(MyAutoPtr& other) : ptr_(other.ptr_) {
        other.ptr_ = nullptr;
    }

    MyAutoPtr& operator=(MyAutoPtr& other) {
        if (this != &other) {
            delete ptr_;
            ptr_ = other.ptr_;
            other.ptr_ = nullptr;
        }
        return *this;
    }

    ~MyAutoPtr() { delete ptr_; }

    T& operator*() const { return *ptr_; }
    T* operator->() const { return ptr_; }
    T* get() const { return ptr_; }

private:
    T* ptr_;
};
```

**unique_ptr**

```cpp
template <typename T>
class MyUniquePtr {
public:
    explicit MyUniquePtr(T* ptr = nullptr) : ptr_(ptr) {}

    MyUniquePtr(const MyUniquePtr&) = delete;
    MyUniquePtr& operator=(const MyUniquePtr&) = delete;
    
    MyUniquePtr(MyUniquePtr&& other) : ptr_(other.ptr_) {
        other.ptr_ = nullptr;
    }

    MyUniquePtr& operator=(MyUniquePtr&& other) {
        if (this != &other) {
            delete ptr_;
            ptr_ = other.ptr_;
            other.ptr_ = nullptr;
        }
        return *this;
    }

    ~MyUniquePtr() {
        delete ptr_;
    }

    T& operator*() const { return *ptr_; }
    T* operator->() const { return ptr_; }
    T* get() const { return ptr_; }

private:
    T* ptr_;
};
```

**shared_ptr**

```cpp
template <typename T>
class MySharedPtr {
public:
    explicit MySharedPtr(T* ptr)
        : ptr_(ptr), ref_count_(new int(1)) {}

    MySharedPtr(const MySharedPtr& other) 
        : ptr_(other.ptr_), ref_count_(other.ref_count_) {
        ++(*ref_count_);
    }

    MySharedPtr& operator=(const MySharedPtr& other) {
        if (this != &other) {
            release();
            ptr_ = other.ptr_;
            ref_count_ = other.ref_count_;
            ++(*ref_count_);
        }
        return this;
    }

    ~MySharedPtr() {
        release();
    }

    T& operator*() const { return *ptr_; }
    T* operator->() const { return ptr_; }

    T* get() const { return ptr_; }
    int use_count() const { return (*ref_count_); }

private:
    T* ptr_;
    int* ref_count_;

    void release() {
        if (--(*ref_count_) == 0) {
            delete ptr_;
            delete ref_count_;
        }
    }
};
```

**weak_ptr**

```cpp
template <class T>
class MyWeakPtr {
public:
    MyWeakPtr() :_ptr(nullptr) {}

    MyWeakPtr(const MySharedPtr<T>& sp)
        :_ptr(sp.get()) {}

    MyWeakPtr<T>& operator=(const MySharedPtr& sp) {
        _ptr = sp.get();
        return *this;
    }

    T& operator*() const { return *ptr_; }
    T* operator->() const { return ptr_; }
    
private:
    T* _ptr;
};
```

# 单例模式

**饿汉**

```cpp
#include <iostream>

class EagerSingleton {
private:
    // 静态成员变量，在程序启动时就已经初始化
    static EagerSingleton instance;
    
    // 私有构造函数，防止外部实例化
    EagerSingleton() {
        std::cout << "EagerSingleton instance created" << std::endl;
    }
    
    // 防止拷贝和赋值
    EagerSingleton(const EagerSingleton&) = delete;
    EagerSingleton& operator=(const EagerSingleton&) = delete;
    
public:
    // 获取单例实例的静态方法
    static EagerSingleton& getInstance() {
        return instance;
    }
    
    void doSomething() {
        std::cout << "EagerSingleton do something" << std::endl;
    }
};

// 在类外初始化静态成员变量
EagerSingleton EagerSingleton::instance;

// 使用示例
int main() {
    EagerSingleton::getInstance().doSomething();
    return 0;
}
```

**懒汉**

```cpp
#include <iostream>
#include <mutex>

class ThreadSafeLazySingleton {
private:
    static ThreadSafeLazySingleton* instance;
    static std::mutex mtx;
    
    ThreadSafeLazySingleton() {
        std::cout << "ThreadSafeLazySingleton instance created" << std::endl;
    }
    
    ThreadSafeLazySingleton(const ThreadSafeLazySingleton&) = delete;
    ThreadSafeLazySingleton& operator=(const ThreadSafeLazySingleton&) = delete;
    
public:
    static ThreadSafeLazySingleton* getInstance() {
        // 第一次检查，避免不必要的锁开销
        if (instance == nullptr) {
            std::lock_guard<std::mutex> lock(mtx);
            // 第二次检查，确保只有一个线程创建实例
            if (instance == nullptr) {
                instance = new ThreadSafeLazySingleton();
            }
        }
        return instance;
    }
    
    void doSomething() {
        std::cout << "ThreadSafeLazySingleton do something" << std::endl;
    }
};

// 初始化静态成员变量
ThreadSafeLazySingleton* ThreadSafeLazySingleton::instance = nullptr;
std::mutex ThreadSafeLazySingleton::mtx;

// 使用示例
int main() {
    ThreadSafeLazySingleton::getInstance()->doSomething();
    return 0;
}
```

```cpp
#include <iostream>

class Cpp11LazySingleton {
private:
    Cpp11LazySingleton() {
        std::cout << "Cpp11LazySingleton instance created" << std::endl;
    }
    
    Cpp11LazySingleton(const Cpp11LazySingleton&) = delete;
    Cpp11LazySingleton& operator=(const Cpp11LazySingleton&) = delete;
    
public:
    static Cpp11LazySingleton& getInstance() {
        // C++11保证局部静态变量的初始化是线程安全的
        static Cpp11LazySingleton instance;
        return instance;
    }
    
    void doSomething() {
        std::cout << "Cpp11LazySingleton do something" << std::endl;
    }
};

// 使用示例
int main() {
    Cpp11LazySingleton::getInstance().doSomething();
    return 0;
}
```

# 线程池

```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <functional>
#include <thread>
#include <mutex>
#include <atomic>
#include <condition_variable>

class ThreadPool {
public:
    explicit ThreadPool(size_t cnt) : stop(false) {
        for (size_t i = 0; i < cnt; i++) {
            workers.emplace_back([this] {
                while (true) {
                    std::function<void()> task;
                    {
                        std::unique_lock<std::mutex> lock(mtx);
                        // 防止虚假唤醒
                        cond.wait(lock, [this] {
                            return stop || !tasks.empty();
                        });
                        // 检查停止
                        if (stop && tasks.empty()) {
                            return;
                        }
                        task = tasks.front();
                        tasks.pop();
                    }
                    task();
                }
            });
        }
    }

    ~ThreadPool() {
        {
            std::lock_guard<std::mutex> lock(mtx);
            stop = true;
        }
        cond.notify_all();
        for (std::thread& worker : workers) {
            worker.join();
        }
    }

    void addTask(const std::function<void()>& f) {
        {
            std::lock_guard<std::mutex> lock(mtx);
            tasks.push(f);
        }
        cond.notify_one();
    }

private:
    std::vector<std::thread> workers;
    std::queue<std::function<void()>> tasks;
    std::mutex mtx;
    std::condition_variable cond;
    std::atomic<bool> stop;
};

int main() {
    ThreadPool p(4);

    // 添加带ID的任务
    for (int i = 0; i < 5; ++i) {
        p.addTask([i] {  // 捕获i
            std::cout << "Task " << i << " started on thread " 
                      << std::this_thread::get_id() << std::endl;
            std::this_thread::sleep_for(std::chrono::milliseconds(100));
            std::cout << "Task " << i << " finished" << std::endl;
        });
    }
    
    // 等待所有任务完成
    std::this_thread::sleep_for(std::chrono::seconds(1));
}
```

# 多线程打印abc

**有锁**

**mutex + condition_variable**

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>
using namespace std;

mutex mtx;
condition_variable cv;
int turn = 0; // 0: a, 1: b, 2: c

void printChar(char ch, int myTurn, int n) {
    for (int i = 0; i < n; ++i) {
        unique_lock<mutex> lock(mtx);
        cv.wait(lock, [&] { return turn == myTurn; });
        cout << ch;
        turn = (turn + 1) % 3;
        cv.notify_all();
    }
}

int main() {
    int n = 10;
    thread t1(printChar,'a', 0, n);
    thread t2(printChar,'b', 1, n);
    thread t3(printChar,'c', 2, n);
    t1.join();
    t2.join();
    t3.join();
    cout << endl;

    return 0;
}
```

**无锁**

**atomic + 自旋**

```cpp
#include <iostream>
#include <thread>
#include <atomic>
using namespace std;

atomic<int> turn(0); // 0: a, 1: b, 2: c

void printChar(char ch, int myTurn, int n) {
    while(n--) {
        while (turn.load() != myTurn) {
            // 自旋等待
            this_thread::yield();
        }
        cout << ch;
        turn = (turn + 1) % 3;
    }
}

int main() {
    int n = 10;
    thread t1(printChar,'a', 0, n);
    thread t2(printChar,'b', 1, n);
    thread t3(printChar,'c', 2, n);
    t1.join();
    t2.join();
    t3.join();
	cout << endl;
    
    return 0;
}
```
