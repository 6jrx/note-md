在Linux中，`wait()`函数是一个重要的系统调用，用于**父进程等待子进程的状态变化**并回收子进程资源。

## 1. 函数原型与基本概念

### 头文件

```
#include <sys/types.h>
#include <sys/wait.h>
```

### 函数原型

```
pid_t wait(int *status);
```

**功能**：`wait()`函数会使父进程**阻塞**，直到任意一个子进程终止。一旦有子进程终止，`wait()`会回收该子进程的资源，防止其变成僵尸进程。

**参数**：

- `status`：整型指针，用于存储子进程的退出状态。如果不需要状态信息，可设为`NULL`。

**返回值**：

- 成功：返回被终止子进程的PID
- 失败：返回-1

## 2. 使用wait()的基本示例

### 示例1：简单使用（不关心退出状态）

```
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>
#include <stdlib.h>

int main() {
    pid_t pid;
    
    pid = fork();
    
    if (pid == 0) {
        // 子进程
        printf("子进程运行中，PID: %d\n", getpid());
        sleep(3);  // 模拟子进程工作
        printf("子进程结束\n");
        exit(0);
    } else if (pid > 0) {
        // 父进程
        printf("父进程等待子进程...\n");
        pid_t child_pid = wait(NULL);  // 不关心退出状态
        printf("回收了子进程 PID: %d\n", child_pid);
    } else {
        perror("fork失败");
        exit(1);
    }
    
    return 0;
}
```

**运行结果**：

```
父进程等待子进程...
子进程运行中，PID: 1508
子进程结束
回收了子进程 PID: 1508
```

### 示例2：获取子进程退出状态

```
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>
#include <stdlib.h>

int main() {
    pid_t pid;
    int status;
    
    pid = fork();
    
    if (pid == 0) {
        // 子进程
        printf("子进程运行中\n");
        sleep(2);
        exit(42);  // 子进程退出码为42
    } else if (pid > 0) {
        // 父进程
        pid_t child_pid = wait(&status);  // 获取退出状态
        
        if (WIFEXITED(status)) {
            printf("子进程 %d 正常退出，退出码: %d\n", 
                   child_pid, WEXITSTATUS(status));
        }
    }
    
    return 0;
}
```

## 3. 解析子进程退出状态

当需要获取子进程的退出详情时，可以使用以下宏来解析`status`参数：

| 宏                    | 功能描述                               |
| --------------------- | -------------------------------------- |
| `WIFEXITED(status)`   | 子进程是否正常退出（通过exit或return） |
| `WEXITSTATUS(status)` | 如果正常退出，提取退出码               |
| `WIFSIGNALED(status)` | 子进程是否因信号终止                   |
| `WTERMSIG(status)`    | 如果因信号终止，提取信号编号           |
| `WIFSTOPPED(status)`  | 子进程是否处于暂停状态                 |
| `WSTOPSIG(status)`    | 提取导致暂停的信号编号                 |

### 示例3：详细的状态检查

```
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>
#include <stdlib.h>

int main() {
    pid_t pid;
    int status;
    
    pid = fork();
    
    if (pid == 0) {
        // 子进程
        printf("子进程 PID: %d 开始运行\n", getpid());
        sleep(1);
        // 模拟异常退出：除以零
        int a = 1, b = 0;
        int c = a / b;  // 这将导致SIGFPE信号
        exit(0);
    } else if (pid > 0) {
        // 父进程
        wait(&status);
        
        if (WIFEXITED(status)) {
            printf("子进程正常退出，退出码: %d\n", WEXITSTATUS(status));
        } else if (WIFSIGNALED(status)) {
            printf("子进程因信号终止，信号编号: %d\n", WTERMSIG(status));
        }
    }
    
    return 0;
}
```

## 4. 等待多个子进程

当有多个子进程时，`wait()`会按子进程终止的顺序回收它们。

### 示例4：等待多个子进程

```
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>
#include <stdlib.h>

int main(int argc, char *argv[]) {
    pid_t pid;
    int i, num_children = 3;
    
    // 创建多个子进程
    for (i = 0; i < num_children; i++) {
        pid = fork();
        
        if (pid == 0) {
            // 子进程
            printf("子进程 %d (PID: %d) 运行中\n", i+1, getpid());
            sleep(i+1);  // 每个子进程睡眠时间不同
            printf("子进程 %d 结束\n", i+1);
            exit(i+1);   // 退出码为i+1
        } else if (pid < 0) {
            perror("fork失败");
            exit(1);
        }
    }
    
    // 父进程等待所有子进程
    int finished = 0;
    while (finished < num_children) {
        int status;
        pid_t child_pid = wait(&status);
        
        if (child_pid > 0) {
            finished++;
            if (WIFEXITED(status)) {
                printf("回收子进程 PID: %d, 退出码: %d\n", 
                       child_pid, WEXITSTATUS(status));
            }
        } else {
            break;  // 没有更多子进程
        }
    }
    
    printf("所有子进程已回收\n");
    return 0;
}
```

## 5. wait()的局限性及waitpid()的替代方案

`wait()`函数有几个局限性：

1. 只能等待任意子进程，不能指定特定子进程
2. 始终阻塞调用进程
3. 无法检测子进程的暂停和恢复

为了解决这些问题，可以使用`waitpid()`函数：

### 示例5：使用waitpid()实现非阻塞等待

```
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>
#include <stdlib.h>

int main() {
    pid_t pid = fork();
    
    if (pid == 0) {
        // 子进程
        sleep(5);
        exit(10);
    } else if (pid > 0) {
        // 父进程：非阻塞等待
        int status;
        pid_t result;
        
        do {
            result = waitpid(pid, &status, WNOHANG);  // 非阻塞
            
            if (result == 0) {
                printf("子进程尚未结束，父进程可做其他工作...\n");
                sleep(1);
            }
        } while (result == 0);
        
        if (WIFEXITED(status)) {
            printf("子进程退出码: %d\n", WEXITSTATUS(status));
        }
    }
    
    return 0;
}
```

## 6. 重要注意事项

1. **僵尸进程处理**：如果不调用`wait()`，已终止的子进程会变成僵尸进程，占用系统资源。
2. **阻塞特性**：`wait()`会使调用进程阻塞，直到有子进程终止。如果需要非阻塞操作，应使用`waitpid()`with `WNOHANG`选项。
3. **错误处理**：始终检查`wait()`的返回值，如果返回-1且`errno == ECHILD`，表示没有子进程可等待。
4. **信号中断**：`wait()`可能被信号中断，需要适当处理：

```
pid_t r_wait(int *stat_loc) {
    int retval;
    // 如果被信号中断，重启wait()
    while (((retval = wait(stat_loc)) == -1) && (errno == EINTR));
    return retval;
}
```

`wait()`函数是Linux进程管理的基础工具，合理使用可以确保进程间正常同步并避免资源泄漏。