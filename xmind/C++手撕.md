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

## MyWeakPtr

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

