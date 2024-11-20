---
title: select和epoll
tags:
  - c
  - 网络编程
abbrlink: e112f59a
date: 2024-01-08 19:51:20
---

# select
`select()` 是用于实现多路复用（I/O 多路复用）的系统调用，广泛应用于网络编程中。它允许服务器同时监听多个文件描述符（如套接字、文件或管道），并在这些文件描述符之一变为可读、可写或有错误时作出响应。这样可以避免为每个连接创建一个线程或进程，提升服务器并发性能。

### 函数定义

```c
#include <sys/select.h>

int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
```

### 参数解析：
1. **`nfds`**：
   - 需要监听的文件描述符数量，即所有文件描述符中最大的值加 1。这个参数用于告诉内核需要监听的文件描述符范围。

2. **`readfds`**：
   - 一个指向 `fd_set` 结构的指针，代表要检查是否有**可读**事件的文件描述符集合。如果没有需要检查的描述符，传递 `NULL`。

3. **`writefds`**：
   - 一个指向 `fd_set` 结构的指针，代表要检查是否有**可写**事件的文件描述符集合。如果没有需要检查的描述符，传递 `NULL`。

4. **`exceptfds`**：
   - 一个指向 `fd_set` 结构的指针，代表要检查是否有**异常**事件的文件描述符集合，如带外数据。如果没有需要检查的描述符，传递 `NULL`。

5. **`timeout`**：
   - 用于设定 `select()` 的超时时间。可以是以下三种情况：
     - `NULL`：永远阻塞，直到某个文件描述符上有事件发生。
     - 设定 `struct timeval` 值：设定阻塞的时间限制，超时后 `select()` 返回。
     - `timeval` 结构中的值为 0：表示立即返回，非阻塞模式。

### 返回值：
- **成功**：返回就绪的文件描述符的数量。
- **失败**：返回 `-1`，并设置 `errno`。
- **超时**：返回 `0`，表示在指定的时间内没有任何文件描述符变为可读、可写或发生异常。

### 核心数据结构 `fd_set`：
`fd_set` 是一个位集合（bitmap），用于表示文件描述符的集合。

相关操作函数：
- **`FD_ZERO(fd_set *set)`**：清空集合。
- **`FD_SET(int fd, fd_set *set)`**：将文件描述符 `fd` 添加到集合中。
- **`FD_CLR(int fd, fd_set *set)`**：从集合中移除文件描述符 `fd`。
- **`FD_ISSET(int fd, fd_set *set)`**：检查 `fd` 是否在集合中。

### 使用步骤：
1. 创建并初始化 `fd_set` 集合。
2. 使用 `FD_SET()` 将需要监控的文件描述符加入集合。
3. 调用 `select()` 进行事件监听。
4. 检查 `select()` 返回的集合，确定哪些文件描述符已准备好进行 I/O 操作。

### 使用示例：

以下是一个简单的服务器使用 `select()` 同时监听多个客户端连接的例子：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/select.h>
#include <sys/socket.h>

#define PORT 8080
#define MAX_CLIENTS 30

int main() {
    int server_sock, client_sock, max_sd, sd, activity;
    int client_sockets[MAX_CLIENTS] = {0};
    struct sockaddr_in server_addr, client_addr;
    fd_set readfds;
    socklen_t addrlen = sizeof(client_addr);

    // 创建套接字
    server_sock = socket(AF_INET, SOCK_STREAM, 0);
    if (server_sock == 0) {
        perror("Socket creation failed");
        exit(EXIT_FAILURE);
    }

    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = INADDR_ANY;
    server_addr.sin_port = htons(PORT);

    // 绑定地址和端口
    if (bind(server_sock, (struct sockaddr *)&server_addr, sizeof(server_addr)) < 0) {
        perror("Bind failed");
        close(server_sock);
        exit(EXIT_FAILURE);
    }

    // 监听连接
    if (listen(server_sock, 3) < 0) {
        perror("Listen failed");
        close(server_sock);
        exit(EXIT_FAILURE);
    }

    printf("Server is listening on port %d\n", PORT);

    while (1) {
        // 清空读描述符集合
        FD_ZERO(&readfds);

        // 将服务器套接字加入集合
        FD_SET(server_sock, &readfds);
        max_sd = server_sock;

        // 将所有客户端套接字加入集合
        for (int i = 0; i < MAX_CLIENTS; i++) {
            sd = client_sockets[i];

            // 有效的套接字则加入集合
            if (sd > 0) {
                FD_SET(sd, &readfds);
            }

            // 找到最大的文件描述符
            if (sd > max_sd) {
                max_sd = sd;
            }
        }

        // 等待文件描述符上有活动
        activity = select(max_sd + 1, &readfds, NULL, NULL, NULL);

        if (activity < 0) {
            perror("Select error");
            continue;
        }

        // 检查是否有新的客户端连接
        if (FD_ISSET(server_sock, &readfds)) {
            if ((client_sock = accept(server_sock, (struct sockaddr *)&client_addr, &addrlen)) < 0) {
                perror("Accept failed");
                exit(EXIT_FAILURE);
            }

            printf("New connection: socket fd is %d, IP is %s, port is %d\n", client_sock,
                   inet_ntoa(client_addr.sin_addr), ntohs(client_addr.sin_port));

            // 将新连接加入客户端套接字数组
            for (int i = 0; i < MAX_CLIENTS; i++) {
                if (client_sockets[i] == 0) {
                    client_sockets[i] = client_sock;
                    printf("Adding client socket to list at index %d\n", i);
                    break;
                }
            }
        }

        // 检查哪个客户端发送了数据
        for (int i = 0; i < MAX_CLIENTS; i++) {
            sd = client_sockets[i];

            if (FD_ISSET(sd, &readfds)) {
                char buffer[1024];
                int valread = read(sd, buffer, sizeof(buffer));

                if (valread == 0) {
                    // 客户端关闭连接
                    printf("Client disconnected: socket fd %d\n", sd);
                    close(sd);
                    client_sockets[i] = 0;
                } else {
                    // 回显接收到的数据
                    buffer[valread] = '\0';
                    printf("Received from client: %s\n", buffer);
                    send(sd, buffer, strlen(buffer), 0);
                }
            }
        }
    }

    return 0;
}
```

### 代码解读：

1. **创建和绑定服务器套接字**：创建套接字后，绑定到指定的端口并开始监听客户端连接。

2. **`fd_set` 处理**：使用 `FD_SET` 将服务器套接字和客户端套接字添加到 `readfds` 集合中，表示我们关心这些套接字上的可读事件。

3. **`select()` 调用**：`select()` 会阻塞直到某个文件描述符有事件发生（即有可读、可写或异常事件），在这里我们只关心可读事件。

4. **处理新连接**：如果服务器套接字上有事件发生，表示有新的客户端连接到来，我们通过 `accept()` 接收新的连接并将其加入客户端套接字数组中。

5. **处理客户端通信**：遍历所有客户端套接字，使用 `FD_ISSET` 判断是否有客户端发送数据，如果有，则读取数据并回显给客户端。

### 优点：
- **低开销**：相比为每个客户端创建一个线程，`select()` 仅使用一个线程处理多个连接，减少了上下文切换的开销。
- **跨平台性**：`select()` 函数在几乎所有平台上都可用。

### 缺点：
- **文件描述符限制**：`select()` 的文件描述符数量有限（通常是 1024），无法处理非常大量的连接。
- **性能**：对于大规模并发连接，`select()` 每次都需要遍历整个文件描述符集，性能不佳。

在现代高并发场景中，`select()` 通常会被 `epoll` 或 `kqueue` 这些更高效的 I/O 多路复用机制替代，但 `select()` 仍然是一种经典且常用的技术。

# I/O多路复用

I/O 多路复用（I/O Multiplexing）是一种允许程序同时监视多个 I/O 操作（如读写文件、网络套接字、管道等）的技术。当其中某个 I/O 操作准备好时，操作系统会通知程序执行相应的操作。这种机制使得一个线程或进程能够管理多个 I/O 通道，而不用为每个 I/O 操作创建单独的线程或进程。

### I/O 多路复用的背景：
通常情况下，I/O 操作（如读写网络、文件）是阻塞的，即如果一个程序尝试从某个套接字或文件中读取数据，而数据尚未到达，它将进入阻塞状态，直到数据到达为止。这种情况在服务器中尤为常见，服务器要处理多个客户端的连接，如果为每个客户端的 I/O 操作都创建一个线程，系统资源消耗巨大且效率低下。

I/O 多路复用可以解决这一问题，它允许程序在一个线程内管理多个 I/O 通道，并在某个通道有事件（如数据可读、可写等）时进行处理。

### I/O 多路复用的工作方式：
1. **多个 I/O 通道**：程序可以监听多个套接字、文件或其他类型的 I/O 通道。
2. **阻塞与非阻塞**：通过 I/O 多路复用，程序可以避免阻塞在某个 I/O 操作上，而是监听所有通道，只有当某个通道有事件时才会处理，其他通道仍然可以继续监听。
3. **事件通知**：I/O 多路复用会等待多个文件描述符上的事件（如读、写、异常），当其中之一就绪时，内核会通知程序，从而进行相应的读写操作。

### 常见的 I/O 多路复用机制：

1. **`select`**：
   - 最早的多路复用机制，支持几乎所有操作系统。
   - 每次调用 `select()` 时都需要重新初始化文件描述符集，效率相对较低。
   - 文件描述符数量有限（通常是 1024），无法处理大量并发连接。
   
2. **`poll`**：
   - 和 `select()` 类似，但没有文件描述符数量的限制。
   - 需要重新遍历整个文件描述符集，性能在大量连接情况下不佳。

3. **`epoll`**（Linux 特有）：
   - 专为高并发场景设计，效率远高于 `select` 和 `poll`。
   - 采用事件驱动机制，事件就绪时才触发通知，避免了不必要的遍历。
   - 适合处理大量并发连接，是现代 Linux 服务器常用的 I/O 多路复用方式。

4. **`kqueue`**（BSD 系统，包括 macOS）：
   - 类似于 `epoll`，支持高效的事件通知机制。
   - 提供更多类型的事件监控（如文件系统事件、进程事件等）。

### I/O 多路复用的工作流程：

1. **创建监听的文件描述符**：
   - 通过 `socket()` 或其他方法创建多个文件描述符，如监听客户端连接的套接字。

2. **注册 I/O 事件**：
   - 使用多路复用机制（如 `select()`、`epoll` 或 `poll`）注册需要监听的文件描述符及其关注的事件（如可读、可写）。

3. **等待事件发生**：
   - 多路复用调用将阻塞，直到有事件发生或超时。

4. **处理就绪的文件描述符**：
   - 当某个文件描述符准备好（如某个套接字有数据可读），多路复用调用返回，程序可以处理该事件。

5. **重复步骤 2-4**，持续监听文件描述符并处理事件。

### 使用场景：
I/O 多路复用特别适合用于需要处理大量并发 I/O 请求的场景。典型应用包括：

- **网络服务器**：如 Web 服务器或数据库服务器，它们需要同时处理成百上千个客户端的连接请求，I/O 多路复用可以避免为每个连接创建一个线程或进程，节省系统资源。
- **聊天服务器**：聊天服务器需要同时处理多个客户端的消息发送和接收。
- **高效日志系统**：需要同时监视多个文件或设备的变化。

### 优点：
1. **高效**：可以让一个线程同时监控多个 I/O 通道，避免了多线程或多进程带来的上下文切换开销。
2. **灵活性**：可以同时监视多个不同类型的 I/O 设备（如网络、文件、管道）。

### 缺点：
1. **复杂性**：代码相对复杂，尤其是在处理高并发场景时，需要额外的逻辑来管理文件描述符。
2. **性能瓶颈**：对于大量并发连接，`select` 和 `poll` 的效率会随着文件描述符数量的增加而下降，`epoll` 和 `kqueue` 能在高并发场景中表现更好。

### 总结：
I/O 多路复用是一种关键技术，能够在单个线程或进程中同时处理多个 I/O 操作。通过 `select`、`poll` 或更高效的 `epoll` 和 `kqueue`，服务器可以高效处理多个客户端连接，尤其适用于高并发的网络服务。

# epoll

`epoll` 是 Linux 系统中用于 I/O 多路复用的高效机制，专门为大规模并发连接设计。与传统的 `select` 和 `poll` 相比，`epoll` 具有更好的性能和扩展性，特别是在处理大量文件描述符（如网络套接字）时。它采用事件驱动的模型，不需要像 `select` 和 `poll` 那样反复遍历整个文件描述符集，从而大幅提升了效率。

### `epoll` 的优点：
1. **高效的事件通知**：`epoll` 使用事件驱动模式，当有事件发生时才通知应用程序，避免不必要的文件描述符遍历。
2. **支持大量文件描述符**：相比于 `select` 的 1024 文件描述符限制，`epoll` 可以处理几乎无限数量的文件描述符，适用于大规模并发场景。
3. **边缘触发（Edge-triggered）与水平触发（Level-triggered）**：`epoll` 提供两种事件通知模式，边缘触发适合高效、非阻塞的 I/O 操作，而水平触发则更加传统。

### `epoll` 的工作原理：
`epoll` 是基于内核的事件通知机制，包含三个核心操作：
1. **`epoll_create`**：创建一个 `epoll` 实例，用于管理文件描述符。
2. **`epoll_ctl`**：向 `epoll` 实例中添加、修改或删除文件描述符，指定关注的事件类型（如可读、可写、异常）。
3. **`epoll_wait`**：等待事件的发生，当某个文件描述符有事件（如数据到达）时，内核会通知应用程序，返回该文件描述符列表。

### `epoll` 函数原型：

```c
int epoll_create1(int flags);
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
```

1. **`epoll_create1(int flags)`**：创建一个 `epoll` 实例，返回一个文件描述符。
   - `flags`：可以为 `0` 或者 `EPOLL_CLOEXEC`（在 `fork()` 后关闭文件描述符）。
   - 返回值：如果成功，返回 `epoll` 实例的文件描述符；失败返回 `-1`。

2. **`epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)`**：对 `epoll` 实例进行控制操作，添加、修改或删除监听的文件描述符。
   - `epfd`：`epoll` 实例的文件描述符。
   - `op`：控制操作类型，可以是以下三种：
     - `EPOLL_CTL_ADD`：向 `epoll` 实例中添加文件描述符。
     - `EPOLL_CTL_MOD`：修改已经在 `epoll` 实例中的文件描述符的事件类型。
     - `EPOLL_CTL_DEL`：从 `epoll` 实例中删除文件描述符。
   - `fd`：要监听的文件描述符。
   - `event`：指定监听的事件类型。

3. **`epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout)`**：等待 `epoll` 实例中注册的事件发生。
   - `epfd`：`epoll` 实例的文件描述符。
   - `events`：用于存储返回的事件列表。
   - `maxevents`：一次可以返回的最大事件数。
   - `timeout`：等待的超时时间，单位为毫秒；`-1` 表示阻塞等待，`0` 表示立即返回。

### `epoll_event` 结构体：

```c
struct epoll_event {
    uint32_t events;    // 事件类型，例如 EPOLLIN, EPOLLOUT
    epoll_data_t data;  // 用户数据，可以用来存储文件描述符
};
```

- **`events`**：指定文件描述符上监听的事件，如 `EPOLLIN`（可读）、`EPOLLOUT`（可写）、`EPOLLERR`（错误）、`EPOLLET`（边缘触发）。
- **`data`**：用于存储用户数据，通常存储文件描述符。

### `epoll` 触发模式：
1. **水平触发（Level-triggered, LT）**：
   - 这是 `epoll` 的默认模式，类似于 `select` 和 `poll`，当文件描述符准备好后，每次调用 `epoll_wait` 都会返回该事件，直到事件被处理完。
   - 使用简单，适合大部分场景。
   
2. **边缘触发（Edge-triggered, ET）**：
   - 边缘触发模式只在状态从不可用到可用时触发事件。换句话说，如果数据已经可读，再次调用 `epoll_wait` 时不会通知，除非有新的数据到达。
   - 边缘触发更高效，减少了重复通知的次数，但编程更复杂，要求必须使用非阻塞 I/O。

### `epoll` 使用步骤：

1. **创建 `epoll` 实例**：
   ```c
   int epfd = epoll_create1(0);
   ```

2. **注册文件描述符**：
   使用 `epoll_ctl` 将需要监听的文件描述符注册到 `epoll` 实例中。
   ```c
   struct epoll_event ev;
   ev.events = EPOLLIN;  // 监听可读事件
   ev.data.fd = listen_fd;  // 文件描述符
   epoll_ctl(epfd, EPOLL_CTL_ADD, listen_fd, &ev);
   ```

3. **等待事件**：
   调用 `epoll_wait` 等待文件描述符上发生的事件。
   ```c
   struct epoll_event events[10];
   int nfds = epoll_wait(epfd, events, 10, -1);
   for (int i = 0; i < nfds; i++) {
       if (events[i].events & EPOLLIN) {
           // 处理可读事件
       }
   }
   ```

### `epoll` 示例代码：
下面是一个使用 `epoll` 的简单服务器程序，它可以监听多个客户端连接，并处理客户端发送的数据。

```c
#include <sys/epoll.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <string.h>

#define PORT 8080
#define MAX_EVENTS 10

int main() {
    int listen_fd, conn_fd, epfd;
    struct sockaddr_in server_addr;
    struct epoll_event ev, events[MAX_EVENTS];

    // 创建监听套接字
    listen_fd = socket(AF_INET, SOCK_STREAM, 0);
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = INADDR_ANY;
    server_addr.sin_port = htons(PORT);

    bind(listen_fd, (struct sockaddr *)&server_addr, sizeof(server_addr));
    listen(listen_fd, 10);

    // 创建 epoll 实例
    epfd = epoll_create1(0);
    if (epfd == -1) {
        perror("epoll_create1");
        exit(EXIT_FAILURE);
    }

    // 添加监听套接字到 epoll
    ev.events = EPOLLIN;
    ev.data.fd = listen_fd;
    if (epoll_ctl(epfd, EPOLL_CTL_ADD, listen_fd, &ev) == -1) {
        perror("epoll_ctl: listen_fd");
        exit(EXIT_FAILURE);
    }

    while (1) {
        int nfds = epoll_wait(epfd, events, MAX_EVENTS, -1);
        for (int i = 0; i < nfds; i++) {
            if (events[i].data.fd == listen_fd) {
                // 新客户端连接
                conn_fd = accept(listen_fd, NULL, NULL);
                ev.events = EPOLLIN;
                ev.data.fd = conn_fd;
                epoll_ctl(epfd, EPOLL_CTL_ADD, conn_fd, &ev);
                printf("New connection accepted\n");
            } else {
                // 处理客户端发送的数据
                char buf[1024];
                int len = read(events[i].data.fd, buf, sizeof(buf));
                if (len > 0) {
                    buf[len] = '\0';
                    printf("Received: %s\n", buf);
                    write(events[i].data.fd, buf, len);
                } else {
                    close(events[i].data.fd);
                    printf("Client disconnected\n");
                }
            }
        }
    }

    close(listen_fd);
    close(epfd);
    return 0;
}
```

### `epoll` 与 `select` 的对比：

| 特性            | `select`                   | `epoll`                      |
|-----------------|----------------------------|------------------------------|
| 文件描述符限制  | 最大 1024 个（可调整）      | 文件描述符几乎无限制         |
| 事件处理模式    | 每次调用都要遍历整个集合    | 只处理有事件的文件