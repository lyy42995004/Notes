# Unix I/O

**了解 Unix I/O 将帮助你理解其他的系统概念。**I/O 是系统操作不可或缺的一部分。我们经常遇到 I/O 和其他系统概念之间的循环依赖。例如，I/O 在进程的创建和执行中扮演着关键的角色。反过来，进程创建又在不同进程间的文件共享中扮演着关键角色。因此，要真正理解 I/O，你必须理解进程，反之亦然。在对存储器层次结构、链接和加载、进程以及虚拟内存的讨论中，我们已经接触了 I/O 的某些方面。对这些概念有了比较好的理解，我们才能闭合这个循环，更加深入地研究 I/O。

**有时你除了使用 Unix I/O 以外别无选择。**在某些重要的情况中，使用高级 I/O 函数不太可能，或者不太合适。例如，标准 I/O 库没有提供读取文件元数据的方式，例如文件大小或文件创建时间。另外，I/O 库还存在一些问题，使得用它来进行网络编程非常冒险。

Linux 文件可看作是由 m 个字节构成的序列。在 Linux 系统中，所有 I/O 设备（如网络、磁盘、终端等）都被模型化为文件，输入和输出被当作对相应文件的读写来操作，基于此引出了名为 Unix I/O 的简单、低级应用接口，能让输入输出以统一且一致的方式执行。

- **打开文件**：应用程序向内核申请打开对应文件，内核会返回一个称作描述符的小非负整数，用于后续标识该文件，应用程序只需记住此描述符即可，且 Linux shell 创建的每个进程初始有三个已打开的文件，即标准输入（描述符为 0，对应常量 **STDIN_FILENO**）、标准输出（描述符为 1，对应常量 **STDOUT_FILENO**）、标准错误（描述符为 2，对应常量 **STDERR_FILENO**）。
- **改变当前文件位置**：每个打开的文件，内核会维持一个初始为 0 的文件位置 k（字节偏移量），应用程序可通过执行 `seek` 操作来显式设置当前文件位置。
- **读写文件**：读操作是从文件当前位置 k 开始复制 n>0 个字节到内存，并将 k 更新为 k + n，当 k≥m（文件大小为 m 字节）时读操作会触发 EOF 条件，文件结尾无明确 “EOF 符号”；写操作则是从内存复制 n>0 个字节到文件，同样从当前位置 k 开始并更新 k。
- **关闭文件**：应用完成文件访问后通知内核关闭文件，内核会释放相关数据结构，将描述符恢复到可用池中，并且进程无论因何种原因终止，内核都会自动关闭其所有打开文件并释放相应内存资源。

# 文件

**Linux 文件类型**：

- **普通文件**：包含任意数据，应用程序会区分文本文件（仅含 ASCII 或 Unicode 字符的普通文件）和二进制文件（除此之外的普通文件），但内核对二者无区别。Linux 文本文件由以新行符（“\n”，与 ASCII 换行符 LF 即 0x0a 相同）结尾的文本行序列构成。
- **目录**：包含一组链接，可将文件名映射到文件（可能是另一个目录），每个目录至少含指向自身及父目录的两个条目，能用 `mkdir` 命令创建、`ls` 查看内容、`rmdir` 删除。
- **套接字**：用于和另一个进程进行跨网络通信的文件。
- **其他文件类型**：包含命名通道、符号链接以及字符和块设备等。

Linux 内核将所有文件都组织成一个**目录层次结构**（directory hierarchy），由名为 /（斜杠）的根目录确定。系统中的每个文件都是根目录的直接或间接的后代。

<img src="https://s2.loli.net/2024/12/04/4juq5cwTgCvL6RK.png" alt="image.png" style="zoom: 67%;" />

<center> Linux 目录层次的一部分</center>

作为其上下文的一部分，每个进程都有一个**当前工作目录**（current working directory）来确定其在目录层次结构中的当前位置。你可以用 cd 命令来修改 shell 中的当前工作目录。

**路径名**：用于指定目录层次结构中的位置，分两种形式：

- **绝对路径名**：以斜杠开始，代表从根节点开始的路径，如 `/home/droh/hello.c`。
- **相对路径名**：以文件名开始，表示从当前工作目录出发的路径，其形式会随当前工作目录不同而变化，例如当前工作目录为 `/home/droh` 时，相对路径名可为`./hello.c`；若当前工作目录为 `/home/bryant`，则相对路径名可为`../home/droh/hello.c`。

# 打开和关闭文件

进程借助`open`函数来打开已存在文件或创建新文件。open 函数将 filename 转换为一个文件描述符，并且返回描述符数字。返回的描述符总是在进程中**当前没有打开的最小描述符**。

```C
#include <sys/types.h>
#include <sys/stat.h>
#inlcude <fcnth.h>

int open(char* filename, int flags, mode_t mode);
// 返回：若成功则为新文件描述符，若出错为 -1。
```

**flags** 指明了进程打算如何访问这个文件：

- `O_RDONLY`：只读方式访问文件。
- `O_WRONLY`：只进行写操作访问文件。
- `O_RDWR`：可读可写文件。

- `O_CREAT`：若文件不存在，会创建一个截断的（空）文件。
- `O_TRUNC`：若文件已存在，则对其进行截断操作。
- `O_APPEND`：每次写操作前，会将文件位置设置到文件结尾处。

**umask **：每个进程都有一个`umask`，通过`umask`函数设置。当进程用带`mode`参数的`open`函数创建新文件时，文件访问权限位被设置为`mode & ~umask`。

```C
#define DEF_MODE   S_IRUSR|S_IWUSR|S_IRGRP|S_IWGRP|S_IROTH|S_IWOTH
#define DEF_UMASK  S_IWGRP|S_IWOTH
```

例如`DEF_MODE`和`DEF_UMASK`默认值示例，可以据此创建出拥有者有读写权限、其他用户有读权限的新文件。

| 掩码    | 描述                               |
| ------- | ---------------------------------- |
| S_IRUSR | 使用者（拥有者）能够读这个文件     |
| S_IWUSR | 使用者（拥有者）能够写这个文件     |
| S_IXUSR | 使用者（拥有者）能够执行这个文件   |
| S_IRGRP | 拥有者所在组的成员能够读这个文件   |
| S_IWGRP | 拥有者所在组的成员能够写这个文件   |
| S_IXGRP | 拥有者所在组的成员能够执行这个文件 |
| S_IROTH | 其他人（任何人）能够读这个文件     |
| S_IWOTH | 其他人（任何人）能够写这个文件     |
| S_IXOTH | 其他人（任何人）能够执行这个文件   |

```C
#include <unistd.h>

int close(int fd);
// 返回：若成功则为 0，若出错则为 -1。
```

进程通过调用`close`函数关闭打开的文件，关闭已关闭的描述符会出现错误情况。

# 读和写文件

应用程序是通过分别调用 read 和 write 函数来执行输入和输出的。

```C
#include <unistd.h>

ssize_t read(int fd, void *buf, size_t n);
// 返回：若成功则为读的字节数，若 EOF 则为0，若出错为 -1。

ssize_t write(int fd, const void *buf, size_t n);
// 返回：若成功则为写的字节数，若出错则为 -1。
```

`read` 函数从描述符为 fd 的当前文件位置复制最多 n 个字节到内存位置 buf。返回值 -1 表示一个错误，而返回值 0 表示 EOF。否则，返回值表示的是实际传送的字节数量。

`write` 函数从内存位置 buf 复制至多 n 个字节到描述符 fd 的当前文件位置。 下面展示了一个程序使用 `read` 和 `write` 调用一次一个字节地从标准输入复制到标准输出。

```C
int main(void)
{
    char c;
    while(Read(STDIN_FILENO, &c, 1) != 0)
        Write(STDOUT_FILENO, &c, 1);
    exit(0);
}
```

> **ssize_t 和 size_t 类型区别**
>
> 在 x86-64 系统中，`size_t`被定义为`unsigned long`，是无符号类型；而`ssize_t`（有符号的大小）被定义为`long`，是有符号类型。`read`函数采用`size_t`作为输入参数（表示要读取的字节数等情况），用`ssize_t`作为返回值，之所以返回有符号值是因为出错时需要返回 -1，而这使得`read`函数可返回的最大值减小了一半。

**读、写操作出现不足值（short count）情况及原因**：

1. **读时遇到 EOF**：比如读取一个文件，从当前位置起所含字节数小于应用程序要求读取的字节数时，会出现不足值情况，后续再读会通过返回不足值 0 来表示遇到 EOF，例如准备读 50 字节但实际只剩 20 多字节的情况，首次读返回不足值 20，后续读返回 0 表示 EOF。
2. **从终端读文本行**：当打开文件与终端相关联（像键盘和显示器），每次`read`函数通常一次传送一个文本行，返回的不足值就等于该文本行大小。
3. **读和写网络套接字（socket）**：若打开文件对应网络套接字（详见 11.4 节），受内部缓冲约束和较长网络延迟影响，`read`和`write`会返回不足值；对 Linux 管道（pipe）调用`read`和`write`也可能出现不足值，但管道这种进程间通信机制不在当前讨论范围。

在读磁盘文件时除了遇到 EOF 外基本不会遇到不足值，写磁盘文件时通常也不会出现不足值，但创建像 Web 服务器这类可靠的网络应用时，就需要反复调用`read`和`write`来处理不足值，直至所有要求的字节都传送完毕。

# RIO包

**RIO（Robust I/O，健壮的 I/O）**的 I/O 包，它能自动处理前文提到的读、写操作中出现的不足值情况，在容易产生不足值的应用（如网络程序）里，可提供方便、健壮且高效的 I/O 操作。

RIO 包提供的函数类型：

- **无缓冲的输入输出函数**：此类函数直接在内存与文件之间传输数据，不存在应用级缓冲，对于网络中二进制数据的读写操作特别有用。

- **带缓冲的输入函数**：这些函数能够让使用者高效地从文件里读取文本行以及二进制数据，文件内容会缓存在应用级缓冲区内，类似标准 I/O 函数（如 printf）的缓冲区。不过它与其他带缓冲的 I/O 例程有所不同，带缓冲的 RIO 输入函数是线程安全的，在同一个描述符上能够交错调用，比如可以先从一个描述符读一些文本行，接着读一些二进制数据，然后再读一些文本行。

## RIO 的无缓冲的输入输出函数

应用程序可借助`rio_readn`和`rio_writen`函数在内存与文件之间直接传送数据，这两个函数包含在`"csapp.h"`头文件中。[CSAPP官网代码](https://csapp.cs.cmu.edu/3e/code.html)

```C
#include "csapp.h"

ssize_t rio_readn(int fd, void *usrbuf, size_t n);
ssize_t rio_writen(int fd, void *usrbuf, size_t n);
// 返回：若成功则为传送的字节数，若 EOF 则为 0(只对 rio_readn 而言)，若出错则为 -1。
```

`rio_readn` 函数从描述符 fd 的当前文件位置最多传送 `n` 个字节到内存位置 usrbuf。`rio_readn` 函数在遇到 EOF 时只能返回一个不足值。`rio_writen` 函数从位置 `usrbuf` 传送 `n` 个字节到描述符 fd。`rio_writen` 函数决不会返回不足值。对同一个描述符，可以任意交错地调用 `rio_readn` 和 `rio_writen`。

如果 `rio_readn` 和 `rio_writen` 函数被一个从应用信号处理程序的返回中断，那么每个函数都会手遍地重启 `read` 或 `write`。为了尽可能有较好的可移植性，我们允许被中断的系统调用，且在必要时重启它们。

```C
ssize_t rio_readn(int fd, void *usrbuf, size_t n)
{
    size_t nleft = n;
    ssize_t nread;
    char *bufp = usrbuf;

    while (nleft > 0) {
        if ((nread = read(fd, bufp, nleft)) < 0) {
            if (errno == EINTR) /* Interrupted by sig handler return */
                nread = 0;      /* and call read() again */
            else
                return -1;      /* errno set by read() */
        }
        else if (nread == 0)
            break;              /* EOF */
        nleft -= nread;
        bufp += nread;
    }
    return (n - nleft);         /* Return >= 0 */
}
```

```C
ssize_t rio_writen(int fd, void *usrbuf, size_t n)
{
    size_t nleft = n;
    ssize_t nwritten;
    char *bufp = usrbuf;

    while (nleft > 0) {
        if ((nwritten = write(fd, bufp, nleft)) <= 0) {
            if (errno == EINTR)  /* Interrupted by sig handler return */
                nwritten = 0;    /* and call write() again */
            else
                return -1;       /* errno set by write() */
        }
        nleft -= nwritten;
        bufp += nwritten;
    }
    return n;
}
```

## RIO 的带缓冲的输入函数

若要编写计算文本文件中文本行数量的程序，一种做法是用`read`函数逐字节从文件传送到用户内存，再检查每个字节找换行符，但此方法效率低，因为**每读取一个字节都需陷入内核**。一种更好的方法是调用一个包装函数（`rio_readlineb`），它从内部读缓冲区复制文本行，缓冲区变空时会自动调用`read`重新填满。对于包含文本行和二进制数据的文件，还有带缓冲区版本的`rio_readn`即`rio_readnb`，能从相同读缓冲区传送原始字节。

```C
#include "csapp.h"

void rio_readinitb(rio_t *rp, int fd);
// 返回：无。

ssize_t rio_readlineb(rio_t *rp, void *usrbuf, size_t maxlen);
ssize_t rio_readnb(rio_t *rp, void *usrbuf, size_t n);
// 返回：若成功则为读的字节数，若 EOF 则为 0，若出错则为 -1。 
```

**rio_readinitb 函数**：打开一个描述符时调用该函数，它将描述符`fd`和地址`rp`处类型为`rio_t`的读缓冲区建立联系。

**rio_readlineb 函数**：从文件读出下一个文本行（含结尾换行符），复制到内存位置`usrbuf`，并用`NULL`字符结束文本行。最多读`maxlen - 1`个字节，超长文本行截断并用`NULL`字符结束。

**rio_readnb 函数**：从文件最多读`n`个字节到内存位置`usrbuf`。

一个读缓冲区的格式，以及初始化它的 `rio_readinitb` 函数的代码。`rio_readinitb` 函数创建了一个空的读缓冲区，并且将一个打开的文件描述符和这个缓冲区联系起来。

```C
// csapp.h
#define RIO_BUFSIZE 8192
typedef struct {
    int rio_fd;                /* Descriptor for this internal buf */
    int rio_cnt;               /* Unread bytes in internal buf */
    char *rio_bufptr;          /* Next unread byte in internal buf */
    char rio_buf[RIO_BUFSIZE]; /* Internal buffer */
} rio_t;
```

```C
void rio_readinitb(rio_t *rp, int fd)
{
    rp->rio_fd = fd;
    rp->rio_cnt = 0;
    rp->rio_bufptr = rp->rio_buf;
}
```

`rio_read`函数是`Linux read`函数的带缓冲版本，处于`RIO`读程序的核心地位。当应用程序调用它要求读取`n`个字节时，它会先查看读缓冲区（通过`rp->rio_cnt`判断）内剩余未读字节数量。

```C
static ssize_t rio_read(rio_t *rp, char *usrbuf, size_t n)
{
    int cnt;

    while (rp->rio_cnt <= 0) {  /* Refill if buf is empty */
        rp->rio_cnt = read(rp->rio_fd, rp->rio_buf,
                           sizeof(rp->rio_buf));
        if (rp->rio_cnt < 0) {
            if (errno != EINTR) /* Interrupted by sig handler return */
                return -1;
        }
        else if (rp->rio_cnt == 0)  /* EOF */
            return 0;
        else
            rp->rio_bufptr = rp->rio_buf; /* Reset buffer ptr */
    }

    /* Copy min(n, rp->rio_cnt) bytes from internal buf to user buf */
    cnt = n;
    if (rp->rio_cnt < n)
        cnt = rp->rio_cnt;
    memcpy(usrbuf, rp->rio_bufptr, cnt);
    rp->rio_bufptr += cnt;
    rp->rio_cnt -= cnt;
    return cnt;
}
```

```C
ssize_t rio_readlineb(rio_t *rp, void *usrbuf, size_t maxlen)
{
    int n, rc;
    char c, *bufp = usrbuf;

    for (n = 1; n < maxlen; n++) {
        if ((rc = rio_read(rp, &c, 1)) == 1) {
            *bufp++ = c;
            if (c == ’\n’) {
                n++;
                break;
            }
        } else if (rc == 0) {
            if (n == 1)
                return 0; /* EOF, no data read */
            else
                break;    /* EOF, some data was read */
        } else
            return -1;    /* Error */
    }
    *bufp = 0;
    return n - 1;
}
```

```C
ssize_t rio_readnb(rio_t *rp, void *usrbuf, size_t n)
{
    size_t nleft = n;
    ssize_t nread;
    char *bufp = usrbuf;

    while (nleft > 0) {
        if ((nread = rio_read(rp, bufp, nleft)) < 0)
            return -1;          /* errno set by read() */
        else if (nread == 0)
            break;              /* EOF */
        nleft -= nread;
        bufp += nread;
    }
    return (n - nleft);         /* Return >= 0 */	
}
```

下面展示了如何使用 RIO 函数来一次一行地从标准输入复制一个文本文件到标准输出。

```C
#include "csapp.h"

int main(int argc, char **argv)
{
    int n;
    rio_t rio;
    char buf[MAXLINE];

    Rio_readinitb(&rio, STDIN_FILENO);
    while ((n = Rio_readlineb(&rio, buf, MAXLINE)) != 0)
        Rio_writen(STDOUT_FILENO, buf, n);
}
```

# 读文件元数据

```C
#include <unistd.h>
#include <sys/stat.h>

int stat(const char *filename, struct stat *buf);
int fstat(int fd, struct stat *buf);
// 返回：若成功则为 0，若出错则为 -1。
```

stat 函数以一个文件名作为输入，并填写下面所示的一个 stat 数据结构中的各个成员。

fstat 函数是类似的，只不过是以文件描述符而不是文件名作为输入。

```C
/* Metadata returned by the stat and fstat functions */
struct stat {
    dev_t         st_dev;      /* Device */
    ino_t         st_ino;      /* inode */
    mode_t        st_mode;     /* Protection and file type */
    nlink_t       st_nlink;    /* Number of hard links */
    uid_t         st_uid;      /* User ID of owner */
    gid_t         st_gid;      /* Group ID of owner */
    dev_t         st_rdev;     /* Device type (if inode device) */
    off_t         st_size;     /* Total size, in bytes */
    unsigned long st_blksize;  /* Block size for filesystem I/O */
    unsigned long st_blocks;   /* Number of blocks allocated */
    time_t        st_atime;    /* Time of last access */
    time_t        st_mtime;    /* Time of last modification */
    time_t        st_ctime;    /* Time of last change */
};
```

`st_size` 成员包含了文件的字节数大小。`st_mode` 成员则编码了文件访问许可位和文件类型。`Linux` 在 `sys/stat.h` 中定义了**宏**来确定 `st_mode` 成员的文件类型：

- `S_ISREG(m)`：用于判断是否为普通文件。
- `S_ISDIR(m)`：用于判断是否为目录文件。
- `S_ISSOCK(m)`：用于判断是否为网络套接字。

下面展示了如何使用这些宏和 stat 函数来读取和解释一个文件的 `st_mode` 位。

```C
int main (int argc, char **argv)
{
    struct stat stat;
    char *type, *readok;

    Stat(argv[1], &stat);
    if (S_ISREG(stat.st_mode))     /* Determine file type */
        type = "regular";
    else if (S_ISDIR(stat.st_mode))
        type = "directory";
    else
        type = "other";
    if ((stat.st_mode & S_IRUSR))  /* Check read access */
        readok = "yes";
    else
        readok = "no";

    printf("type: %s, read: %s\n", type, readok);
    exit(0);
}
```

# 读目录内容

```C
#include <sys/types.h>
#include <dirent.h>

DIR *opendir(const char *name);
// 返回：若成功，则为处理的指针；若出错，则为 NULL。
```

函数 `opendir` 以路径名为参数，返回指向**目录流**（directory stream）的指针。流是对条目有序列表的抽象，在这里是指目录项的列表。

```C
#include <dirent.h>

struct dirent *readdir(DIR *dirp);
// 返回：若成功，则为指向下一个目录项的指针；
//      若没有更多的目录项或出错，则为 NULL。
```

对 readdir 的调用返回的都是指向流 dirp 中下一个目录项的指针，或者，如果没有更多目录项则返回 NULL。每个目录项都是一个结构，其形式如下：成员 `d_name` 是文件名，`d_ino` 是文件位置。

```C
struct dirent {
    ino_t d_ino;      /* inode number */
    char d_name[256]; /* Filename */
};
```

函数 closedir 关闭流并释放其所有的资源。

```C
#include <dirent.h>

int closedir(DIR *dirp);
// 返回：成功为 0；错误为 -1。
```

下面展示了怎样用 readdir 来读取目录的内容。

```C
int main(int argc, char **argv)
{
    DIR *streamp;
    struct dirent *dep;

    streamp = Opendir(argv[1]);

    errno = 0;
    while ((dep = readdir(streamp)) != NULL) {
        printf("Found file: %s\n", dep->d_name);
    }
    if (errno != 0)
        unix_error("readdir error");

    Closedir(streamp);
    exit(0);
}
```

# 共享文件

核用三个相关的数据结构来表示打开的文件：

- **描述符表**（descriptor table）：每个进程都有它独立的描述符表，它的表项是由进程打开的文件描述符来索引的。每个打开的描述符表项指向文件表中的一个表项。
- **文件表**（file table）：打开文件的集合是由一张文件表来表示的，所有的进程共享这张表。每个文件表的表项组成（针对我们的目的）包括当前的文件位置、**引用计数**（reference count）（即当前指向该表项的描述符表项数），以及一个指向 v-node 表中对应表项的指针。关闭一个描述符会减少相应的文件表表项中的引用计数。内核不会删除这个文件表表项，直到它的引用计数为零。
- **v-node 表**（v-node table）：同文件表一样，所有的进程共享这张 v-node 表。每个表项包含 stat 结构中的大多数信息，包括 st_mode 和 st_size 成员。

<img src="https://s2.loli.net/2024/12/04/b97XpEMQVrOTLwf.png" alt="image _2_.png" style="zoom:67%;" />

<center>一种典型的情况，没有共享文件，并且每个描述符对应一个不同的文件。</center>

<img src="https://s2.loli.net/2024/12/04/1OumSoPQiLD69xf.png" alt="image _3_.png" style="zoom:67%;" />

<center>两个描述符通过两个打开文件表表项共享同一个磁盘文件</center>

<img src="https://s2.loli.net/2024/12/04/TdnMDGYWwmopRrK.png" alt="image _4_.png" style="zoom:67%;" />

<center>子进程如何继承父进程的打开文件</center>

# I/O重定向

Linuxshell 提供了 I/O 重定向操作符，允许用户将磁盘文件和标准输入输出联系起来。例如，键入
`linux> ls > foo.txt`使得 shell 加载和执行 Is 程序，将标准输出重定向到磁盘文件 foo.txt。

一种方式是使用 dup2 函数。

```C
#include <unistd.h>

int dup2(int oldfd, int newfd);
// 返回：若成功则为非负的描述符，若出错则为 -1。
```

dup2 函数复制描述符表表项 oldfd 到描述符表表项 newfd，覆盖描述符表表项 newfd 以前的内容。如果 newfd 已经打开了，dup2 会在复制 oldfd 之前关闭 newfd。

调用`dup2(4,1)`前，描述符 1 对应文件 A（如终端）、描述符 4 对应文件 B（如磁盘文件），二者引用计数均为 1。调用后，描述符 1、4 都指向文件 B，文件 A 被关闭且其相关表项被删，文件 B 引用计数增加，此后标准输出的数据会重定向到文件 B。

<img src="https://s2.loli.net/2024/12/04/ynIwtof8UWD7z5X.png" alt="image _1_.png" style="zoom: 67%;" />
