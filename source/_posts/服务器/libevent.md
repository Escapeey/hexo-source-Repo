---
title: libevent
date: 2024-01-08 20:20:56
tags: 
- c
- 网络编程
---
`libevent` 是一个轻量级、跨平台的事件通知库，专门用于高效管理大量并发 I/O 操作。它为基于事件驱动的应用程序提供了抽象层，使得程序可以统一使用不同平台上的 I/O 多路复用机制（如 `epoll`、`kqueue`、`select` 等），从而简化了开发工作，提升了可移植性。`libevent` 特别适合开发高性能的网络服务器、事件驱动的应用程序和实时系统。

### `libevent` 的特点：
1. **跨平台**：支持 Linux、Windows、macOS 以及其他 Unix 系统。根据系统的特性，`libevent` 会自动选择最佳的 I/O 多路复用机制，如 `epoll`（Linux）、`kqueue`（BSD、macOS）、`select`（通用）等。
2. **事件驱动**：提供基于事件的异步处理方式，支持 I/O、信号、定时器等多种事件源。
3. **高效的 I/O 多路复用**：对于大量并发的 I/O 处理，`libevent` 能够利用高效的 I/O 多路复用机制，实现高效的事件分发。
4. **支持定时器和信号事件**：除了 I/O 事件，`libevent` 还支持定时器事件和信号处理，使得它适用于更多类型的事件驱动应用。

### `libevent` 的事件类型：
`libevent` 支持以下几种常见的事件类型：
1. **I/O 事件**：监听文件描述符上的读、写、异常事件（如套接字的连接、数据传输）。
2. **定时器事件**：基于时间的事件处理，允许设置某个事件在一定时间后触发。
3. **信号事件**：捕捉和处理系统信号（如 `SIGINT`、`SIGTERM`）。
4. **持久事件**：事件触发后不会被自动移除，能够持续监听。

### `libevent` 的基本使用步骤：

1. **初始化事件基础结构**：
   在 `libevent` 中，首先要创建一个 `event_base`，它是管理和调度事件的核心结构体。

2. **创建事件**：
   通过 `event_new` 函数创建一个新的事件。事件可以是 I/O 事件、定时器事件或信号事件。

3. **添加事件到事件循环**：
   使用 `event_add` 将事件添加到事件循环中。

4. **启动事件循环**：
   调用 `event_base_dispatch` 或 `event_base_loop` 启动事件循环，程序会进入一个阻塞状态，等待事件触发。

5. **事件处理**：
   当事件触发时，`libevent` 调用关联的回调函数处理事件。

6. **清理**：
   在程序结束或不再需要事件时，可以使用 `event_free` 释放事件，并销毁 `event_base`。

### `libevent` 的 API 介绍：

#### 1. 初始化事件基础结构

```c
struct event_base *base = event_base_new();
```
`event_base_new()` 用来创建一个新的事件循环基础结构。`event_base` 是 `libevent` 的核心结构，用来管理所有的事件。

#### 2. 创建事件

```c
struct event *ev = event_new(base, fd, EV_READ|EV_PERSIST, callback_fn, arg);
```
`event_new` 用于创建一个事件，指定要监听的文件描述符 `fd`、事件类型 `EV_READ`（可读事件）、回调函数 `callback_fn` 以及传递给回调的参数 `arg`。`EV_PERSIST` 表示事件是持久的，不会在触发后自动删除。

- **文件描述符**：可以是套接字、文件或其他可以进行 I/O 操作的对象。
- **事件类型**：
  - `EV_READ`：监听文件描述符的读事件。
  - `EV_WRITE`：监听写事件。
  - `EV_PERSIST`：事件触发后不会移除，继续监听。

#### 3. 添加事件到事件循环

```c
event_add(ev, NULL);
```
`event_add` 将事件注册到事件循环中。第二个参数为超时时间，`NULL` 表示没有超时。

#### 4. 启动事件循环

```c
event_base_dispatch(base);
```
`event_base_dispatch` 开始事件循环，程序将阻塞在这里，直到有事件发生。

#### 5. 事件处理

```c
void callback_fn(evutil_socket_t fd, short event, void *arg) {
    // 处理事件的逻辑
}
```
事件发生时，会调用 `callback_fn` 回调函数。参数 `fd` 是触发事件的文件描述符，`event` 是触发的事件类型（读、写等），`arg` 是传递的用户数据。

#### 6. 清理资源

```c
event_free(ev);
event_base_free(base);
```
当不再需要某个事件或事件循环时，可以通过 `event_free` 和 `event_base_free` 来释放资源。

### `libevent` 示例代码：

以下是一个使用 `libevent` 创建简单的 TCP 服务器的例子，能够处理客户端连接并回显客户端发送的数据。

```c
#include <event2/event.h>
#include <event2/bufferevent.h>
#include <event2/buffer.h>
#include <event2/listener.h>
#include <arpa/inet.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

#define PORT 8080

// 回调函数：处理客户端发送的数据
void echo_read_cb(struct bufferevent *bev, void *ctx) {
    char buf[1024];
    size_t len = bufferevent_read(bev, buf, sizeof(buf));
    buf[len] = '\0';
    printf("Received message: %s\n", buf);
    // 回显消息给客户端
    bufferevent_write(bev, buf, len);
}

// 回调函数：处理客户端连接关闭
void echo_event_cb(struct bufferevent *bev, short events, void *ctx) {
    if (events & BEV_EVENT_EOF) {
        printf("Client disconnected.\n");
    } else if (events & BEV_EVENT_ERROR) {
        printf("Error on connection.\n");
    }
    bufferevent_free(bev);
}

// 回调函数：处理新客户端连接
void accept_conn_cb(struct evconnlistener *listener, evutil_socket_t fd, struct sockaddr *addr, int len, void *ctx) {
    struct event_base *base = evconnlistener_get_base(listener);
    struct bufferevent *bev = bufferevent_socket_new(base, fd, BEV_OPT_CLOSE_ON_FREE);
    bufferevent_setcb(bev, echo_read_cb, NULL, echo_event_cb, NULL);
    bufferevent_enable(bev, EV_READ | EV_WRITE);
}

int main() {
    struct event_base *base;
    struct evconnlistener *listener;
    struct sockaddr_in sin;

    base = event_base_new();  // 创建事件基础结构

    memset(&sin, 0, sizeof(sin));
    sin.sin_family = AF_INET;
    sin.sin_addr.s_addr = htonl(INADDR_ANY);
    sin.sin_port = htons(PORT);

    // 创建监听器，监听客户端连接
    listener = evconnlistener_new_bind(base, accept_conn_cb, NULL,
                                       LEV_OPT_REUSEABLE | LEV_OPT_CLOSE_ON_FREE, -1,
                                       (struct sockaddr*)&sin, sizeof(sin));

    if (!listener) {
        perror("Could not create listener");
        return 1;
    }

    // 运行事件循环
    event_base_dispatch(base);

    // 释放资源
    evconnlistener_free(listener);
    event_base_free(base);
    return 0;
}
```

### 主要函数解析：
- **`event_base_new()`**：创建事件循环的基础结构。
- **`evconnlistener_new_bind()`**：创建一个监听器，用于接受客户端连接。
- **`bufferevent_socket_new()`**：为每个连接创建一个缓冲区事件（`bufferevent`），用于处理 I/O。
- **`bufferevent_setcb()`**：设置 I/O 事件的回调函数。
- **`event_base_dispatch()`**：启动事件循环，处理所有的 I/O 事件。

### `libevent` 的优势：
1. **跨平台支持**：自动根据操作系统选择最佳的 I/O 多路复用机制。
2. **简化并发处理**：通过事件驱动模型处理多个客户端连接，无需手动创建线程或进程。
3. **支持多种事件类型**：不仅支持 I/O 事件，还支持定时器和信号事件，适合各种事件驱动的应用。
4. **高性能**：尤其适用于高并发场景，如网络服务器和实时系统。

### 应用场景：
- **高并发网络服务器**：如 Web 服务器、代理服务器。
- **实时系统**：需要同时处理大量 I/O 和定时器事件。
- **分布式系统**：处理多个节点的网络通信。 