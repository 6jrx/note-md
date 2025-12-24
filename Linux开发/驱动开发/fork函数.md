## 1. fork() 函数基础

### 函数原型
```c
#include <unistd.h>

pid_t fork(void);
```

### 返回值
- **成功时**：
  - **在父进程中**：返回子进程的 PID
  - **在子进程中**：返回 0
- **失败时**：返回 -1，并设置 errno

## 2. fork() 的工作原理

### 核心机制：写时复制
现代 Linux 实现 `fork()` 时使用**写时复制** 技术：

1. **初始状态**：父进程和子进程共享相同的物理内存页
2. **读取操作**：双方都可以读取相同的内存内容
3. **写入操作**：当任一进程尝试写入时，内核会为该页创建副本，使修改独立

### 复制的内容
| 被复制的资源     | 共享的资源   |
| ---------------- | ------------ |
| 进程控制块 (PCB) | 文件描述符表 |
| 虚拟地址空间     | 打开的文件   |
| 堆、栈、数据段   | 信号处理程序 |
| 寄存器状态       | 当前工作目录 |
| 程序计数器       | 用户ID和组ID |

## 3. fork() 的执行流程

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>

int main() {
    pid_t pid;
    int count = 0;
    
    printf("Before fork: PID=%d, count=%d\n", getpid(), count);
    
    pid = fork();
    
    if (pid < 0) {
        // fork 失败
        perror("fork failed");
        return 1;
    } else if (pid == 0) {
        // 子进程代码
        count++;
        printf("Child process: PID=%d, PPID=%d, count=%d\n", 
               getpid(), getppid(), count);
    } else {
        // 父进程代码
        count--;
        printf("Parent process: PID=%d, Child PID=%d, count=%d\n", 
               getpid(), pid, count);
    }
    
    // 父子进程都会执行这里的代码
    printf("Process %d finished\n", getpid());
    return 0;
}
```

**可能的输出**：
```
Before fork: PID=1234, count=0
Parent process: PID=1234, Child PID=1235, count=-1
Process 1234 finished
Child process: PID=1235, PPID=1234, count=1
Process 1235 finished
```

## 4. fork() 的常见使用模式

### 模式1：父子进程执行不同任务
```c
pid_t pid = fork();

if (pid == 0) {
    // 子进程任务
    child_task();
    exit(0);  // 子进程必须退出
} else if (pid > 0) {
    // 父进程任务
    parent_task();
    wait(NULL);  // 等待子进程结束
} else {
    // 错误处理
    perror("fork");
}
```

### 模式2：创建守护进程
```c
pid_t pid = fork();

if (pid > 0) {
    // 父进程退出，让子进程成为孤儿进程，被init接管
    exit(0);
} else if (pid == 0) {
    // 子进程成为守护进程
    setsid();  // 创建新会话
    // 其他守护进程初始化...
    daemon_work();
}
```

### 模式3：进程池
```c
#define WORKER_NUM 4

for (int i = 0; i < WORKER_NUM; i++) {
    pid_t pid = fork();
    
    if (pid == 0) {
        // 工作进程
        worker_process(i);
        exit(0);
    } else if (pid < 0) {
        perror("fork");
        break;
    }
    // 父进程继续创建更多工作进程
}

// 父进程等待所有子进程
for (int i = 0; i < WORKER_NUM; i++) {
    wait(NULL);
}
```

## 5. fork() 的深入细节

### 文件描述符的继承
```c
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>

int main() {
    int fd = open("test.txt", O_CREAT | O_WRONLY, 0644);
    write(fd, "Before fork\n", 12);
    
    pid_t pid = fork();
    
    if (pid == 0) {
        // 子进程写入
        write(fd, "Child writing\n", 14);
        close(fd);
    } else {
        // 父进程写入
        write(fd, "Parent writing\n", 15);
        close(fd);
    }
    
    return 0;
}
```
**说明**：父子进程共享相同的文件描述符表，它们会竞争写入文件。

### 内存隔离演示
```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int global_var = 100;

int main() {
    int local_var = 200;
    pid_t pid = fork();
    
    if (pid == 0) {
        // 子进程修改变量
        global_var++;
        local_var++;
        printf("Child: global=%d, local=%d\n", global_var, local_var);
    } else {
        // 父进程等待后查看变量
        wait(NULL);
        printf("Parent: global=%d, local=%d\n", global_var, local_var);
    }
    
    return 0;
}
```
**输出**：
```
Child: global=101, local=201
Parent: global=100, local=200
```

## 6. fork() 的常见错误和陷阱

### 错误1：忘记处理返回值
```c
// 错误示例
fork();
printf("Hello\n");  // 父子进程都会执行，打印两次

// 正确做法
pid_t pid = fork();
if (pid == 0) {
    // 子进程代码
} else if (pid > 0) {
    // 父进程代码
} else {
    // 错误处理
}
```

### 错误2：子进程不退出
```c
// 错误示例
pid_t pid = fork();
if (pid == 0) {
    child_work();
    // 忘记调用 exit()，子进程会继续执行父进程的后续代码
}

// 正确做法
if (pid == 0) {
    child_work();
    exit(0);  // 明确退出
}
```

### 错误3：僵尸进程
```c
// 创建僵尸进程的示例
pid_t pid = fork();
if (pid == 0) {
    exit(0);  // 子进程立即退出
} else {
    // 父进程不调用 wait()，子进程成为僵尸
    sleep(60);  // 在此期间子进程是僵尸状态
}

// 避免僵尸进程
if (pid > 0) {
    wait(NULL);  // 等待子进程
    // 或者使用信号处理忽略SIGCHLD
    // signal(SIGCHLD, SIG_IGN);
}
```

## 7. 高级用法和技巧

### 使用 fork() 和 exec() 组合
```c
pid_t pid = fork();

if (pid == 0) {
    // 子进程执行新程序
    execl("/bin/ls", "ls", "-l", NULL);
    
    // 如果 exec 失败
    perror("execl failed");
    exit(1);
} else if (pid > 0) {
    // 父进程等待子进程完成
    int status;
    waitpid(pid, &status, 0);
    printf("Child exited with status %d\n", WEXITSTATUS(status));
}
```

### 多级 fork()
```c
printf("A: PID=%d\n", getpid());
fork();
printf("B: PID=%d\n", getpid());
fork();
printf("C: PID=%d\n", getpid());
```
**输出**（可能的顺序）：
```
A: PID=100
B: PID=100
B: PID=101
C: PID=100
C: PID=101
C: PID=102
C: PID=103
```

## 8. fork() 的性能考虑

### 影响 fork() 性能的因素
1. **进程大小**：地址空间越大，fork() 开销可能越大
2. **写时复制页数**：实际修改的页面越多，复制开销越大
3. **系统负载**：系统进程数影响调度
4. **内存压力**：内存紧张时性能下降

### 替代方案
- **vfork()**：更轻量，但子进程与父进程共享地址空间
- **clone()**：更灵活，可以控制共享哪些资源
- **pthread_create()**：创建线程而不是进程

## 9. 实际应用场景

### Web 服务器
```c
while (1) {
    int client_fd = accept(server_fd, NULL, NULL);
    pid_t pid = fork();
    
    if (pid == 0) {
        close(server_fd);  // 子进程关闭监听socket
        handle_client(client_fd);
        close(client_fd);
        exit(0);
    } else {
        close(client_fd);  // 父进程关闭客户端socket
    }
}
```

### Shell 命令执行
```c
// 模拟 shell 执行命令
pid_t pid = fork();
if (pid == 0) {
    // 子进程执行命令
    execlp("grep", "grep", "pattern", "file.txt", NULL);
    exit(1);  // 如果 exec 失败
} else {
    // 父进程等待命令完成
    int status;
    waitpid(pid, &status, 0);
    if (WIFEXITED(status)) {
        printf("Command exited with status %d\n", WEXITSTATUS(status));
    }
}
```

## 总结

`fork()` 是 Linux 进程管理的核心函数，它的关键特点：

1. **一次调用，两次返回**：在父子进程中返回不同的值
2. **写时复制**：高效的内存管理机制
3. **资源共享**：文件描述符等资源被继承
4. **执行并发**：父子进程独立执行，顺序不确定

理解 `fork()` 的机制对于编写稳定的并发程序至关重要，特别是在服务器编程、进程管理和系统工具开发中。