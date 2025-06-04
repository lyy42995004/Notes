# 字符串函数

## strcpy()

```cpp
char *strcpy(char *strDest, const char* strSrc) {
    assert(strDest && strSrc);
    char *addr = strDest;
    while ((*strDest++ = *strSrc++) != '\0') {};
    return addr
}
```

## strlen()

```cpp
int strlen(const char *str) {
    assert(str != NULL);
    int len = 0;
    while ((*str++) != '\0')
        len++;
   	return len;
}
```

## strcmp()

```cpp
int strcmp(const char* str1, const char* str2) {
    assert(str1 && str2);
    while (*str1 && *str2 && (*str1 == *str2))
        str1++, str2++;
    return *str1 - str2;
}
```

# 智能指针

## auto_ptr

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

## unique_ptr

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

## shared_ptr

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

## weak_ptr

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

# LRU 缓存

```cpp
#include <unordered_map>
#include <iostream>
using namespace std;

struct Node {
    int key, value;
    Node* next, *prev;
    Node() : key(0), value(0), next(nullptr), prev(nullptr) {}
    Node(int key_, int value_)
    : key(key_), value(value_),  next(nullptr), prev(nullptr) {}
};

class LRUCache {
public:
    LRUCache(int capacity) : size_(0), capacity_(capacity) {
        head_ = new Node();
        tail_ = new Node();
        head_->next = tail_;
        tail_->prev = head_;
    }

    ~LRUCache() {
        Node* cur = head_;
        while (cur) {
            Node* next = cur->next;
            delete cur;
            cur = next;
        }
    }
    
    int get(int key) {
        if (!hash_.count(key)) {
            return -1;
        }
        Node* node = hash_[key];
        moveToHead(node);
        return node->value;
    }
    
    void put(int key, int value) {
        if (hash_.count(key)) {
            Node* node = hash_[key];
            node->value = value;
            moveToHead(node);
        } else {
            size_++;
            Node* node = new Node(key, value);
            hash_[key] = node;
            addToHead(node);
            if (size_ > capacity_) {
                Node* tail = removeTail();
                hash_.erase(tail->key);
                delete tail;
                size_--;
            }
        }
    }

private:
    void removeNode(Node* node) {
        node->prev->next = node->next;
        node->next->prev = node->prev;
    }

    void addToHead(Node* node) {
        Node* head = head_->next;
        head_->next = node;
        node->prev = head_;
        node->next = head;
        head->prev = node;
    }

    void moveToHead(Node* node) {
        removeNode(node);
        addToHead(node);
    }

    Node* removeTail() {
        Node* tail = tail_->prev;
        removeNode(tail);
        return tail;
    }

private:
    Node* head_, *tail_;
    unordered_map<int, Node*> hash_;
    int size_, capacity_;
};

int main() {
    LRUCache* lrucache = new LRUCache(2);

    // 1 -1 -1 3 4
    lrucache->put(1, 1);                // 缓存是 {1=1}
    lrucache->put(2, 2);                // 缓存是 {1=1, 2=2}
    cout << lrucache->get(1) << " ";    // 返回 1
    lrucache->put(3, 3);                // 该操作会使得关键字 2 作废，缓存是 {1=1, 3=3}
    cout << lrucache->get(2) << " ";    // 返回 -1 (未找到)
    lrucache->put(4, 4);                // 该操作会使得关键字 1 作废，缓存是 {4=4, 3=3}
    cout << lrucache->get(1) << " ";    // 返回 -1 (未找到)
    cout << lrucache->get(3) << " ";    // 返回 3
    cout << lrucache->get(4) << " ";    // 返回 4
    cout << endl;

    return 0;
}
```

# 单例模式

## 饿汉

```cpp
class Singleton {
public:
    static Singleton& getInstance() {
        return instance;
    }

private:
    Singleton() {}
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;

    static Singleton instance;
};

Singleton Singleton::instance;
```

## 懒汉

```cpp
#include <mutex>

class Singleton {
public:
    static Singleton* getInstance() {
        if (instance == nullptr) {                 // 第一次判空
            std::lock_guard<std::mutex> lock(mtx);
            if (instance == nullptr) {             // 第二次判空
                instance = new Singleton();
            }
        }
        return instance;
    }
    
    // static Singleton* getInstance() {
    //     // C++11 保证局部静态变量初始化线程安全
    //     static Singleton instance;
    //     return &instance;
    // }

private:
    Singleton() {}
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;
    
    static Singleton* instance;
    static std::mutex mtx;
};

Singleton* Singleton::instance = nullptr;
std::mutex Singleton::mtx;
```

# 线程池

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

# 多线程打印abc

## 有锁

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

## 无锁

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
