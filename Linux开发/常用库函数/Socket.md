# Linux C Socket编程指南

## 常用函数手册

这个表格汇总了Linux C Socket编程的核心函数，帮助你快速了解它们的用途和基本用法。

| 函数名称   | 功能描述                                                                 | 定义                                                         | 关键参数说明                                                                                                                              | 返回值                                       |
| :--------- | :----------------------------------------------------------------------- | :----------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------- |
| **socket** | 创建通信端点（套接字）                                                 | `int socket(int domain, int type, int protocol);`            | `domain`: 协议族（如 `AF_INET` IPv4）；`type`: 套接字类型（如 `SOCK_STREAM` TCP）；`protocol`: 通常设为0 | 成功返回套接字描述符（≥0），失败返回-1 |
| **bind**   | 将套接字与特定IP地址和端口号绑定                                       | `int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);` | `sockfd`: 套接字描述符；`addr`: 指向包含IP和端口的结构体指针（常用`sockaddr_in`）；`addrlen`: 地址结构长度                                                  | 成功返回0，失败返回-1                  |
| **listen** | 使套接字进入被动监听状态，等待客户端连接                               | `int listen(int sockfd, int backlog);`                       | `sockfd`: 绑定的套接字描述符；`backlog`: 连接请求队列的最大长度                                                                      | 成功返回0，失败返回-1                  |
| **accept** | 接受客户端的连接请求，并返回一个新的套接字用于与客户端通信                 | `int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);` | `sockfd`: 处于监听状态的套接字；`addr`: （可选）用于存储客户端地址信息；`addrlen`: （可选）客户端地址结构长度的指针                          | 成功返回新套接字描述符（≥0），失败返回-1 |
| **connect** | 客户端用于连接服务器                                          | `int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);` | `sockfd`: 套接字描述符；`addr`: 指向服务器地址结构的指针；`addrlen`: 服务器地址结构长度                                                          | 成功返回0，失败返回-1                  |
| **send**   | 通过已连接的TCP套接字发送数据                                          | `ssize_t send(int sockfd, const void *buf, size_t len, int flags);` | `sockfd`: 已连接的套接字描述符；`buf`: 发送数据缓冲区；`len`: 发送数据长度；`flags`: 标志位（通常为0）                                                      | 成功返回实际发送的字节数，失败返回-1               |
| **recv**   | 从已连接的TCP套接字接收数据                                          | `ssize_t recv(int sockfd, void *buf, size_t len, int flags);` | `sockfd`: 已连接的套接字描述符；`buf`: 接收数据缓冲区；`len`: 缓冲区长度；`flags`: 标志位（通常为0）                                                      | 成功返回实际接收的字节数，失败返回-1               |
| **close**  | 关闭套接字，终止通信                                                | `int close(int sockfd);`                                      | `sockfd`: 需要关闭的套接字描述符                                                                                               | 成功返回0，失败返回-1                 |

📚 **其他重要函数**
*   `sendto()` / `recvfrom()`: 主要用于UDP协议的无连接数据收发。
*   `select()` / `poll()`: 用于I/O多路复用，实现同时监控多个套接字。
*   `setsockopt()`: 设置套接字选项（如地址复用）。
*   `htonl()`, `htons()`, `ntohl()`, `ntohs()`: 字节序转换函数（主机字节序与网络字节序互换）。
*   `inet_addr()`, `inet_ntoa()`: 点分十进制IP地址与网络字节序二进制地址的相互转换。

## 案例详解：TCP服务器与客户端

### TCP服务器代码 (server.c)

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>
#include <netinet/in.h>

#define SERVER_PORT 8000    // 服务器监听的端口号
#define BUFFER_SIZE 1024    // 缓冲区大小
#define BACKLOG 5           // 监听队列的最大长度

int main() {
    int server_sock, client_sock;                // 定义服务器和客户端套接字描述符
    struct sockaddr_in server_addr, client_addr; // 定义服务器和客户端的地址结构体
    socklen_t client_addr_len = sizeof(client_addr); // 客户端地址结构体长度
    char buffer[BUFFER_SIZE];                    // 数据缓冲区
    char *message = "Hello from server!";        // 要发送给客户端的消息

    // 1. 创建TCP套接字 (IPv4, 流式套接字, TCP协议)
    if ((server_sock = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
        perror("socket creation failed");
        exit(EXIT_FAILURE);
    }

    // 设置套接字选项，避免地址复用错误（SO_REUSEADDR）
    int opt = 1;
    if (setsockopt(server_sock, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt))) {
        perror("setsockopt failed");
        close(server_sock);
        exit(EXIT_FAILURE);
    }

    // 2. 初始化服务器地址结构
    memset(&server_addr, 0, sizeof(server_addr));    // 清空结构体
    server_addr.sin_family = AF_INET;                // 使用IPv4地址族
    server_addr.sin_addr.s_addr = htonl(INADDR_ANY); // 监听所有本地IP地址
    server_addr.sin_port = htons(SERVER_PORT);       // 设置端口号（转换为网络字节序）

    // 3. 将套接字绑定到指定的IP地址和端口
    if (bind(server_sock, (struct sockaddr*)&server_addr, sizeof(server_addr)) < 0) {
        perror("bind failed");
        close(server_sock);
        exit(EXIT_FAILURE);
    }

    // 4. 开始监听，等待客户端连接
    if (listen(server_sock, BACKLOG) < 0) {
        perror("listen failed");
        close(server_sock);
        exit(EXIT_FAILURE);
    }

    printf("Server is listening on port %d...\n", SERVER_PORT);

    // 5. 接受客户端连接
    if ((client_sock = accept(server_sock, (struct sockaddr*)&client_addr, &client_addr_len)) < 0) {
        perror("accept failed");
        close(server_sock);
        exit(EXIT_FAILURE);
    }

    // 打印连接的客户端信息
    printf("Connection accepted from %s:%d\n",
           inet_ntoa(client_addr.sin_addr), ntohs(client_addr.sin_port));

    // 6. 从客户端接收数据
    ssize_t recv_len;
    if ((recv_len = recv(client_sock, buffer, BUFFER_SIZE - 1, 0)) < 0) {
        perror("recv failed");
        close(client_sock);
        close(server_sock);
        exit(EXIT_FAILURE);
    }
    buffer[recv_len] = '\0'; // 在接收到的数据末尾添加字符串结束符
    printf("Received from client: %s\n", buffer);

    // 7. 向客户端发送数据
    if (send(client_sock, message, strlen(message), 0) < 0) {
        perror("send failed");
        close(client_sock);
        close(server_sock);
        exit(EXIT_FAILURE);
    }
    printf("Message sent to client: %s\n", message);

    // 8. 关闭套接字
    close(client_sock);
    close(server_sock);

    return 0;
}
```

### TCP客户端代码 (client.c)

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>
#include <netinet/in.h>

#define SERVER_IP "127.0.0.1"  // 服务器的IP地址（localhost）
#define SERVER_PORT 8000       // 服务器端口号
#define BUFFER_SIZE 1024       // 缓冲区大小

int main() {
    int sock;                            // 客户端套接字描述符
    struct sockaddr_in server_addr;      // 服务器地址结构体
    char buffer[BUFFER_SIZE];            // 数据缓冲区
    char *message = "Hello from client!"; // 要发送给服务器的消息

    // 1. 创建TCP套接字
    if ((sock = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
        perror("socket creation failed");
        exit(EXIT_FAILURE);
    }

    // 2. 初始化服务器地址结构
    memset(&server_addr, 0, sizeof(server_addr));
    server_addr.sin_family = AF_INET;                 // 使用IPv4
    server_addr.sin_addr.s_addr = inet_addr(SERVER_IP); // 设置服务器IP地址
    server_addr.sin_port = htons(SERVER_PORT);        // 设置服务器端口号

    // 3. 连接到服务器
    if (connect(sock, (struct sockaddr*)&server_addr, sizeof(server_addr)) < 0) {
        perror("connection failed");
        close(sock);
        exit(EXIT_FAILURE);
    }
    printf("Connected to server %s:%d\n", SERVER_IP, SERVER_PORT);

    // 4. 向服务器发送数据
    if (send(sock, message, strlen(message), 0) < 0) {
        perror("send failed");
        close(sock);
        exit(EXIT_FAILURE);
    }
    printf("Message sent to server: %s\n", message);

    // 5. 接收服务器的响应
    ssize_t recv_len;
    if ((recv_len = recv(sock, buffer, BUFFER_SIZE - 1, 0)) < 0) {
        perror("recv failed");
        close(sock);
        exit(EXIT_FAILURE);
    }
    buffer[recv_len] = '\0'; // 添加字符串结束符
    printf("Received from server: %s\n", buffer);

    // 6. 关闭套接字
    close(sock);

    return 0;
}
```

## 编译和运行

1.  **编译服务器和客户端**
    在终端中使用GCC编译器分别编译服务器和客户端代码：
    ```bash
    gcc -o server server.c
    gcc -o client client.c
    ```

2.  **运行程序**
    *   首先启动服务器，它会开始监听指定端口：
        ```bash
        ./server
        ```
    *   然后在另一个终端窗口中启动客户端，它会尝试连接服务器并进行通信：
        ```bash
        ./client
        ```

3.  **预期输出**
    *   **服务器端**:
        ```
        Server is listening on port 8000...
        Connection accepted from 127.0.0.1:some_port
        Received from client: Hello from client!
        Message sent to client: Hello from server!
        ```
    *   **客户端端**:
        ```
        Connected to server 127.0.0.1:8000
        Message sent to server: Hello from client!
        Received from server: Hello from server!
        ```

## 🛠️ 常见错误处理

-   **`EACCES` (权限不足)**: 绑定到知名端口（<1024）通常需要root权限。
-   **`EADDRINUSE` (地址已在使用)**: 端口已被其他进程占用。可使用 `setsockopt` 设置 `SO_REUSEADDR` 选项来避免。
-   **`ECONNREFUSED` (连接被拒绝)**: 目标端口没有进程监听。
-   **`ETIMEDOUT` (连接超时)**: 建立连接时超时。
-   总是检查Socket函数返回值并进行错误处理（使用 `perror` 或 `strerror(errno)`）。

## 📚 进阶学习建议

掌握了这些基础之后，你可以进一步探索：
*   **UDP编程**：使用 `SOCK_DGRAM` 和 `sendto()`/`recvfrom()` 函数实现无连接通信。
*   **I/O多路复用**：学习使用 `select`、`poll` 或 `epoll` 来处理高并发连接。
*   **协议设计**：思考如何设计应用层协议来解决粘包问题（如添加消息头、使用定长消息或分隔符）。
*   **网络安全**：了解SSL/TLS，实现加密通信。
