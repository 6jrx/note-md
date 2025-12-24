# 一、基本认识

在 Linux 系统中，线程是轻量级的执行单元，允许在同一个进程内实现并发任务，能有效提升程序性能和资源利用率。要掌握多线程编程，关键在于理解其使用方法和相关注意事项。下面我将用一个对比表格来清晰展示线程共享与非共享的关键资源，然后详细介绍线程的使用步骤和核心注意事项。

### 线程共享与非共享资源

多线程编程中，首先要明确哪些资源是线程间共享的，哪些是线程私有的，这是避免数据混乱的基础。

| 共享资源                                         | 非共享资源                               |
| ------------------------------------------------ | ---------------------------------------- |
| 全局变量和内存地址空间（.text, .data, .bss, 堆） | 线程ID（Thread ID）                      |
| 文件描述符表                                     | 寄存器值、栈指针（内核栈）               |
| 信号的处理方式                                   | 独立的栈空间（用户栈），用于存放局部变量 |
| 当前工作目录、用户ID和组ID                       | 错误号 `errno`                           |
| -                                                | 信号屏蔽字和调度优先级                   |

### 线程的使用步骤 ✨

在 Linux 中，线程操作主要通过 POSIX 线程（pthread）库实现。

1. **创建线程**

   使用 `pthread_create`函数来创建新线程。其原型如下：

   ```
   int pthread_create(pthread_t *thread, const pthread_attr_t *attr, void *(*start_routine) (void *), void *arg);
   ```
   
   - **thread**：传出参数，用于保存新线程的 ID。
   - **attr**：设置线程属性，通常传 `NULL`使用默认属性。
   - **start_routine**：线程主函数的函数指针，新线程会从这个函数开始执行。
   - **arg**：传递给线程主函数的参数。 **编译时需链接 pthread 库**：`gcc -o program program.c -lpthread`。
   
2. **终止线程** 安全地终止线程有几种方式： 线程函数自然执行到末尾并返回。 在线程内调用 `pthread_exit(void *retval)`。 被同一进程内的其他线程通过 `pthread_cancel(pthread_t thread)`取消。 **重要提示**：在线程函数中**严禁使用** `exit()`函数，因为 `exit()`会导致整个进程终止，从而影响所有其他线程。

3. **回收线程资源** 线程终止后，需要回收其资源，避免产生"僵尸线程"。主要有两种方法： **`pthread_join(pthread_t thread, void **retval)`**：这是一个阻塞调用，会等待指定线程结束，并获取其退出状态。适用于需要知道线程执行结果的场景。 **`pthread_detach(pthread_t thread)`**：将线程设置为分离（detached）状态。处于分离状态的线程结束后，系统会自动回收其资源，无需其他线程调用 `join`。可以通过 `pthread_detach(pthread_self())`在线程内自行分离。

### 核心注意事项与同步机制 ⚠️

多线程编程最大的挑战来自于对共享资源的协调访问。

1. **使用互斥锁（Mutex）保护共享数据**

   当多个线程需要读写同一个全局变量或数据结构时，必须使用互斥锁来保证任一时刻只有一个线程处于临界区。

   ```
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER; // 静态初始化互斥锁
   
   // 在线程函数中
   pthread_mutex_lock(&lock);   // 加锁
   // ... 操作共享资源
   pthread_mutex_unlock(&lock); // 解锁
   ```
   
   使用互斥锁时要小心**死锁**（Deadlock）情况，例如一个线程试图对同一把锁连续加锁两次，或多个线程互相持有并等待对方释放锁资源。

2. **使用条件变量（Condition Variable）进行线程协作**

   当线程需要等待某个条件满足时才继续执行时（例如，生产者-消费者模型），条件变量是比循环检查更高效的机制。

   ```
   pthread_cond_t cond = PTHREAD_COND_INITIALIZER; // 初始化条件变量
   // 线程A：等待条件
   pthread_mutex_lock(&mutex);
   while (condition_is_false) {
       pthread_cond_wait(&cond, &mutex); // 等待条件变量，会暂时释放互斥锁
   }
   // ... 条件满足后执行操作
   pthread_mutex_unlock(&mutex);
   
   // 线程B：通知条件满足
   pthread_mutex_lock(&mutex);
   // ... 改变条件
   pthread_cond_signal(&cond); // 唤醒一个等待线程（或broadcast唤醒所有）
   pthread_mutex_unlock(&mutex);
   ```
   
3. **谨慎处理线程参数和返回值** 传递给线程的参数和线程的返回值，其指向的内存空间必须是全局的或由 `malloc`分配的，**不能**是线程函数栈上的局部变量。因为当线程函数退出后，其栈空间可能被覆盖，导致数据无效。 在创建线程时，如果主线程会立即循环接受新连接（例如Socket服务端），务必通过参数将必要的值（如客户端socket文件描述符）传递给新线程，而不是让新线程直接去读可能已被主线程修改的全局变量。

### 实践建议

- **避免滥用全局变量**：尽管线程共享全局变量很方便，但应尽量限制全局变量的使用，通过参数传递数据，以降低程序的耦合度和复杂度。
- **理解线程与信号的关系**：多线程环境下对信号的处理比较复杂，建议尽量避免在线程中使用信号。如果必须使用，需要仔细研究信号屏蔽字等相关函数。
- **利用工具排查问题**：可以使用 `top -H`或 `ps -xH`命令来查看进程中的线程信息。对于死锁或竞态条件问题，可以使用 `gdb`调试器或 `valgrind`等工具辅助分析。

希望这份详细的说明能帮助你更好地理解和应用 Linux 多线程编程。如果你在具体的实现场景中遇到更细致的问题，欢迎继续探讨。



---



# 二、基础使用



## 1. 线程基础使用

### 创建线程
```c
#include <pthread.h>
#include <stdio.h>
#include <unistd.h>

void* thread_function(void* arg) {
    int thread_id = *(int*)arg;
    printf("线程 %d 正在执行\n", thread_id);
    return NULL;
}

int main() {
    pthread_t thread1, thread2;
    int id1 = 1, id2 = 2;
    
    // 创建线程
    pthread_create(&thread1, NULL, thread_function, &id1);
    pthread_create(&thread2, NULL, thread_function, &id2);
    
    // 等待线程结束
    pthread_join(thread1, NULL);
    pthread_join(thread2, NULL);
    
    printf("所有线程执行完毕\n");
    return 0;
}
```

### 编译时需要链接pthread库
```bash
gcc -o program program.c -lpthread
```

## 2. 线程同步机制

### 互斥锁（Mutex）
```c
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
int shared_data = 0;

void* thread_func(void* arg) {
    pthread_mutex_lock(&mutex);
    shared_data++;  // 临界区
    pthread_mutex_unlock(&mutex);
    return NULL;
}
```

### 条件变量（Condition Variable）
```c
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;
int ready = 0;

void* producer(void* arg) {
    pthread_mutex_lock(&mutex);
    ready = 1;
    pthread_cond_signal(&cond);  // 通知消费者
    pthread_mutex_unlock(&mutex);
    return NULL;
}

void* consumer(void* arg) {
    pthread_mutex_lock(&mutex);
    while (!ready) {
        pthread_cond_wait(&cond, &mutex);  // 等待条件
    }
    // 处理数据
    pthread_mutex_unlock(&mutex);
    return NULL;
}
```

### 读写锁（Read-Write Lock）
```c
pthread_rwlock_t rwlock = PTHREAD_RWLOCK_INITIALIZER;

void* reader(void* arg) {
    pthread_rwlock_rdlock(&rwlock);  // 读锁
    // 读取数据
    pthread_rwlock_unlock(&rwlock);
    return NULL;
}

void* writer(void* arg) {
    pthread_rwlock_wrlock(&rwlock);  // 写锁
    // 写入数据
    pthread_rwlock_unlock(&rwlock);
    return NULL;
}
```

## 3. 重要注意事项

### 3.1 线程安全
```c
// 不安全的例子 - 使用非线程安全函数
void unsafe_function() {
    printf("结果: %s\n", strerror(errno));  // strerror不是线程安全的
}

// 安全的替代方案
void safe_function() {
    char buffer[256];
    strerror_r(errno, buffer, sizeof(buffer));  // 线程安全版本
    printf("结果: %s\n", buffer);
}
```

### 3.2 资源管理
```c
void* worker_thread(void* arg) {
    // 动态分配的资源要在线程内释放
    char* buffer = malloc(1024);
    if (!buffer) {
        // 处理分配失败
        return NULL;
    }
    
    // 使用buffer...
    
    free(buffer);  // 必须释放！
    return NULL;
}
```

### 3.3 避免竞争条件
```c
// 有竞争条件的代码
int counter = 0;

void* increment(void* arg) {
    for (int i = 0; i < 1000000; i++) {
        counter++;  // 这不是原子操作！
    }
    return NULL;
}

// 修正版本 - 使用原子操作或互斥锁
void* safe_increment(void* arg) {
    for (int i = 0; i < 1000000; i++) {
        pthread_mutex_lock(&mutex);
        counter++;
        pthread_mutex_unlock(&mutex);
    }
    return NULL;
}
```

## 4. 高级特性

### 线程属性设置
```c
pthread_attr_t attr;
pthread_t thread;

// 初始化属性
pthread_attr_init(&attr);

// 设置线程为分离状态（不需要pthread_join）
pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED);

// 设置栈大小
size_t stack_size = 1024 * 1024;  // 1MB
pthread_attr_setstacksize(&attr, stack_size);

// 创建线程
pthread_create(&thread, &attr, thread_function, NULL);

// 销毁属性
pthread_attr_destroy(&attr);
```

### 线程局部存储
```c
// 定义线程局部变量
__thread int thread_local_var = 0;

void* thread_func(void* arg) {
    thread_local_var = (int)(long)arg;  // 每个线程有自己的副本
    printf("线程局部变量: %d\n", thread_local_var);
    return NULL;
}
```

## 5. 常见陷阱和最佳实践

### 5.1 避免的陷阱
```c
// 错误：传递栈上变量的地址
void wrong_way() {
    int local_var = 42;
    pthread_t thread;
    pthread_create(&thread, NULL, thread_function, &local_var);  // 危险！
    // local_var可能在被线程使用前就失效了
}

// 正确：使用堆分配或全局变量
void correct_way() {
    int* heap_var = malloc(sizeof(int));
    *heap_var = 42;
    pthread_t thread;
    pthread_create(&thread, NULL, thread_function, heap_var);
    // 在线程函数中记得释放
}
```

### 5.2 优雅终止
```c
volatile int should_exit = 0;

void* worker(void* arg) {
    while (!should_exit) {
        // 执行工作
        // 定期检查退出标志
    }
    return NULL;
}

void stop_worker() {
    should_exit = 1;
}
```

## 6. 调试技巧

### 使用gdb调试多线程
```bash
# 编译时添加调试信息
gcc -g -o program program.c -lpthread

# 在gdb中调试
gdb ./program
(gdb) break main
(gdb) run
(gdb) info threads      # 查看所有线程
(gdb) thread 2          # 切换到线程2
(gdb) bt                # 查看线程栈
```

### 线程检查工具
- **Helgrind**: 检测数据竞争
- **DRD**: 检测线程错误
- **TSan**: ThreadSanitizer

## 总结

使用Linux线程时需要注意：
1. **同步机制**：合理使用互斥锁、条件变量等
2. **资源管理**：确保资源正确分配和释放
3. **线程安全**：避免使用非线程安全函数
4. **错误处理**：检查所有pthread函数的返回值
5. **性能考虑**：避免过度同步导致的性能下降

正确使用线程可以充分利用多核CPU的优势，但需要仔细处理同步和资源共享问题。



---



# 三、常用函数



Linux 中的 pthread 库提供了丰富的线程操作函数。以下是一些常用方法及示例：

## 1. 线程创建和管理

### pthread_create - 创建线程
```c
#include <pthread.h>
#include <stdio.h>

void* thread_function(void* arg) {
    printf("线程执行中，参数: %d\n", *(int*)arg);
    return NULL;
}

int main() {
    pthread_t thread_id;
    int arg = 42;
    
    int result = pthread_create(&thread_id, NULL, thread_function, &arg);
    if (result != 0) {
        perror("线程创建失败");
        return 1;
    }
    
    pthread_join(thread_id, NULL);
    return 0;
}
```

### pthread_join - 等待线程结束
```c
void* calculate_sum(void* arg) {
    int* numbers = (int*)arg;
    int sum = numbers[0] + numbers[1];
    
    int* result = malloc(sizeof(int));
    *result = sum;
    return result;
}

int main() {
    pthread_t thread;
    int numbers[2] = {10, 20};
    int* sum_result;
    
    pthread_create(&thread, NULL, calculate_sum, numbers);
    pthread_join(thread, (void**)&sum_result);
    
    printf("计算结果: %d\n", *sum_result);
    free(sum_result);
    return 0;
}
```

### pthread_exit - 线程退出
```c
void* worker_thread(void* arg) {
    for (int i = 0; i < 5; i++) {
        printf("工作线程运行中... %d\n", i);
        sleep(1);
    }
    pthread_exit(NULL); // 显式退出线程
}
```

## 2. 线程同步

### pthread_mutex - 互斥锁
```c
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
int shared_counter = 0;

void* increment_counter(void* arg) {
    for (int i = 0; i < 1000; i++) {
        pthread_mutex_lock(&mutex);
        shared_counter++;
        pthread_mutex_unlock(&mutex);
    }
    return NULL;
}

int main() {
    pthread_t threads[5];
    
    for (int i = 0; i < 5; i++) {
        pthread_create(&threads[i], NULL, increment_counter, NULL);
    }
    
    for (int i = 0; i < 5; i++) {
        pthread_join(threads[i], NULL);
    }
    
    printf("最终计数器值: %d\n", shared_counter);
    pthread_mutex_destroy(&mutex);
    return 0;
}
```

### pthread_cond - 条件变量
```c
pthread_mutex_t cond_mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t condition = PTHREAD_COND_INITIALIZER;
int ready = 0;

void* producer(void* arg) {
    sleep(2); // 模拟生产时间
    pthread_mutex_lock(&cond_mutex);
    ready = 1;
    printf("生产者: 数据准备好了\n");
    pthread_cond_signal(&condition); // 通知消费者
    pthread_mutex_unlock(&cond_mutex);
    return NULL;
}

void* consumer(void* arg) {
    pthread_mutex_lock(&cond_mutex);
    while (!ready) {
        printf("消费者: 等待数据...\n");
        pthread_cond_wait(&condition, &cond_mutex); // 等待条件
    }
    printf("消费者: 收到数据\n");
    pthread_mutex_unlock(&cond_mutex);
    return NULL;
}
```

## 3. 线程属性

### 设置线程属性
```c
void* detached_thread(void* arg) {
    printf("这是分离线程，自动清理资源\n");
    return NULL;
}

int main() {
    pthread_t thread;
    pthread_attr_t attr;
    
    pthread_attr_init(&attr);
    pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED);
    
    pthread_create(&thread, &attr, detached_thread, NULL);
    
    pthread_attr_destroy(&attr);
    sleep(1); // 给线程执行时间
    return 0;
}
```

## 4. 线程特定数据

### pthread_key - 线程私有数据
```c
pthread_key_t thread_key;

void destructor(void* data) {
    printf("清理线程特定数据: %s\n", (char*)data);
    free(data);
}

void* thread_func(void* arg) {
    char* data = malloc(100);
    sprintf(data, "线程 %ld 的私有数据", (long)arg);
    
    pthread_setspecific(thread_key, data);
    
    char* retrieved = pthread_getspecific(thread_key);
    printf("获取到: %s\n", retrieved);
    
    return NULL;
}

int main() {
    pthread_key_create(&thread_key, destructor);
    
    pthread_t threads[3];
    for (long i = 0; i < 3; i++) {
        pthread_create(&threads[i], NULL, thread_func, (void*)i);
    }
    
    for (int i = 0; i < 3; i++) {
        pthread_join(threads[i], NULL);
    }
    
    pthread_key_delete(thread_key);
    return 0;
}
```

## 5. 其他常用函数

### pthread_self - 获取线程ID
```c
void* show_thread_id(void* arg) {
    pthread_t self_id = pthread_self();
    printf("线程ID: %lu\n", (unsigned long)self_id);
    return NULL;
}
```

### pthread_equal - 比较线程ID
```c
void* compare_threads(void* arg) {
    pthread_t current = pthread_self();
    pthread_t main_thread = *(pthread_t*)arg;
    
    if (pthread_equal(current, main_thread)) {
        printf("这是主线程\n");
    } else {
        printf("这是工作线程\n");
    }
    return NULL;
}
```

## 6. 读写锁示例

```c
pthread_rwlock_t rwlock = PTHREAD_RWLOCK_INITIALIZER;
int shared_data = 0;

void* reader(void* arg) {
    for (int i = 0; i < 3; i++) {
        pthread_rwlock_rdlock(&rwlock);
        printf("读者 %ld: 读取数据 %d\n", (long)arg, shared_data);
        pthread_rwlock_unlock(&rwlock);
        sleep(1);
    }
    return NULL;
}

void* writer(void* arg) {
    for (int i = 0; i < 2; i++) {
        pthread_rwlock_wrlock(&rwlock);
        shared_data++;
        printf("写者: 写入数据 %d\n", shared_data);
        pthread_rwlock_unlock(&rwlock);
        sleep(2);
    }
    return NULL;
}
```

## 编译说明

编译时需要链接 pthread 库：
```bash
gcc -o program program.c -lpthread
```

这些是 pthread 库中最常用的方法，涵盖了线程的创建、同步、通信等基本操作。实际使用时需要根据具体场景选择合适的同步机制。