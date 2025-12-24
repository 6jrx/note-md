# Linux中的互斥锁(Mutex)

## 什么是互斥锁？

互斥锁（Mutex）是一种用于多线程编程中的同步机制，用于保护共享资源，防止多个线程同时访问同一资源而导致的数据竞争和不一致性问题。

## 互斥锁的基本特性

- **互斥性**：同一时刻只有一个线程可以持有互斥锁
- **原子性**：加锁和解锁操作是原子的
- **阻塞机制**：当锁被占用时，其他尝试获取锁的线程会被阻塞
- **所有权概念**：通常由加锁的线程负责解锁

## 常用函数

### 1. 初始化和销毁

```c
#include <pthread.h>

// 动态初始化
int pthread_mutex_init(pthread_mutex_t *mutex, 
                      const pthread_mutexattr_t *attr);

// 静态初始化
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

// 销毁
int pthread_mutex_destroy(pthread_mutex_t *mutex);
```

### 2. 加锁操作

```c
// 阻塞加锁
int pthread_mutex_lock(pthread_mutex_t *mutex);

// 非阻塞加锁
int pthread_mutex_trylock(pthread_mutex_t *mutex);

// 带超时的加锁
int pthread_mutex_timedlock(pthread_mutex_t *mutex,
                           const struct timespec *abs_timeout);
```

### 3. 解锁操作

```c
int pthread_mutex_unlock(pthread_mutex_t *mutex);
```

## 使用方法

### 基本使用示例

```c
#include <stdio.h>
#include <pthread.h>

pthread_mutex_t mutex;
int shared_data = 0;

void* thread_function(void* arg) {
    for (int i = 0; i < 100000; i++) {
        // 加锁
        pthread_mutex_lock(&mutex);
        
        // 临界区 - 操作共享数据
        shared_data++;
        
        // 解锁
        pthread_mutex_unlock(&mutex);
    }
    return NULL;
}

int main() {
    pthread_t threads[2];
    
    // 初始化互斥锁
    pthread_mutex_init(&mutex, NULL);
    
    // 创建线程
    pthread_create(&threads[0], NULL, thread_function, NULL);
    pthread_create(&threads[1], NULL, thread_function, NULL);
    
    // 等待线程结束
    pthread_join(threads[0], NULL);
    pthread_join(threads[1], NULL);
    
    printf("Final shared_data: %d\n", shared_data);
    
    // 销毁互斥锁
    pthread_mutex_destroy(&mutex);
    
    return 0;
}
```

### 使用静态初始化

```c
#include <pthread.h>

// 静态初始化互斥锁
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

void* worker_thread(void* arg) {
    pthread_mutex_lock(&mutex);
    
    // 临界区代码
    printf("Thread %ld in critical section\n", (long)arg);
    
    pthread_mutex_unlock(&mutex);
    return NULL;
}
```

### 错误处理示例

```c
int result = pthread_mutex_lock(&mutex);
if (result != 0) {
    // 处理错误
    perror("pthread_mutex_lock failed");
    return -1;
}

// 临界区操作

result = pthread_mutex_unlock(&mutex);
if (result != 0) {
    perror("pthread_mutex_unlock failed");
}
```

## 互斥锁属性

### 设置互斥锁属性

```c
pthread_mutexattr_t attr;
pthread_mutex_t mutex;

// 初始化属性
pthread_mutexattr_init(&attr);

// 设置互斥锁类型
pthread_mutexattr_settype(&attr, PTHREAD_MUTEX_RECURSIVE);

// 使用属性初始化互斥锁
pthread_mutex_init(&mutex, &attr);

// 销毁属性
pthread_mutexattr_destroy(&attr);
```

### 常用的互斥锁类型

- `PTHREAD_MUTEX_NORMAL` - 标准互斥锁
- `PTHREAD_MUTEX_RECURSIVE` - 递归互斥锁，同一线程可多次加锁
- `PTHREAD_MUTEX_ERRORCHECK` - 错误检查互斥锁

## 实际应用示例

### 线程安全的计数器

```c
#include <pthread.h>

typedef struct {
    int count;
    pthread_mutex_t mutex;
} thread_safe_counter;

void counter_init(thread_safe_counter* counter) {
    counter->count = 0;
    pthread_mutex_init(&counter->mutex, NULL);
}

void counter_increment(thread_safe_counter* counter) {
    pthread_mutex_lock(&counter->mutex);
    counter->count++;
    pthread_mutex_unlock(&counter->mutex);
}

int counter_get(thread_safe_counter* counter) {
    int value;
    pthread_mutex_lock(&counter->mutex);
    value = counter->count;
    pthread_mutex_unlock(&counter->mutex);
    return value;
}

void counter_destroy(thread_safe_counter* counter) {
    pthread_mutex_destroy(&counter->mutex);
}
```

## 注意事项

1. **避免死锁**：确保加锁顺序一致，及时释放锁
2. **锁粒度**：锁的粒度要适当，不要过大或过小
3. **性能考虑**：尽量减少临界区的代码量
4. **异常安全**：确保在异常情况下也能释放锁
5. **递归锁使用**：谨慎使用递归锁，避免逻辑复杂化

## 返回值说明

所有pthread互斥锁函数成功时返回0，失败时返回错误码。常见的错误码包括：
- `EAGAIN`：资源暂时不可用
- `EDEADLK`：检测到死锁
- `EINVAL`：参数无效
- `EPERM`：当前线程不持有该锁

互斥锁是多线程编程中最重要的同步机制之一，正确使用可以确保程序的正确性和稳定性。