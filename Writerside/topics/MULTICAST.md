# MULTICAST

## 概念
Server可以将数据包只发送给指定组内的客户端，而不发送给指定组外的客户端。
特点：
- 可以在internet中进行组播
- 加入组播中的客户端才会收到消息

## 问题
1、如何划分组
2、如何在internet中进行
3、组播的的带宽限制是什么
4、是否保证传输

## 1、组播组
组播必须依赖于 IP 组播地址。范围从 224.0.0.0 -> 239.255.255.255.

<img alt="多播地址" height="700" src="multicast.png" width="600"/>

组播的 Server 端将 Message 发送到固定的组播地址，让加入组播组的用户收到 Message。  
组播的 Client 端需要创建一个本地 AF_INET 套接字，绑定到指定的 组播 IP 端口，
为了加入到组播组，还需要使用 setsockopt 函数加入到组播中，然后直接使用该 socket 进行通信。

## 2、组播的基本编码流程

### 2.1 确定组播地址和组播端口
根据自己的需要选择组播 IP 地址，确定一个组播 IP 端口。
```C++
constexpr std::string MULTI_ADDRESS = "239.0.0.1";
constexpr short MULTI_PORT = 8888;
```

### 2.2 服务端
组播的服务器是数据的 Publisher 端，它发送的 Message 可以被所有加入到该组播组中的客户端所
接收到。
```C++
#include <arpa/inet.h>
#include <string>
#include <iostream>
#include <thread>
#include <csignal>
#include <unistd.h>

constexpr const char *MULTI_ADDR = "239.0.0.1";
constexpr short MULTI_PORT = 8888;

bool flag{true};

void SigHandler(int sig) {
    std::cout << "deal sig exit!" << std::endl;
    flag = false;
}

int main() {
    signal(SIGTERM, SigHandler);
    
    // 创建套接字
    auto fd = socket(AF_INET, SOCK_DGRAM, 0);
    if (-1 == fd) {
        std::cerr << "socket error!" << std::endl;
        return -1;
    }
    
    // 设置该套接字可以发送组播
    struct in_addr imr_multiaddr{};
    inet_pton(AF_INET, MULTI_ADDR, &imr_multiaddr.s_addr);
    setsockopt(fd, IPPROTO_IP, IP_MULTICAST_IF, &imr_multiaddr, sizeof(imr_multiaddr));

    // 发送端需要指定将消息发送到某一个组播地址即可。
    struct sockaddr_in client{};
    client.sin_family = AF_INET;
    client.sin_port = htons(MULTI_PORT);
    inet_pton(AF_INET, MULTI_ADDR, &client.sin_addr.s_addr);

    int num = 0;
    std::string Msg = "hello ";

    while (flag) {
        auto SendMsg = Msg + std::to_string(num++);
        // 调用 sendto 函数发送数据到指定组播地址上
        sendto(fd, SendMsg.data(), SendMsg.size(), 0, (struct sockaddr *) &client, sizeof(client));
        std::cout << "组播消息: " << SendMsg << std::endl;
        std::this_thread::sleep_for(std::chrono::seconds(1));
    }
    close(fd);
    return 0;
}
```
### 2.3 客户端
组播客户端需要创建一个套接字，然后调用 setsockopt 函数将该套接字设置为广播属性，然后调用 bind 
函数绑定到客户端套接字上, 然后就可以正常接收到来自组播中的消息了。
```C++
#include <iostream>
#include <csignal>
#include <arpa/inet.h>
#include <unistd.h>
#include <fcntl.h>

bool b_exit{false};

void ControlExit(int number) {
    std::cout << "TERM exit!" << std::endl;
    b_exit = true;
}

constexpr const char *MULTI_ADDR = "239.0.0.1";
constexpr short MULTI_PORT = 8888;

int main() {
    signal(SIGTERM, ControlExit);
    signal(SIGINT, ControlExit);

    int fd = socket(AF_INET, SOCK_DGRAM, 0);
    if (-1 == fd) {
        std::cerr << "socket create error" << std::endl;
        return -1;
    }

    // 设置套接字为非阻塞
    int flags = fcntl(fd, F_GETFL, 0);
    fcntl(fd, F_SETFL, flags | O_NONBLOCK);

    // 设置设置套接字为组播套接字，并且指定要加入的组播地址
    struct ip_mreq merq{};
    inet_pton(AF_INET, MULTI_ADDR, &merq.imr_multiaddr.s_addr);
    merq.imr_interface.s_addr = htonl(INADDR_ANY);
    int ret = setsockopt(fd, IPPROTO_IP, IP_ADD_MEMBERSHIP, (const char *) &merq, sizeof(merq));
    if (-1 == ret) {
        std::cerr << "setsockopt error" << std::endl;
        return -1;
    }

    // 套接字绑定地址，多播端口，本地分配 IP 地址
    struct sockaddr_in addr{};
    addr.sin_family = AF_INET;
    addr.sin_port = htons(MULTI_PORT);
    addr.sin_addr.s_addr = INADDR_ANY;
    ret = bind(fd, (struct sockaddr *) &addr, sizeof(addr));
    if (-1 == ret) {
        std::cerr << "bind address error" << std::endl;
        return -1;
    }
    
    std::vector<char> RecvMsg;
    struct sockaddr_in Multiaddr{};
    socklen_t MultiLen = sizeof(Multiaddr);
    char CLientIP[INET_ADDRSTRLEN];
    RecvMsg.resize(100, 0);
    
    while (!b_exit) {
        auto len = recvfrom(fd, RecvMsg.data(), RecvMsg.size(), 0,
                            (struct sockaddr *) &Multiaddr, &MultiLen);

        if (len != -1) {
            // 读取发送端的 IP 地址
            inet_ntop(AF_INET, &Multiaddr.sin_addr, CLientIP, INET_ADDRSTRLEN);
            std::cout << ntohs(Multiaddr.sin_port) << std::endl;
            std::cout << CLientIP << std::endl;
            std::cout << "receive Msg: " << RecvMsg.data() << std::endl;
        }
    }
    close(fd);
}
```
