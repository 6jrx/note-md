在Linux中，FIFO（命名管道）是一种特殊的文件类型，允许不相关的进程进行通信。下面详细介绍FIFO相关的函数、使用方法和注意事项。

## 1. FIFO基本概念

FIFO（First In First Out）也称为命名管道，与普通管道（pipe）的主要区别在于：

- **普通管道**：只能用于有亲缘关系的进程间通信（如父子进程）
- **命名管道**：可用于系统中任意两个进程间的通信，通过文件系统中的路径名标识

FIFO在文件系统中以文件名的形式存在，但数据存储在内存中，不占用磁盘空间。

## 2. FIFO相关函数详解

### 2.1 创建FIFO：mkfifo()函数

**函数原型**：

```
#include <sys/types.h>
#include <sys/stat.h>
int mkfifo(const char *pathname, mode_t mode);
```

**参数说明**：

- `pathname`：FIFO文件的路径名
- `mode`：文件权限（如0666）

**返回值**：成功返回0，失败返回-1

**示例代码**：

```
#include <sys/types.h>
#include <sys/stat.h>
#include <stdio.h>
#include <stdlib.h>

int main() {
    if (mkfifo("/tmp/my_fifo", 0666) != 0) {
        perror("mkfifo失败");
        exit(EXIT_FAILURE);
    }
    printf("FIFO创建成功\n");
    return 0;
}
```

**替代创建函数**：`mknod()`（较老的方法）

```
int mknod(const char *pathname, mode_t mode | S_IFIFO, (dev_t)0);
```

### 2.2 打开FIFO：open()函数

打开FIFO时有四种常见方式，行为各不相同：

| 打开方式   | 行为描述                               |
| ---------- | -------------------------------------- |
| `O_RDONLY` | 阻塞，直到有进程以写方式打开同一个FIFO |
| `O_WRONLY` | 阻塞，直到有进程以读方式打开同一个FIFO |
| `O_RDONLY  | O_NONBLOCK`                            |
| `O_WRONLY  | O_NONBLOCK`                            |

**重要限制**：不能以`O_RDWR`（读写）模式打开FIFO，因为其行为未定义。

### 2.3 命令行创建FIFO

除了函数调用，也可以在shell中直接创建FIFO：

```
mkfifo filename                 # 创建FIFO
mkfifo -m 666 myfifo           # 指定权限模式
mknod filename p               # 较老的创建方式（不推荐）
```

## 3. FIFO读写操作

### 3.1 写入端示例

```
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/stat.h>
#include <string.h>

#define FIFO_PATH "/tmp/my_fifo"
#define BUFFER_SIZE 100

int main() {
    int fd;
    char buffer[BUFFER_SIZE];
    
    // 检查并创建FIFO（如果不存在）
    if (access(FIFO_PATH, F_OK) == -1) {
        if (mkfifo(FIFO_PATH, 0666) == -1) {
            perror("mkfifo");
            exit(EXIT_FAILURE);
        }
    }
    
    // 以只写方式打开FIFO（阻塞）
    fd = open(FIFO_PATH, O_WRONLY);
    if (fd == -1) {
        perror("open");
        exit(EXIT_FAILURE);
    }
    
    for (int i = 0; i < 5; i++) {
        snprintf(buffer, BUFFER_SIZE, "消息 %d from PID: %d", i, getpid());
        ssize_t bytes_written = write(fd, buffer, strlen(buffer) + 1);
        if (bytes_written == -1) {
            perror("write");
            break;
        }
        printf("发送: %s\n", buffer);
        sleep(1);
    }
    
    close(fd);
    return 0;
}
```

### 3.2 读取端示例

```
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/stat.h>

#define FIFO_PATH "/tmp/my_fifo"
#define BUFFER_SIZE 100

int main() {
    int fd;
    char buffer[BUFFER_SIZE];
    
    // 以只读方式打开FIFO（阻塞）
    fd = open(FIFO_PATH, O_RDONLY);
    if (fd == -1) {
        perror("open");
        exit(EXIT_FAILURE);
    }
    
    printf("等待接收数据...\n");
    ssize_t bytes_read;
    while ((bytes_read = read(fd, buffer, BUFFER_SIZE)) > 0) {
        printf("接收: %s\n", buffer);
    }
    
    if (bytes_read == -1) {
        perror("read");
    }
    
    close(fd);
    return 0;
}
```

## 4. 重要注意事项

### 4.1 阻塞行为处理

- **读取阻塞**：当FIFO为空时，读进程会阻塞直到有数据写入
- **写入阻塞**：当FIFO已满时，写进程会阻塞直到有空间可用
- **非阻塞模式**：使用`O_NONBLOCK`标志可避免阻塞

### 4.2 原子性写入

为了保证数据完整性，应注意：

- 当写入数据长度**小于等于**`PIPE_BUF`（通常是4096字节）时，Linux保证写入操作的原子性
- 多个进程同时写入时，小于`PIPE_BUF`的数据块不会交错

### 4.3 错误处理重点

```
// 处理SIGPIPE信号（当读端关闭时）
#include <signal.h>
signal(SIGPIPE, SIG_IGN);  // 忽略SIGPIPE信号

// 或者捕获信号自定义处理
void sigpipe_handler(int sig) {
    printf("管道断裂，读端已关闭\n");
    // 清理资源
    exit(EXIT_FAILURE);
}
signal(SIGPIPE, sigpipe_handler);
```

### 4.4 多进程同步

```
// 使用select实现多路复用
#include <sys/select.h>

fd_set read_fds;
struct timeval timeout;
int fd = open(FIFO_PATH, O_RDONLY | O_NONBLOCK);

FD_ZERO(&read_fds);
FD_SET(fd, &read_fds);
timeout.tv_sec = 5;  // 5秒超时

int retval = select(fd + 1, &read_fds, NULL, NULL, &timeout);
if (retval == -1) {
    perror("select");
} else if (retval) {
    printf("有数据可读\n");
    // 读取数据
} else {
    printf("超时，无数据可读\n");
}
```

## 5. 完整的生产者-消费者示例

### 生产者程序

```
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <fcntl.h>
#include <sys/stat.h>
#include <limits.h>

#define FIFO_NAME "/tmp/comm_fifo"
#define BUFFER_SIZE PIPE_BUF

int main() {
    int pipe_fd;
    int bytes_sent = 0;
    char buffer[BUFFER_SIZE + 1];
    
    if (access(FIFO_NAME, F_OK) == -1) {
        if (mkfifo(FIFO_NAME, 0777) != 0) {
            fprintf(stderr, "无法创建FIFO\n");
            exit(EXIT_FAILURE);
        }
    }
    
    printf("进程 %d 以只写方式打开FIFO\n", getpid());
    pipe_fd = open(FIFO_NAME, O_WRONLY);
    
    if (pipe_fd != -1) {
        for (int i = 0; i < 10; i++) {
            snprintf(buffer, BUFFER_SIZE, "生产者消息 %d", i);
            int res = write(pipe_fd, buffer, strlen(buffer));
            if (res == -1) {
                fprintf(stderr, "写入错误\n");
                exit(EXIT_FAILURE);
            }
            bytes_sent += res;
            sleep(1);
        }
        close(pipe_fd);
    } else {
        exit(EXIT_FAILURE);
    }
    
    printf("进程 %d 完成，发送了 %d 字节\n", getpid(), bytes_sent);
    return 0;
}
```

### 消费者程序

```
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/stat.h>
#include <limits.h>

#define FIFO_NAME "/tmp/comm_fifo"
#define BUFFER_SIZE PIPE_BUF

int main() {
    int pipe_fd;
    int bytes = 0;
    char buffer[BUFFER_SIZE + 1];
    
    printf("进程 %d 以只读方式打开FIFO\n", getpid());
    pipe_fd = open(FIFO_NAME, O_RDONLY);
    memset(buffer, '\0', sizeof(buffer));
    
    if (pipe_fd != -1) {
        int res;
        do {
            res = read(pipe_fd, buffer, BUFFER_SIZE);
            bytes += res;
            printf("消费者读取: %s\n", buffer);
        } while (res > 0);
        close(pipe_fd);
    } else {
        exit(EXIT_FAILURE);
    }
    
    printf("进程 %d 完成，读取了 %d 字节\n", getpid(), bytes);
    return 0;
}
```

## 6. 最佳实践总结

1. **始终检查返回值**：所有FIFO相关函数调用都应检查返回值
2. **合理选择阻塞模式**：根据应用需求选择阻塞或非阻塞模式
3. **数据长度限制**：单次写入数据建议不超过`PIPE_BUF`以确保原子性
4. **资源清理**：使用完毕后及时关闭文件描述符并删除FIFO文件
5. **信号处理**：妥善处理`SIGPIPE`等可能的中断信号
6. **超时机制**：对于关键应用，实现超时机制避免永久阻塞

通过合理使用这些函数和注意事项，可以有效地利用FIFO实现进程间通信。