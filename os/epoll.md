# epoll

## 1. 简介
epoll 是 Linux 内核提供的一种高效的 I/O 事件通知机制，适用于需要处理大量并发连接的场景。它的主要作用是监控多个文件描述符，以便在这些文件描述符上有事件发生时通知应用程序。epoll 相比于传统的 select 和 poll，在处理大量文件描述符时具有更高的性能和更好的扩展性。

## 2. 数据结构
epoll 实例：每个 epoll 实例在内核中对应一个 struct eventpoll 结构体，它包含了所有需要监控的文件描述符及其事件。
红黑树：用于存储所有被监控的文件描述符及其事件。红黑树的特点是插入、删除和查找操作的时间复杂度都是 O(log N)。
双向链表：用于存储已经准备好的事件。双向链表的特点是插入和删除操作的时间复杂度都是 O(1)。

## 3. 系统调用
epoll 提供了三个主要的系统调用：
- epoll_create：创建一个 epoll 实例，返回一个文件描述符。
- epoll_ctl：用于控制 epoll 实例，添加、修改或删除监控的文件描述符。
- epoll_wait：等待事件的发生，并返回已经准备好的事件。

## 4. 工作流程
1. 创建 epoll 实例：
- 调用 epoll_create 创建一个 epoll 实例，内核会分配一个 struct eventpoll 结构体，并初始化红黑树和双向链表。

2. 添加/修改/删除文件描述符：
- 调用 epoll_ctl 可以将文件描述符添加到 epoll 实例的红黑树中，或者修改/删除已经存在的文件描述符。
- 每个文件描述符会关联一个 epitem 结构体，包含文件描述符和事件信息。

3. 等待事件：
- 调用 epoll_wait，内核会检查双向链表中是否有已经准备好的事件。
- 如果有，直接返回这些事件。
- 如果没有，内核会将当前进程/线程挂起，直到有事件发生。
- 当文件描述符的状态发生变化时（例如可读、可写或错误），内核会将对应的 epitem 结构体从红黑树移动到双向链表中，并唤醒等待的进程/线程。

## 5. 内核通知机制
- 当文件描述符的状态发生变化时，内核会调用回调函数，将对应的 epitem 结构体从红黑树移动到双向链表中。
- 这种回调机制使得 epoll 可以高效地处理大量文件描述符，而不需要遍历所有文件描述符。

## 6. 优化
- 边缘触发（Edge Triggered, ET）：epoll 支持边缘触发模式，只有在文件描述符状态从未准备好变为准备好时才通知用户。这种模式下，用户需要一次性处理所有可用的数据。
- 水平触发（Level Triggered, LT）：默认模式，文件描述符只要处于准备好状态，就会不断通知用户。

## 7.完整例子
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <errno.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <sys/epoll.h>

#define MAX_EVENTS 10
#define PORT 8080
#define BUFFER_SIZE 1024

void handle_connection(int client_fd) {
    char buffer[BUFFER_SIZE];
    ssize_t bytes_read;

    bytes_read = read(client_fd, buffer, sizeof(buffer) - 1);
    if (bytes_read == -1) {
        perror("read");
        close(client_fd);
        return;
    } else if (bytes_read == 0) {
        // Client closed the connection
        printf("Client disconnected\n");
        close(client_fd);
        return;
    }

    buffer[bytes_read] = '\0';
    printf("Received: %s\n", buffer);

    // Echo the message back to the client
    if (write(client_fd, buffer, bytes_read) == -1) {
        perror("write");
        close(client_fd);
    }
}

int main() {
    int listen_fd, epoll_fd;
    struct sockaddr_in server_addr;
    struct epoll_event ev, events[MAX_EVENTS];

    // Create listening socket
    listen_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (listen_fd == -1) {
        perror("socket");
        exit(EXIT_FAILURE);
    }

    // Set up server address
    memset(&server_addr, 0, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = INADDR_ANY;
    server_addr.sin_port = htons(PORT);

    // Bind socket to address
    if (bind(listen_fd, (struct sockaddr *)&server_addr, sizeof(server_addr)) == -1) {
        perror("bind");
        close(listen_fd);
        exit(EXIT_FAILURE);
    }

    // Start listening for connections
    if (listen(listen_fd, SOMAXCONN) == -1) {
        perror("listen");
        close(listen_fd);
        exit(EXIT_FAILURE);
    }

    // Create epoll instance
    epoll_fd = epoll_create1(0);
    if (epoll_fd == -1) {
        perror("epoll_create1");
        close(listen_fd);
        exit(EXIT_FAILURE);
    }

    // Add listening socket to epoll
    ev.events = EPOLLIN;
    ev.data.fd = listen_fd;
    if (epoll_ctl(epoll_fd, EPOLL_CTL_ADD, listen_fd, &ev) == -1) {
        perror("epoll_ctl: listen_fd");
        close(listen_fd);
        close(epoll_fd);
        exit(EXIT_FAILURE);
    }

    printf("Server is listening on port %d\n", PORT);

    while (1) {
        int nfds = epoll_wait(epoll_fd, events, MAX_EVENTS, -1);
        if (nfds == -1) {
            perror("epoll_wait");
            close(listen_fd);
            close(epoll_fd);
            exit(EXIT_FAILURE);
        }

        for (int i = 0; i < nfds; ++i) {
            if (events[i].data.fd == listen_fd) {
                // New incoming connection
                struct sockaddr_in client_addr;
                socklen_t client_addr_len = sizeof(client_addr);
                int client_fd = accept(listen_fd, (struct sockaddr *)&client_addr, &client_addr_len);
                if (client_fd == -1) {
                    perror("accept");
                    continue;
                }

                printf("Accepted connection from %s:%d\n",
                       inet_ntoa(client_addr.sin_addr), ntohs(client_addr.sin_port));

                // Add new client socket to epoll
                ev.events = EPOLLIN;
                ev.data.fd = client_fd;
                if (epoll_ctl(epoll_fd, EPOLL_CTL_ADD, client_fd, &ev) == -1) {
                    perror("epoll_ctl: client_fd");
                    close(client_fd);
                }
            } else {
                // Handle data from client
                handle_connection(events[i].data.fd);
            }
        }
    }

    close(listen_fd);
    close(epoll_fd);
    return 0;
}
```

### 代码说明
1. 创建监听套接字：
- 使用 socket 创建一个 TCP 套接字。
- 使用 bind 将套接字绑定到指定的端口。
- 使用 listen 开始监听连接。

2. 创建 epoll 实例：
- 使用 epoll_create1 创建一个 epoll 实例。
- 将监听套接字添加到 epoll 实例中，监听 EPOLLIN 事件。

3. 事件循环：
- 使用 epoll_wait 等待事件发生。
- 如果监听套接字有事件，表示有新的连接到来，使用 accept 接受连接，并将新连接的套接字添加到 epoll 实例中。
- 如果是客户端套接字有事件，表示有数据可读，调用 handle_connection 处理数据。

4. 处理客户端连接：
- 读取客户端发送的数据，并回显给客户端。
- 如果读取失败或客户端关闭连接，关闭套接字。

### 编译和运行
将上述代码保存为 epoll_server.c，然后使用以下命令编译和运行：

```sh
gcc -o epoll_server epoll_server.c
./epoll_server
```

服务器启动后，可以使用 telnet 或其他工具连接到服务器，并发送消息进行测试。
```sh
telnet ip port
```

## 8. macos 上的 kqueue
epoll 是 Linux 特有的 I/O 多路复用机制，macOS 并不支持 epoll。在 macOS 上，可以使用其他 I/O 多路复用机制，如 kqueue、select 和 poll。其中，kqueue 是 macOS 和其他 BSD 系统提供的高效 I/O 事件通知机制，功能和性能上与 epoll 类似。

### kqueue 使用示例
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <errno.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <sys/event.h>
#include <sys/time.h>

#define MAX_EVENTS 10
#define PORT 8080
#define BUFFER_SIZE 1024

void handle_connection(int client_fd) {
    char buffer[BUFFER_SIZE];
    ssize_t bytes_read;

    bytes_read = read(client_fd, buffer, sizeof(buffer) - 1);
    if (bytes_read == -1) {
        perror("read");
        close(client_fd);
        return;
    } else if (bytes_read == 0) {
        // Client closed the connection
        printf("Client disconnected\n");
        close(client_fd);
        return;
    }

    buffer[bytes_read] = '\0';
    printf("Received: %s\n", buffer);

    // Echo the message back to the client
    if (write(client_fd, buffer, bytes_read) == -1) {
        perror("write");
        close(client_fd);
    }
}

int main() {
    int listen_fd, kq;
    struct sockaddr_in server_addr;
    struct kevent ev_set, ev_list[MAX_EVENTS];

    // Create listening socket
    listen_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (listen_fd == -1) {
        perror("socket");
        exit(EXIT_FAILURE);
    }

    // Set up server address
    memset(&server_addr, 0, sizeof(server_addr));
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = INADDR_ANY;
    server_addr.sin_port = htons(PORT);

    // Bind socket to address
    if (bind(listen_fd, (struct sockaddr *)&server_addr, sizeof(server_addr)) == -1) {
        perror("bind");
        close(listen_fd);
        exit(EXIT_FAILURE);
    }

    // Start listening for connections
    if (listen(listen_fd, SOMAXCONN) == -1) {
        perror("listen");
        close(listen_fd);
        exit(EXIT_FAILURE);
    }

    // Create kqueue instance
    kq = kqueue();
    if (kq == -1) {
        perror("kqueue");
        close(listen_fd);
        exit(EXIT_FAILURE);
    }

    // Add listening socket to kqueue
    EV_SET(&ev_set, listen_fd, EVFILT_READ, EV_ADD, 0, 0, NULL);
    if (kevent(kq, &ev_set, 1, NULL, 0, NULL) == -1) {
        perror("kevent: listen_fd");
        close(listen_fd);
        close(kq);
        exit(EXIT_FAILURE);
    }

    printf("Server is listening on port %d\n", PORT);

    while (1) {
        int nev = kevent(kq, NULL, 0, ev_list, MAX_EVENTS, NULL);
        if (nev == -1) {
            perror("kevent");
            close(listen_fd);
            close(kq);
            exit(EXIT_FAILURE);
        }

        for (int i = 0; i < nev; ++i) {
            if (ev_list[i].ident == listen_fd) {
                // New incoming connection
                struct sockaddr_in client_addr;
                socklen_t client_addr_len = sizeof(client_addr);
                int client_fd = accept(listen_fd, (struct sockaddr *)&client_addr, &client_addr_len);
                if (client_fd == -1) {
                    perror("accept");
                    continue;
                }

                printf("Accepted connection from %s:%d\n",
                       inet_ntoa(client_addr.sin_addr), ntohs(client_addr.sin_port));

                // Add new client socket to kqueue
                EV_SET(&ev_set, client_fd, EVFILT_READ, EV_ADD, 0, 0, NULL);
                if (kevent(kq, &ev_set, 1, NULL, 0, NULL) == -1) {
                    perror("kevent: client_fd");
                    close(client_fd);
                }
            } else {
                // Handle data from client
                handle_connection(ev_list[i].ident);
            }
        }
    }

    close(listen_fd);
    close(kq);
    return 0;
}
```

### 代码说明
1. 创建监听套接字：
- 使用 socket 创建一个 TCP 套接字。
- 使用 bind 将套接字绑定到指定的端口。
- 使用 listen 开始监听连接。

2. 创建 kqueue 实例：
- 使用 kqueue 创建一个 kqueue 实例。
- 将监听套接字添加到 kqueue 实例中，监听 EVFILT_READ 事件。

3. 事件循环：
- 使用 kevent 等待事件发生。
- 如果监听套接字有事件，表示有新的连接到来，使用 accept 接受连接，并将新连接的套接字添加到 kqueue 实例中。
- 如果是客户端套接字有事件，表示有数据可读，调用 handle_connection 处理数据。

4. 处理客户端连接：
- 读取客户端发送的数据，并回显给客户端。
- 如果读取失败或客户端关闭连接，关闭套接字。

### 编译和运行
将上述代码保存为 kqueue_server.c，然后使用以下命令编译和运行：
```sh
gcc -o kqueue_server kqueue_server.c
./kqueue_server
```

服务器启动后，可以使用 nc 或其他工具连接到服务器，并发送消息进行测试。
```sh
nc ip port
```