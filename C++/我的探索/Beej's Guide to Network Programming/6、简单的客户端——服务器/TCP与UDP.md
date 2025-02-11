# TCP
### 服务器
```cpp
#include <memory>  
#include <string>  
#include <arpa/inet.h>  
#include <sys/socket.h>  
#include <sys/types.h>  
#include <netinet/in.h>  
#include <unistd.h>  
#include  <cstring>  
#include <netdb.h>  
#include <iostream>  
  
int main() {  
    // 1、获取地址通配  
    addrinfo *result;  
    auto *hint = new addrinfo;  
  
    memset(hint, 0, sizeof(*hint));  
    hint->ai_family = AF_UNSPEC;  
    hint->ai_socktype = SOCK_STREAM;  
    hint->ai_flags = AI_PASSIVE;  
  
    if (getaddrinfo(nullptr, "80", hint,&result) != 0) {  
        perror("getaddrinfo error!");  
        exit(1);  
    }  
  
    delete hint;  
  
    // 建立监听套接字并绑定端口  
    int listen_sock = -1;  
    addrinfo* p;  
    for (p = result; p != nullptr; p = p->ai_next) {  
        // 建立监听套接字  
        if ((listen_sock = socket(p->ai_family, p->ai_socktype, p->ai_protocol)) == 0) {  
            perror("socket create error");  
            continue;  
        }  
  
        int temp = 1; // 设置套接字选项  
        if (setsockopt(listen_sock, SOL_SOCKET, SO_REUSEADDR, &temp, sizeof(int)) != 0) {  
            perror("setsockopt");  
            exit(1);  
        }  
  
        if (bind(listen_sock, p->ai_addr, p->ai_addrlen) != 0) { // 绑定地址  
            perror("bind");  
            continue;  
        }  
  
        break; // 找到第一个没问题的地址，然后给这个地址创建套接字并绑定该地址与端口（这里是万能匹配地址）  
    }  
    freeaddrinfo(result); // 找完了立刻释放  
  
    if (p == nullptr) {  
        std::cout << "遍历了所有，没有找到合适的" << std::endl;  
        exit(1);  
    }  
  
    if (listen(listen_sock, 20) != 0) {  
        perror("listen");  
        exit(1);  
    }  
  
    while (1) {  
        // accept  
        sockaddr_storage others_addr = {};  
        socklen_t others_addr_len = sizeof(others_addr);  
  
        const int accept_sock = accept(listen_sock, reinterpret_cast<sockaddr *>(&others_addr), &others_addr_len);  
        if (accept_sock == -1) {  
            perror("accept_sock");  
            continue;  
        }  
  
        // 1、打印对方的地址信息  
        char other_addr_message[INET6_ADDRSTRLEN];  
        memset(other_addr_message, ' ', 1024);  
        if (others_addr.ss_family == AF_INET) {  
            const auto* temp_in = (sockaddr_in*)&others_addr;  
            inet_ntop(PF_INET, &temp_in->sin_addr, other_addr_message, sizeof(other_addr_message));  
        }else {  
            const auto* temp_in6 = (sockaddr_in6*)&others_addr;  
            inet_ntop(PF_INET6, &temp_in6->sin6_addr, other_addr_message, sizeof(other_addr_message));  
        }  
        std::cout << other_addr_message << std::endl;  
  
        // 给对方发送信息  
        std::string message = "hello, how are you";  
        if (send(accept_sock, message.c_str(), message.size(), MSG_DONTWAIT) < 0) {  
            perror("send");  
            close(accept_sock);  
        }  
  
        // 接受消息  
        char mes[500];  
        int receive_length = 0;  
        if ((receive_length = recv(accept_sock, mes, sizeof(mes), 0)) <0 ) {  
            perror("recv");  
            close(accept_sock);  
        }  
        std::string get_message(mes, receive_length); // 这一步很重要  
        std::cout << get_message << std::endl;  
    }  
}
```

### 客户端
```cpp
// 客户端  
#include <iostream>  
#include <string>  
#include <sys/socket.h>  
#include <netinet/in.h>  
#include <arpa/inet.h>  
  
  
  
int main() {  
    int sock = socket(AF_INET, SOCK_STREAM, 0);  
  
    // 直接指定  
    sockaddr_in* myaddr = new sockaddr_in;  
    myaddr->sin_family = AF_INET;  
    myaddr->sin_addr.s_addr = inet_addr("127.0.0.1");  
    myaddr->sin_port = htons(8080);  
  
    connect(sock, reinterpret_cast<sockaddr *>(myaddr), sizeof(*myaddr));  
  
    const std::string message = "helloworld";  
  
    send(sock, message.c_str(),message.size(),0);  
  
    char receive_message[1024];  
    int receive_length = recv(sock, receive_message, sizeof(receive_message), 0);  
    if (receive_length < 0) {  
        perror("recevie error");  
    }  
    std::string acctual_message(receive_message, receive_length);  
    std::cout << acctual_message << std::endl;  
  
  
}
```


# UDP
### 服务器
```cpp
// 服务器  
#include <memory>  
#include <string>  
#include <arpa/inet.h>  
#include <sys/socket.h>  
#include <sys/types.h>  
#include <netinet/in.h>  
#include <unistd.h>  
#include  <cstring>  
#include <netdb.h>  
#include <iostream>  
  
int main() {  
    // 1、获取地址通配  
    addrinfo *result;  
    auto *hint = new addrinfo;  
  
    memset(hint, 0, sizeof(*hint));  
    hint->ai_family = AF_UNSPEC;  
    hint->ai_socktype = SOCK_DGRAM;  
    hint->ai_flags = AI_PASSIVE;  
  
    if (getaddrinfo(nullptr, "8080", hint,&result) != 0) {  
        perror("getaddrinfo error!");  
        exit(1);  
    }  
  
    delete hint;  
  
    int sock{};  
    addrinfo* p;  
    for (p = result; p != nullptr; p = p->ai_next) {  
        // UDP 创建套接字  
        if ((sock = socket(p->ai_family, p->ai_socktype, p->ai_protocol)) == 0) {  
            perror("socket create error");  
            continue;  
        }  
  
        int temp = 1; // 设置套接字选项  
        if (setsockopt(sock, SOL_SOCKET, SO_REUSEADDR, &temp, sizeof(int)) != 0) {  
            perror("setsockopt");  
            exit(1);  
        }  
  
        if (bind(sock, p->ai_addr, p->ai_addrlen) != 0) { // 绑定地址  
            perror("bind");  
            continue;  
        }  
  
        break; // 找到第一个没问题的地址，然后给这个地址创建套接字并绑定该地址与端口（这里是万能匹配地址）  
    }  
    freeaddrinfo(result); // 找完了立刻释放  
  
    if (p == nullptr) {  
        std::cout << "遍历了所有，没有找到合适的" << std::endl;  
        exit(1);  
    }  
  
    // 发信者的地址情况  
    auto* other_addr = new sockaddr_storage(); // delete点之一s  
    auto* other_addr_length = new socklen_t(sizeof(*other_addr));  
  
    char message_cstr[1024]; // 存放收的信息  
    char addr_message[INET6_ADDRSTRLEN]; // 存放可阅读的地址  
    for (int i = 0; i < 5; i++) { // 测5次  
        // 先接收信息，获取发信方的地址情况  
        // 再给发信方发送信息  
        const size_t recv_length = recvfrom(sock, message_cstr, sizeof(message_cstr), 0, (sockaddr*)other_addr, other_addr_length);  
        if (recv_length < 0) {  
            perror("recvfrom");  
            continue;  
        }  
  
        if (other_addr->ss_family == AF_INET) {  
            auto temp = (sockaddr_in*)other_addr;  
            std::cout << "ip4地址" << std::endl;  
            inet_ntop(AF_INET, &temp->sin_addr, addr_message, sizeof(addr_message));  
        }else {  
            auto temp = (sockaddr_in6*)other_addr;  
            std::cout << "ip6地址" << std::endl;  
            inet_ntop(AF_INET6, &temp->sin6_addr, addr_message, sizeof(addr_message));  
        }  
        std::cout << "发信地址：" << addr_message << std::endl; // 打印来信者的信息  
        std::cout << "发信内容：" << std::string(message_cstr, recv_length) << std::endl;  
  
        // 返还信息  
        std::string send_message = "好久不见，sayori";  
        const size_t send_length = sendto(sock, send_message.c_str(), send_message.size(), 0, (sockaddr*)other_addr, *other_addr_length);  
        if (send_length < 0) {  
            perror("senderror");  
            continue;  
        }  
    }  
  
    delete other_addr;  
    delete other_addr_length;  
}
```

### 客户端
```cpp
// 客户端  
#include <iostream>  
#include <string>  
#include <sys/socket.h>  
#include <netinet/in.h>  
#include <arpa/inet.h>  
  
  
  
int main() {  
    int sock = socket(AF_INET, SOCK_DGRAM, 0);  
  
    // 直接指定  
    auto* myaddr = new sockaddr_in;  
    myaddr->sin_family = AF_INET;  
    myaddr->sin_addr.s_addr = inet_addr("127.0.0.1");  
    myaddr->sin_port = htons(8080);  
  
    connect(sock, reinterpret_cast<sockaddr *>(myaddr), sizeof(*myaddr));  
  
  
    const std::string message = "hello";  
  
    send(sock, message.c_str(),message.size(),0);  
  
    char receive_message[1024];  
    const ssize_t receive_length = recv(sock, receive_message, sizeof(receive_message), 0);  
    if (receive_length <= 0) {  
        perror("recevie error");  
    }  
    receive_message[receive_length] = '\0';  
    const std::string actual_message(receive_message, receive_length);  
    std::cout << actual_message << std::endl;  
  
    delete myaddr;  
}

```