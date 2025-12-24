在 Linux 系统中，信号是一种非常重要的进程间通信机制，它本质上是一个异步通知，用于告知某个进程有特定事件发生。下面我们来详细了解信号的概念、使用方法及关键注意事项。

# 🔍 Linux 信号核心概念

信号是一种软件中断，它提供了一种处理异步事件的方法。当信号被发送给一个进程时，操作系统会中断进程的正常控制流程，转而处理该信号。

## 信号的生命周期

- **产生**：信号可以由内核、其他进程或进程自身产生（如硬件异常、终端按键、系统调用等）
- **递达**：信号被发送到目标进程并开始执行处理动作
- **未决**：信号已产生但尚未递达的状态，通常是因为信号被阻塞
- **处理**：进程执行信号的处理动作

## 信号的默认处理动作

每个信号都有默认处理方式，主要包括：

- **Term**：终止进程
- **Ign**：忽略信号
- **Core**：终止进程并生成核心转储文件
- **Stop**：暂停进程
- **Cont**：继续已暂停的进程

# 📋 常见信号及其含义

下表列出了 Linux 中最常用的信号及其默认行为：

| 信号名称 | 信号值 | 默认动作 | 说明                                    |
| -------- | ------ | -------- | --------------------------------------- |
| SIGHUP   | 1      | Term     | 终端挂起或控制进程终止                  |
| SIGINT   | 2      | Term     | 键盘中断（如 Ctrl+C）                   |
| SIGQUIT  | 3      | Core     | 键盘退出（如 Ctrl+\）                   |
| SIGILL   | 4      | Core     | 非法指令                                |
| SIGABRT  | 6      | Core     | 由 abort() 产生的信号                   |
| SIGFPE   | 8      | Core     | 浮点异常                                |
| SIGKILL  | 9      | Term     | **强制终止**（不可捕获、阻塞或忽略）    |
| SIGSEGV  | 11     | Core     | 无效内存访问                            |
| SIGPIPE  | 13     | Term     | 管道破裂，向无读端的管道写数据          |
| SIGALRM  | 14     | Term     | 由 alarm() 设置的定时器超时             |
| SIGTERM  | 15     | Term     | **优雅终止**信号（kill 默认发送的信号） |
| SIGCHLD  | 17     | Ign      | 子进程状态改变                          |
| SIGCONT  | 18     | Cont     | 继续已停止的进程                        |
| SIGSTOP  | 19     | Stop     | **停止进程**（不可捕获、阻塞或忽略）    |
| SIGTSTP  | 20     | Stop     | 终端停止信号（如 Ctrl+Z）               |

# 💻 信号的使用方法

## 1. 发送信号

### kill 命令

最常用的发送信号工具是 `kill`命令：

```
# 发送指定信号给进程
kill -信号编号 PID
# 示例：优雅终止进程 1234
kill -15 1234
# 示例：强制杀死进程 1234
kill -9 1234
```

### 系统调用

在 C 程序中使用系统调用发送信号：

```
#include <sys/types.h>
#include <signal.h>

// 向指定进程发送信号
int kill(pid_t pid, int sig);

// 向自身发送信号
int raise(int sig);

// 向进程组发送信号
int killpg(int pgrp, int sig);

// 发送实时信号（可附带数据）
int sigqueue(pid_t pid, int sig, const union sigval value);
```

示例：在程序中使用 kill 发送信号

```
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <unistd.h>

int main() {
    pid_t target_pid = 1234; // 目标进程ID
    
    // 向目标进程发送 SIGTERM 信号
    if (kill(target_pid, SIGTERM) == -1) {
        perror("发送信号失败");
        return 1;
    }
    
    return 0;
}
```

## 2. 处理信号

### signal() 函数

简单的信号处理函数注册方法：

```
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <unistd.h>

void signal_handler(int signum) {
    printf("捕获到信号: %d\n", signum);
}

int main() {
    // 注册信号处理函数
    if (signal(SIGINT, signal_handler) == SIG_ERR) {
        perror("signal() 失败");
        return 1;
    }
    
    // 保持程序运行以接收信号
    while(1) {
        printf("程序运行中... PID: %d\n", getpid());
        sleep(1);
    }
    
    return 0;
}
```

### sigaction() 函数（推荐使用）

更强大且可移植的信号处理方法：

```
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <unistd.h>

void signal_handler(int signum) {
    printf("捕获到信号: %d\n", signum);
}

int main() {
    struct sigaction new_action, old_action;
    
    // 设置信号处理函数
    new_action.sa_handler = signal_handler;
    sigemptyset(&new_action.sa_mask);
    new_action.sa_flags = 0;
    
    // 注册 SIGINT 信号处理
    if (sigaction(SIGINT, &new_action, &old_action) == -1) {
        perror("sigaction() 失败");
        return 1;
    }
    
    // 保持程序运行
    while(1) {
        sleep(1);
    }
    
    return 0;
}
```

### sigaction 的优势

- 可以指定在执行处理函数期间阻塞哪些其他信号
- 提供更多控制选项（`sa_flags`）
- 对于实时信号，可以获取发送者的信息
- 更好的可移植性和可靠性

# ⚠️ 重要注意事项

## 1. 特殊信号的处理限制

- **SIGKILL 和 SIGSTOP 不能被捕获、阻塞或忽略**
- 这是 Linux 系统的安全机制，确保管理员总是能控制进程
- 尝试为这些信号设置处理函数会失败

## 2. 信号处理函数的安全性

信号处理函数应保持简单，因为它在异步环境中执行。避免在信号处理函数中使用：

- 非可重入函数（如 `malloc`, `printf`等）
- 复杂的系统调用
- 静态数据结构操作

安全的做法：

```
void safe_handler(int signum) {
    // 只设置一个 volatile sig_atomic_t 类型的标志
    signal_received = signum;
    // 或者使用简单的 write() 系统调用
    write(STDOUT_FILENO, "信号收到!\n", 12);
}
```

## 3. 信号丢失与可靠性

- **标准信号（1-31）不支持排队**：短时间内收到多个相同信号，可能只处理一次
- **实时信号（34-64）支持排队**：不会丢失信号
- 如果信号处理很重要，考虑使用实时信号

## 4. 阻塞信号

可以使用信号掩码临时阻塞特定信号：

```
sigset_t new_mask, old_mask;

// 初始化信号集
sigemptyset(&new_mask);
sigaddset(&new_mask, SIGINT);

// 阻塞 SIGINT 信号
sigprocmask(SIG_BLOCK, &new_mask, &old_mask);

// 临界区代码（不会被 SIGINT 中断）

// 恢复原来的信号掩码
sigprocmask(SIG_SETMASK, &old_mask, NULL);
```

## 5. 多线程中的信号处理

- 在多线程程序中，信号处理更加复杂
- 信号可以发送到特定线程（使用 `pthread_kill()`）
- 建议在主线程中设置信号处理，并阻塞工作线程的信号
- 或者使用专门的信号处理线程

## 6. 系统调用中断与重启

信号可能中断慢速系统调用（如 read、write）：

```
// 设置 SA_RESTART 标志可自动重启被中断的系统调用
new_action.sa_flags = SA_RESTART;
```

如果不使用 SA_RESTART，系统调用可能返回 EINTR 错误，需要在代码中手动处理。

# 💎 总结

Linux 信号机制是进程间通信的重要方式，适用于事件通知、异常处理和进程控制等场景。使用时需要注意信号的特性和限制，特别是信号处理函数的安全性和特殊信号（SIGKILL、SIGSTOP）的不可捕获性。对于复杂的应用程序，建议使用 `sigaction`而非 `signal`函数，并考虑实时信号的需求和信号阻塞机制。