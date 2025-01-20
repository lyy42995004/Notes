> `Buffer`仿照的是**陈硕老师的muduo库**，用于WebServer的简单`Buffer`，学习和本篇博客参照的主要是[muduo学习笔记：net部分之实现TCP网络编程库-Buffer](https://blog.csdn.net/wanggao_1990/article/details/119426351)和[muduo库buffer源码](https://github.com/chenshuo/muduo/blob/master/muduo/net/Buffer.h)。

# 为什么需要Buffer缓冲区？

- **output buffer**：在 non-blocking 网络编程中，write () 调用可能无法一次性发送全部数据，网络库需将剩余数据暂存于 TCP 连接的 output buffer，并注册 POLLOUT 事件后续发送。若程序再次写入数据，应追加至 buffer 中，且在有未发送数据时关闭连接需等待数据发送完毕。
- **input buffer**：TCP 协议的字节流特性使接收方可能收到不完整消息，网络库需将数据读至 input buffer 等待组装成完整消息后再通知业务逻辑，以避免反复触发 POLLIN 事件。

# Buffer设计

- 对外呈现为连续内存块 `(char*, len)`
- 内部以 `vector<char>` 存储数据
- 通过 `readIndex` 和 `writeIndex` 两个下标将数据分为 `prependable`、`readable`、`writable` 三部分

![](https://s2.loli.net/2025/01/17/z9rGigU2ZSIho1e.png)

- `readable = writeIndex - readIndex`
- `writable = size() - writeIndex`
- `prependable = readIndex`

Buffer如何不断进行读写循环的，**推荐看[muduo学习笔记：net部分之实现TCP网络编程库-Buffer](https://blog.csdn.net/wanggao_1990/article/details/119426351)**，给出了很多图非常直观，**唯一区别是它有前置8字节`kCheapPrepend`**。

## 成员变量

**用`vector<char>`存储数据，vector的大小的自动增长，大小指数增长，减小内存分配次数**

**`readindex`和`writeindex`采用`atomic`；`atomic`是一种原子类型，可以保证在多线程的情况下，安全高性能得执行程序，更新变量。**

**`readIndex` 和 `writeIndex`采用`size_t`而非`char*`可以避免`vector`迭代器失效**

```C++
// vector：1自动扩容 2大小指数增长，减小内存分配次数
std::vector<char> buffer_;
// size_t而非指针（char*）避免迭代器失效
std::atomic<size_t> read_index_;
std::atomic<size_t> write_index_;
```

## ReadFD()

在栈上准备一个 `65536` 字节的 `stackbuf`，然后利用 `readv()` 来读取数据；`iovec` 有两块，第一块指向 `muduo Buffer` 中的 `writable` 字节，另一块指向栈上的 `stackbuf`。这样如果读入的数据不多，那么全部都读到 Buffer 中去了；如果长度超过 `Buffer` 的 `writable` 字节数，就会读到栈上的 `stackbuf` 里，然后程序再把 `stackbuf` 里的数据 `append` 到 `Buffer` 中。

> `readv` 是一个系统调用，它允许将数据从文件描述符（如套接字、文件等）读取到多个不连续的缓冲区中。
>
> ```C++
> #include <sys/uio.h>
> 
> ssize_t readv(int fd, const struct iovec *iov, int iovcnt);
> ```
>
> - `fd`：文件描述符，代表要读取数据的源，例如从一个套接字或文件中读取。
> - `iov`：指向 `iovec` 结构体数组的指针。`iovec` 结构体定义如下：
>
> ```c
> struct iovec {
>     void  *iov_base;  // 指向数据缓冲区的起始地址
>     size_t iov_len;  // 缓冲区的长度
> };
> ```
>
> - `iovcnt`：`iovec` 数组中元素的数量，也就是缓冲区的数量。

```C++
// 将fd的内容读到buffer_中
ssize_t Buffer::ReadFD(int fd, int* Errno) {
    char buffer[65535]; // 栈区
    int writeable_bytes = WritableBytes();
    struct iovec iov[2];
    iov[0].iov_base = WriteBegin();
    iov[0].iov_len = writeable_bytes;
    iov[1].iov_base = &buffer;
    iov[1].iov_len = sizeof(buffer);

    ssize_t len = readv(fd, iov, 2);
    if (len < 0) {
        *Errno = errno;
    } else if (static_cast<size_t>(len) <= writeable_bytes) {
        write_index_ += len;
    } else {
        write_index_ = buffer_.size();
        Append(buffer, static_cast<size_t>(len) - writeable_bytes);
    }
    return len;
}
```

## WriteFD()

直接用`write()`函数来读取`fd`对应内容，并调用`Retrieve()`修改`read_index`。

```C++
// 将buffer_的内容写到fd中
ssize_t Buffer::WriteFD(int fd, int* Errno) {
    ssize_t len = write(fd, ReadBegin(), ReadableBytes());
    if (len < 0)
        *Errno = errno;
    else
        Retrieve(len);
    return len;
}
```

## MakeSpace()

`MakeSpace` 函数的功能是确保 `Buffer` 对象有足够的空间来存储新的数据。如果`len`比`writable` 字节数和`prepend`字节数的和小，利用库函数`copy`将数据向前移，保证有`len`长度的空间用来写；如果`len`比`writable` 字节数和`prepend`字节数的和大，直接调用`vector`的resize()扩容。

```C++
// 扩展空间
void Buffer::MakeSpace(size_t len) {
    if (len > WritableBytes() + PrependableBytes()) {
        buffer_.resize(write_index_ + len);
    } else {
        size_t readable_bytes = ReadableBytes();
        std::copy(Begin() + read_index_, Begin() + write_index_, Begin());
        read_index_ = 0;
        write_index_ = readable_bytes;
        assert(readable_bytes == ReadableBytes());
    }
}
```

# Buffer类

```C++
class Buffer {
public:
    Buffer(int init_buffer_size = 1024);
    ~Buffer() = default;

    size_t ReadableBytes() const;
    size_t WritableBytes() const;
    size_t PrependableBytes() const;

    char* WriteBegin();
    const char* WriteBeginConst() const;
    const char* ReadBegin() const;

    void EnsureWriteable(size_t len);
    void HasWritten(size_t len);
    void Retrieve(size_t len);

    void RetrieveUntil(const char* end);
    void RetrieveAll();

    std::string RetrieveAsString(size_t len);
    std::string RetrieveAllAsString();

    void Append(const char* str, size_t len);
    void Append(const std::string& str);
    void Append(const void* data, size_t len);
    void Append(const Buffer& buffer);

    ssize_t ReadFD(int fd, int* Errno);
    ssize_t WriteFD(int fd, int* Errno);

private:
    char* Begin();
    const char* Begin() const;
    void MakeSpace(size_t len);

    // vector：1自动扩容 2大小指数增长，减小内存分配次数
    std::vector<char> buffer_;
    // size_t而非指针（char*）避免迭代器失效
    std::atomic<size_t> read_index_;
    std::atomic<size_t> write_index_;
};
```

**函数实现**

```C++
// vector<char>初始化，读写下标初始化
Buffer::Buffer(int init_buffer_size)
    : buffer_(init_buffer_size), read_index_(0), write_index_(0) {};

// 可读字节数：读下标 - 写下标
size_t Buffer::ReadableBytes() const {
    return write_index_ - read_index_;
}

// 可写字节数：buffer大小 - 写下标
size_t Buffer::WritableBytes() const {
    return buffer_.size() - write_index_;
}

// 前置空闲字节数：读下标
size_t Buffer::PrependableBytes() const {
    return read_index_;
}

// 写指针位置
char* Buffer::WriteBegin() {
    return &buffer_[write_index_];
}
const char* Buffer::WriteBeginConst() const{
    return &buffer_[write_index_];
}

// 读指针位置
const char* Buffer::ReadBegin() const {
    return &buffer_[read_index_];
}

// 确保可写长度
void Buffer::EnsureWriteable(size_t len) {
    if (len > WritableBytes())
        MakeSpace(len);
    assert(len <= WritableBytes());
}

// 移动写下标
void Buffer::HasWritten(size_t len) {
    write_index_ += len;
}

// 读取len长度，移动读下标
void Buffer::Retrieve(size_t len) {
    if (len < ReadableBytes())
        read_index_ += len;
    else
        RetrieveAll();
}

// 读取到end位置
void Buffer::RetrieveUntil(const char* end) {
    assert(ReadBegin() < end);
    Retrieve(end - ReadBegin());
}

// 取出所有数据
void Buffer::RetrieveAll() {
    read_index_ = write_index_ = 0;
}

// 以字符串的形式取出数据
std::string Buffer::RetrieveAsString(size_t len) {
    assert(len <= ReadableBytes());
    std::string str(ReadBegin(), len);
    Retrieve(len);
    return str;

}

// 以字符串的形式取出所有数据
std::string Buffer::RetrieveAllAsString() {
    return RetrieveAsString(ReadableBytes());
}

// 添加数据到buffer_中
void Buffer::Append(const char* str, size_t len) {
    assert(str);
    EnsureWriteable(len);
    std::copy(str, str + len, WriteBegin());
    HasWritten(len);
}
void Buffer::Append(const std::string& str) {
    Append(str.c_str(), str.size());
}
void Buffer::Append(const void* data, size_t len) {
    Append(static_cast<const char*>(data), len);
}
void Buffer::Append(const Buffer& buffer) {
    Append(buffer.ReadBegin(), buffer.ReadableBytes());
}

// 将fd的内容读到buffer_中
ssize_t Buffer::ReadFD(int fd, int* Errno) {
    char buffer[65535]; // 栈区
    int writeable_bytes = WritableBytes();
    struct iovec iov[2];
    iov[0].iov_base = WriteBegin();
    iov[0].iov_len = writeable_bytes;
    iov[1].iov_base = &buffer;
    iov[1].iov_len = sizeof(buffer);

    ssize_t len = readv(fd, iov, 2);
    if (len < 0) {
        *Errno = errno;
    } else if (static_cast<size_t>(len) <= writeable_bytes) {
        write_index_ += len;
    } else {
        write_index_ = buffer_.size();
        Append(buffer, static_cast<size_t>(len) - writeable_bytes);
    }
    return len;
}

// 将buffer_的内容写到fd中
ssize_t Buffer::WriteFD(int fd, int* Errno) {
    ssize_t len = write(fd, ReadBegin(), ReadableBytes());
    if (len < 0)
        *Errno = errno;
    else
        Retrieve(len);
    return len;
}

// buffer_首地址
char* Buffer::Begin() {
    return &buffer_[0];
}
const char* Buffer::Begin() const {
    return &buffer_[0];
}

// 扩展空间
void Buffer::MakeSpace(size_t len) {
    if (len > WritableBytes() + PrependableBytes()) {
        buffer_.resize(write_index_ + len);
    } else {
        size_t readable_bytes = ReadableBytes();
        std::copy(Begin() + read_index_, Begin() + write_index_, Begin());
        read_index_ = 0;
        write_index_ = readable_bytes;
        assert(readable_bytes == ReadableBytes());
    }
}
```

# 测试Buffer

利用***Google Test***对`Bufer`类进行单元测试，测试对`Buffer`追加和取回数据，自动增长，内部增长。

```C++
#include "../code/buffer/buffer.h"
#include <string>
#include <gtest/gtest.h> // Google Test

using std::string;
const int init_buffer_size = 1024;

// 测试 Buffer 追加和取回功能
TEST(BufferTest, TestBufferAppendRetieve) {
    Buffer buf;
    EXPECT_EQ(buf.ReadableBytes(), 0);
    EXPECT_EQ(buf.WritableBytes(), init_buffer_size);
    EXPECT_EQ(buf.PrependableBytes(), 0);

    const string str1(200, 'x');
    buf.Append(str1);
    EXPECT_EQ(buf.ReadableBytes(), str1.size());
    EXPECT_EQ(buf.WritableBytes(), init_buffer_size - str1.size());
    EXPECT_EQ(buf.PrependableBytes(), 0);

    const string str2 = buf.RetrieveAsString(50);
    EXPECT_EQ(str2.size(), 50);
    EXPECT_EQ(buf.ReadableBytes(), str1.size() - str2.size());
    EXPECT_EQ(buf.WritableBytes(), init_buffer_size - str1.size());
    EXPECT_EQ(buf.PrependableBytes(), str2.size());
    EXPECT_EQ(str2, string(50, 'x'));

    buf.Append(str1);
    EXPECT_EQ(buf.ReadableBytes(), 2 * str1.size() - str2.size());
    EXPECT_EQ(buf.WritableBytes(), init_buffer_size - 2 * str1.size());
    EXPECT_EQ(buf.PrependableBytes(), str2.size());

    const string str3 = buf.RetrieveAllAsString();
    EXPECT_EQ(str3.size(), 350);
    EXPECT_EQ(buf.ReadableBytes(), 0);
    EXPECT_EQ(buf.WritableBytes(), init_buffer_size);
    EXPECT_EQ(buf.PrependableBytes(), 0);
    EXPECT_EQ(str3, string(350, 'x'));
}

// 测试 Buffer 增长功能
TEST(BufferTest, TestBufferGrow) {
    Buffer buf;
    buf.Append(string(400, 'y'));
    EXPECT_EQ(buf.ReadableBytes(), 400);
    EXPECT_EQ(buf.WritableBytes(), init_buffer_size - 400);

    buf.Retrieve(50);
    EXPECT_EQ(buf.ReadableBytes(), 350);
    EXPECT_EQ(buf.WritableBytes(), init_buffer_size - 400);
    EXPECT_EQ(buf.PrependableBytes(), 50);

    buf.Append(string(1000, 'z'));
    EXPECT_EQ(buf.ReadableBytes(), 1350);
    EXPECT_EQ(buf.WritableBytes(), 0);
    EXPECT_EQ(buf.PrependableBytes(), 50);

    buf.RetrieveAll();
    EXPECT_EQ(buf.ReadableBytes(), 0);
    EXPECT_EQ(buf.WritableBytes(), 1400);
    EXPECT_EQ(buf.PrependableBytes(), 0);
}


// 测试 Buffer 内部增长功能
TEST(BufferTest, TestBufferInsideGrow) {
    Buffer buf;
    buf.Append(string(800, 'y'));
    EXPECT_EQ(buf.ReadableBytes(), 800);
    EXPECT_EQ(buf.WritableBytes(), init_buffer_size - 800);

    buf.Retrieve(500);
    EXPECT_EQ(buf.ReadableBytes(), 300);
    EXPECT_EQ(buf.WritableBytes(), init_buffer_size - 800);
    EXPECT_EQ(buf.PrependableBytes(), 500);

    buf.Append(string(300, 'z'));
    EXPECT_EQ(buf.ReadableBytes(), 600);
    EXPECT_EQ(buf.WritableBytes(), init_buffer_size - 600);
    EXPECT_EQ(buf.PrependableBytes(), 0);
}

int main(int argc, char* argv[]) {
    ::testing::InitGoogleTest(&argc, argv);
    return RUN_ALL_TESTS();
}
```

**CMakeLists.txt**

```cmake
cmake_minimum_required(VERSION 3.10)
project(buffer_unit_test)

# 查找 Google Test 包
find_package(GTest REQUIRED)
# 包含 Google Test 头文件目录
include_directories(${GTEST_INCLUDE_DIRS})
# 添加可执行文件
add_executable(buffer_unit_test buffer_unit_test.cc ../code/buffer/buffer.cc)
# 链接 Google Test 库
target_link_libraries(buffer_unit_test ${GTEST_LIBRARIES} pthread)
```

