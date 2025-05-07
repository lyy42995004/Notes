C++ 中智能指针都是 RAII（Resource Acquisition Is Initialization）机制的典型应用，在构造时获取资源，在析构时释放资源，将资源管理与对象的生命周期绑定，使得资源管理更加直观和可靠。

# unique_ptr

`std::unique_ptr` 的主要作用是对动态分配的资源进行严格的独占式管理，确保在其生命周期结束时自动释放所管理的资源，从而防止内存泄漏，让资源的生命周期与 `std::unique_ptr` 变量的生命周期紧密绑定，无需手动调用 `delete` 操作符，使资源管理更安全、更高效。

## 功能和特性

- **独占所有权**：同一时刻只能有一个 `std::unique_ptr` 指向给定的资源，保证了资源的独占访问，避免多个指针同时操作同一资源带来的冲突和数据不一致问题。
- **移动语义**：支持移动构造和移动赋值操作，允许将资源的所有权从一个 `std::unique_ptr` 转移到另一个 `std::unique_ptr`，但不支持复制操作，提高了资源转移的效率，避免了不必要的资源复制。
- **自动释放资源**：当 `std::unique_ptr` 对象超出作用域或被显式销毁时，会自动调用其析构函数来释放所管理的资源，无需手动释放，降低了忘记释放资源导致内存泄漏的风险。
- **可定制删除器**：可以通过模板参数指定自定义的删除器，用于定义释放资源的特定方式，以满足不同资源类型的特殊释放需求。
- **空指针值**：可以通过默认构造函数创建一个空的 `std::unique_ptr`，表示不管理任何资源，也可以通过将其赋值为 `nullptr` 来使其变为空指针。

## 使用场景

- **函数内部局部资源管理**：在函数内部动态分配资源时，使用 `std::unique_ptr` 来管理这些资源，确保函数执行结束时资源能被正确释放，例如在函数中动态分配数组或对象。
- **资源独占场景**：当某个资源只需要被一个对象独占使用时，如一个文件操作类中使用 `std::unique_ptr` 来管理文件句柄，保证同一时间只有该对象能访问文件。
- **作为函数返回值**：函数可以返回一个 `std::unique_ptr`，将动态分配资源的所有权安全地转移给调用者，调用者可以继续管理该资源，而无需担心资源的释放问题。
- **管理动态分配的数组**：可以使用 `std::unique_ptr` 来管理动态分配的数组，通过指定数组删除器来确保数组内存的正确释放。

## make_unique

`std::make_unique` 是 C++14 引入的一个辅助函数，用于创建并返回 `std::unique_ptr` 对象。它在动态分配内存时，提供了更简洁和安全的方式，与 `std::unique_ptr` 的构造配合使用，减少了潜在的错误风险。

| 特点 | `std::make_unique` | `std::unique_ptr`和`new` |
| - | - | - |
| **内存分配** | 高效，单一操作分配内存 | 初始化和分配是分离的操作 |
| **异常安全** | 更安全，避免资源泄漏 | 可能因异常导致内存泄漏 |
| **代码简洁性** | 更简洁，无需手动使用 `new` | 手动使用 `new`，代码更冗长 |
| **自定义删除器** | 无法直接使用自定义删除器 | 支持自定义删除器 |

**示例**

```C++
int main() {
    // 使用 unique_ptr 管理一个 vector<int>
    auto vecPtr = std::make_unique<std::vector<int>>();

    vecPtr->push_back(10);
    vecPtr->push_back(20);
    vecPtr->push_back(30);

    std::cout << "Vector elements: ";
    for (const auto& elem : *vecPtr) {
        std::cout << elem << " ";
    }
    std::cout << "\n";

    // vecPtr2 超出作用域时，资源会自动释放
    return 0;
}
```

## 模拟实现

```C++
template <typename T>
class UniquePtr {
public:
    explicit UniquePtr(T* ptr = nullptr) : ptr_(ptr) {}
    
    UniquePtr(const UniquePtr&) = delete;
    UniquePtr<T>& operator=(const UniquePtr&) = delete;
    
    UniquePtr(UniquePtr&& other) noexcept : ptr_(other.ptr_) {
        other.ptr_ = nullptr;
    }
    
    UniquePtr& operator=(UniquePtr&& other) noexcept {
        if (this != &other) {
            delete ptr_;
            ptr_ = other.ptr_;
            other.ptr_ = nullptr;
        }
        return *this;
    }
    
    ~UniquePtr() { delete ptr_; }
    
    T& operator*() { return *ptr_; }
    T* operator->() { return ptr_; }
    
    // 获取原始指针
    T* get() const { return ptr_; }
    
    // 重置指针
    void reset(T* ptr = nullptr) {
        if (ptr != ptr_) {
            delete ptr_;
            ptr_ = ptr;
        }
    }
    
    // 释放所有权
    T* release() {
        T* tmp = ptr_;
        ptr_ = nullptr;
        return tmp;
    }

private:
    T* ptr_;
};
```

# shared_ptr

`std::shared_ptr` 的主要作用是实现资源的共享所有权。多个 `std::shared_ptr` 可以指向同一个对象，通过引用计数机制来跟踪有多少个 `std::shared_ptr` 共享该对象。当最后一个指向该对象的 `std::shared_ptr` 被销毁或重置时，对象的内存会被自动释放，从而避免了内存泄漏。

`std::shared_ptr` 可能会导致循环引用问题，即两个或多个 `std::shared_ptr` 相互引用，使得引用计数永远不会变为 0，从而导致内存泄漏。为了解决这个问题，可以使用 `std::weak_ptr`，它是一种弱引用，不会增加引用计数。

## 功能和特性

- **引用计数**：`std::shared_ptr` 内部维护一个引用计数，记录有多少个 `std::shared_ptr` 共享同一个对象。每当一个新的 `std::shared_ptr` 指向该对象时，引用计数加 1；当一个 `std::shared_ptr` 被销毁或重置时，引用计数减 1。当引用计数变为 0 时，对象的内存会被自动释放。
- **共享所有权**：多个 `std::shared_ptr` 可以同时拥有同一个对象的所有权，这使得资源可以在多个地方被安全地使用，而不用担心资源过早释放或重复释放的问题。
- **自动资源管理**：`std::shared_ptr` 会在引用计数变为 0 时自动释放所管理的资源，无需手动调用 `delete` 操作符，提高了代码的安全性和可维护性。
- **可复制和赋值**：`std::shared_ptr` 支持复制构造和赋值操作，复制或赋值操作会增加引用计数，确保资源的共享和正确管理。
- **自定义删除器**：可以通过模板参数指定自定义的删除器，用于定义释放资源的特定方式，以满足不同资源类型的特殊释放需求。

## 使用场景

- **多个对象共享资源**：当多个对象需要同时访问和使用同一个资源时，使用 `std::shared_ptr` 可以方便地实现资源的共享。例如，多个线程可能需要访问同一个数据结构，使用 `std::shared_ptr` 可以确保该数据结构在所有线程都不再使用时才被释放。
- **实现对象池**：在对象池的实现中，`std::shared_ptr` 可以用于管理对象的生命周期。当对象从对象池中取出时，使用 `std::shared_ptr` 管理该对象；当对象被放回对象池或不再使用时，`std::shared_ptr` 会自动释放对象的内存。

## make_shared

`std::make_shared` 是 C++11 中引入的一个用于创建 `std::shared_ptr` 对象的函数模板。它通过单次内存分配同时创建被管理的对象和控制块（control block），从而提高效率并减少潜在的内存碎片。

| 特点 | `std::make_shared` | `shared_ptr`和`new` |
| - | - | - |
| **内存分配** | 一次分配，控制块和对象共享内存。 | 两次分配，控制块和对象分开存储。 |
| **异常安全** | 构造失败无内存泄漏（更安全）。 | 构造失败时可能导致对象泄漏。 |
| **代码简洁性** | 简洁，无需显式 `new`。 | 需要显式使用 `new`，易出错。 |
| **自定义删除器** | 不支持自定义删除器。 | 支持自定义删除器（适合特殊资源管理）。 |


```C++
auto sp = std::make_shared<int>(9); // 同时分配对象和控制块
```

## 模拟实现

```C++
template <typename T>
class SharedPtr {
public:
    explicit SharedPtr(T* ptr = nullptr) 
        : ptr_(ptr), ref_count_(ptr ? new std::atomic<size_t>(1) : nullptr) {}
    
    SharedPtr(const SharedPtr& other) : ptr_(other.ptr_), ref_count_(other.ref_count_) {
        if (ref_count_)
            ++(*ref_count_);
    }
    
    SharedPtr& operator=(const SharedPtr& other) {
        if (this != other) {
            release();
            ptr_ = other.ptr_;
            ref_count_ = other.ref_count_;
            if (ref_count_)
                	ref_count_->fetch_a;
        }
        return *this;
    }
    
    T& operator*() { return *ptr_; }
    T* operator->() { return ptr_; }
    
    ~SharedPtr() { release(); }
    
    T* get() const { return ptr_; }
    
    int use_count() const {
        return ref_count_ ? ref_count_->load() : 0;
    }
    
private:
    void release() {
        if (ref_count_ && ref_count_->fetch_sub(1) == 0) {
            delete ptr_;
            delete ref_count_;
        }
    }
    
    T* ptr_;
    std::atomic<size_t>* ref_count_;
};
```

# weak_ptr

`std::weak_ptr` 是 C++ 标准库中的一种智能指针，它是为了配合 `std::shared_ptr` 而引入的，用于解决 `std::shared_ptr` 可能存在的循环引用。当多个 `std::shared_ptr` 相互引用形成循环时，它们的引用计数永远不会降为 0，导致对象无法被释放，而 `std::weak_ptr` 不影响引用计数，可作为一种弱引用解决此问题。

```C++
#include <iostream>
#include <memory>

class B; // 前向声明

class A {
public:
    std::shared_ptr<B> b_ptr;
    ~A() { std::cout << "A destroyed\n"; }
};

class B {
public:
    std::weak_ptr<A> a_ptr; // 使用 weak_ptr 避免循环引用
    ~B() { std::cout << "B destroyed\n"; }
};

int main() {
    auto a = std::make_shared<A>();
    auto b = std::make_shared<B>();
    a->b_ptr = b;
    b->a_ptr = a;

    // 此时 a 和 b 的引用计数都为 1，不会造成循环引用
    std::cout << "a use_count: " << a.use_count() << "\n"; // 输出 1
    std::cout << "b use_count: " << b.use_count() << "\n"; // 输出 1

    return 0;
}
```

