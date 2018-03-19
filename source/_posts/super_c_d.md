---
layout:     post                    # 使用的布局（不需要改）
title:      高级C与网络编程复习（4）—— 基本套接字函数（Elementary Sockets Functions）（第四章）         # 标题
subtitle:   基本套接字编程        #副标题
date:       2018-01-04              # 时间
author:     Ming.J                      # 作者
header-img: img/hacker.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - C语言
    - 网络编程
    - 学习
    - 笔记
---

{% img /img/unp_14.png 基本TCP客户/服务器程序的套接字函数 %}
> ## socket函数

  ```c
  #include <sys/socket.h>

  /**
  * 该函数用于创建一个socket套接字
  * @param domin    协议族/地址族
  * @param type     套接字的类型
  *                 SOCK_STREAM ==> TCP套接字
  *                 SOCK_DGRAM  ==> UDP套接字
  *                 SOCK_RAW    ==> 原始套接字
  *                 SOCK_PACKET ==> 可用于链路层访问控制
  * @param protocol 指定协议
  *
  * @return         返回一个socket描述符 sockfd
  *                 sockfd < 0  ==> 创建失败
  *                 sockfd >= 0 ==> 创建成功，之后可用该sockfd进行IO操作
  **/
  int socket(int family, int type, int protocol);
  ```
  - family 常值
    {% img /img/unp_15.png socket函数的family常值 %}
  - type 常值
    {% img /img/unp_16.png socket函数的type常值 %}
  - protocol 常值
    {% img /img/unp_17.png socket函数的AF_INET或AF_INET6的常值 %}
  - socket函数中的family不是任意组合都是有效的，下面是组合效果：
    {% img /img/unp_18.png socket函数中family和type参数的组合 %}

> ## AF_XXX 和 PF_XXX

- **AF_**前缀表示**地址族**，**PF_**前缀表示**协议族**
- 历史上曾有这样的想法：单个协议族可以支持多个地址族，PF_值用来创建套接字，而AF_值用于套接字地址结构。
- 但实际上，支持多个地址族的协议从未出现过，而且**头文件 &lt;sys/socket.h&gt;中为一给定协议定义的PF_值总是与此协议的AF_值相等**

> ## connect函数

  ```c
  /**T
  * 该函数用于建立与指定socket的连接
  * @param sockfd       一个未连接的socket的描述符
  * @param sockaddr     指向要连接的套接字的sockaddr结构体的指针
  * @param addrlen      上述sockaddr结构体的长度
  *
  * @return         成功则返回0, 失败返回-1, 错误原因存于errno 中
  **/
  int connect(int sockfd, const struct sockaddr * servaddr, int addrlen);
  ```
  - 如果是**TCP套接字**，调用**connect**函数将**激发TCP的三路握手过程**。而且**仅在连接建立成功或出错时才返回**
  - ### connect错误：
    - ETIMEOUT（超时错误）: TCP客户**没有**收到对发出的SYN分节的**响应**
    - ECONNREFUSED（连接拒绝错误）：客户在发出SYN分节后收到**RST响应**
      - 表明服务器主机在我们指定的端口上没有进程在等待与之连接（通常是服务器进程没有在运行，或者是客户端连接的时候指定了错误的端口号）
      - 或TCP向取消一个已有的连接
      - 或TCP接收到一个根本不存在的连接上的分节
      - **硬错误**（hard error）
    - EHOSTUNREACH或ENETUNREACH（主机不可达或）
      - 客户咋中间的某个路由器上引发了一个“destination unreachable”（目的地不可达）ICMP错误
      - 并且在某个规定时间（4.4BSD规定75s）内仍未收到响应
      - **软错误**（soft error）
  - ### 状态转换
    - TCP状态转换图
    {% img /img/unp_4.png TCP状态转换图%}
    - connect函数导致当前客户套接字从CLOSED状态（该套接字自从由socket函数创建以来，一直处于CLOSED状态）转移到SYN_SENT状态
    - 如果连接成功则转移到ESTABLISHED状态
    - **若connect失败，则该套接字不可再用**，必须关闭，我们不能对这样的套接字再次调用connect函数
    - 当循环调用函数connect为给定主机尝试各个IP地址直到有一个成功时，在**每次connect失败后，都必须close当前的套接字描述符**并重新调用socket

> ## bind函数
> - **bind函数把一个本地协议地址赋予一个套接字**
> - 对于网际协议，协议地址是32位的IPv4地址或128位的IPv6地址与16位的TCP或UDP端口号的组合
> - 调用bind函数可以指定一个端口号，或指定一个IP地址，也可以两者都指定或两者都不指定

  ```c
  /****
  *  sockfd：   标识一未捆绑套接口的描述字。
  *  my_addr：  赋予套接口的地址。sockaddr结构定义如下：
  *             struct sockaddr{
  *               u_short sa_family;
  *               char sa_data[14];
  *             };
  *  addrlen：  my_addr的长度。
  *  返回值：    成功返回0，失败返回-1.
  ****/
  int bind( int sockfd , const struct sockaddr * my_addr, socklen_t addrlen);
  ```
  - ### 端口的绑定
    - 通常用于**服务器在启动的时候捆绑他们众所周知的端口**
    - 对于客户机，不调用bind绑定端口，而是在发送消息的时候由内核临时分配一个端口，这是正常的
    - 而对于服务器而言，不绑定端口是极为罕见的
  - ### 地址的绑定
    - **进程**可以**把**一个**特定的IP地址绑定到它的套接字**上，不过这个IP必须属于其所在主机的网络接口之一
    - 对于**客户端**而言，**绑定IP地址**就相当于**为该套接字上发送IP数据报指定了源IP地址**
    - 对于**服务器**而言，**绑定IP地址**就相当于**限定该套接字只能接收那些目的地为这个IP地址的客户连接**
  - ### 给bind函数指定要捆绑的IP地址和端口号产生的结果
    {% img /img/unp_19.png 给bind函数指定要捆绑的IP地址和端口号产生的结果 %}
  - ### 通配地址(wildcard address)
    - IPv4: INADDR_ANY
      ```c
      //IPv4
      struct sockaddr_in servaddr;
      servaddr.sin_addr.s_addr = htonl(INADDR_ANY);   //wildcard
      ```
    - IPv6: in6addr_any
      ```c
      //IPv6
      struct sockaddr_in6 serv;
      serv.sin6_addr = in6addr_any;               //wildcard
      ```
  - ### 错误
    - EADDRINUSE("Address already in use", 地址已使用)

> ## listen函数
> **仅由TCP服务器调用**，它做两件事情
> - 当**socket函数创建**一个套接字时，它被假设为**一个主动套接字**(active socket)，也就是说，它是一个将调用connect发起连接的客户套接字。**listen函数把**一个**未连接的套接字转换成一个被动套接字**，之后是内核应接收指向该套接字的连接请求。
> - 本函数的第二个参数规定了内核应该为相应套接字排队的最大连接个数

  ```c
  /**
  * 将一个未连接的套接字转换成监听套接字，这样即可以用来监听来自客户端的请求了
  * @param  sockfd     一个未连接的套接字描述符
  * @param  backlog    等待连接队列的最大长度
  **/
  int listen( int sockfd, int backlog);
  ```
  - ### 调用时机
    - 本函数通常应该在调用socket和bind这两个函数以后，并在调用accept函数之前调用
  - ### 内核为任何一个给定的监听套接字维护两个队列
    {% img /img/unp_20.png TCP为监听套接字维护的两个队列 %}
    - 未完成连接队列（incomplete connection queue）：服务器收到请求的SYN分节，并且正在等待完成相应的TCP三路握手过程。**这些套接字处于SYN_RCVD状态**
    - 已完成连接队列（completed connection queue）：每个已完成TCP三路握手过程的客户对应其中的一项。**这些套接字处于ESTABLISHED状态**
    - [点我可查看TCP状态转换图](../../03/super_c_b#TCP状态转换图)
  - ### listen函数的第二个参数通常指的是已完成连接队列的最大长度
  - ### 两个队列的建立时机
    {% img /img/unp_21.png TCP三路握手和监听套接字的两个队列 %}

> ## accept函数
> - **accpet函数**由TCP服务器调用，用于**从已完成连接队列头返回一个已完成连接**
> - 如果已完成连接队列为空，那么进程将被投入睡眠（假定套接字为默认的阻塞方式）

  ```c
  /**
  * 在一个套接字的监听队列中取一个连接，如果没有，则死等
  *
  * @param sockfd    监听描述符（在调用listen之后监听来自客户端的连接）
  * @param addr      (可选)用来保存新连接的源端地址
  * @param addrlen   (可选)用来保存新连接的源端地址结构的长度
  *
  * @return          如果连接成功，则返回一个已连接的套接字描述符（用于和客户端通信）
  **/
  SOCKET accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
  ```
  - 后两个参数为Value-Result参数 ==> [点我查看Value-Result参数详情](../super_c_c#值-结果参数（Value-Result-Argument）)

> ## fork和exec函数

- ### fork函数
  ```c
  #include <unistd.h>

  /**
  * 调用fork函数创建一个新进程，与当前进程并行执行
  *
  * @return 在子进程中为0，在父进程中为子进程ID，若出错返回-1
  **/
  pid_t fork(void);
  ```
  - **fork函数是Unix中派生新进程的唯一方法**
  - **调用一次，返回两次**。返回值告知当前进程是子进程还是父进程
  - 子进程可通过getppid获取父进程的id
  - 父进程fork之前打开的**所有文件描述符都会copy一份给子进程**（各个[描述符的引用计数](./#描述符引用计数)加1）
  - 两个典型用法：
    - 创建自身副本，每个副本并行执行各自的操作
    - 一个进程想要执行另一个程序，则fork一下，在子进程调用exec执行其它程序
- ### exec函数
  ```c
  #include <unistd.h>

  int execl(const char* pathname, const char *arg0, ... /*(char*)*/);

  int execv(const char* pathname, char* const *argv[])

  ...

  ```
  - 存放在硬盘上的可执行程序文件能够被Unix执行的唯一方法是：由一个现有的进程调用上述6个exec函数中的一个
  - exec把当前进程映像替换成新的程序文件，而且该程序通常从main函数开始执行。进程ID不改变
  - 我们称调用exec的进程为**调用进程**（calling process），称新执行的程序为**新程序**（new program）
  - 这些函数只在出错时才返回到调用跟着，否则，控制将被传递给新程序的起始点，通常就是main函数
  - 6个exec函数的关系
    {% img /img/unp_22.png 6个exec函数的关系 %}

> ## 描述符引用计数

  - Unix系统内核为每个文件描述符（包括socket fd）维护一个**引用计数**,这个引用计数标识当前打开着的引用该文件或套接字的描述符的个数
  - 当某个**文件描述符或套接字描述符关闭的时候**，不是直接关闭文件或套接字，而是**引用计数减1**，当引用计数减到0的时候执行关闭操作
  - 需要注意的是：如果多个进程同时拥有指向同一个文件或套接字的描述符。且其中一个没有关闭（并且不再使用了），则就算其他文件描述符都关闭了，这个文件或套接字也不会关闭。就会造成内存泄露
  - 举个栗子：
    ```c
    pid_t pid;
    int listenfd, connfd;
    listenfd = Socket(...);

    /*fill in sockaddr_in{} with server's well-know port*/

    Bind(listenfd, ...)
    Listen(listenfd, LISTENQ);
    for( ; ; ){
      connfd = Accept(listenfd, ...);
      if( (pid = Fork()) == 0 ){        //子进程
        Close(listenfd);
        doit(connfd);
        Close(connfd);
        exit(0);
      }
      Close(connfd);                    //父进程
    }
    ```
    - 上面的代码中，调用了fork之后，connfd和listenfd在父子进程中都有一份
    - 所以在父进程中，只用处理listenfd，故关掉connfd
    - 在子进程中，只用处理connfd，故关掉listenfd
    - **试想**：如果父进程中没有关闭connfd，则就算子进程执行完毕，connfd关联的套接字的引用计数还是不为0，所以一直不会释放。连接多了之后，每个连接的socket都不释放，慢慢的服务器内存就炸了。

> ## close函数

```c
#include <unistd.h>

/**
* 通常Unix close函数也用来光比套接字，并终止TCP序列
*
* @return    0 ==> 成功
*           -1 ==> 出错
**/
int close(int sockfd);
```
- 通常close函数的默认行为是**把该套接字标记成已关闭**，然后立即返回
- 被标记的套接字**不能再被进程使用**，即不能read/write
- 然后**尝试将缓存或队列中所有的Message发出**
- 接着就是**正常的TCP终止序列**
- close函数会将读和写两个方向的连接都关掉


> ## gesockname 和 getpeername 函数

```c
#include <sys/socket.h>

/**
* 返回与sockfd关联的本地协议地址
**/
int getsockname(int sockfd, struct sockaddr *localaddr, socklen_t *addrlen);

/**
* 返回与sockfd关联的外地协议地址
**/
int getpeername(int sockfd, struct sockaddr *peeraddr, socklen_t *addrlen);
```
- 其中，两个函数的后两个参数均为[Value-Result参数](../super_c_c#值-结果参数（Value-Result-Argument）)
- 可以**用getsockname获取内核为我们分配的地址或端口号**
- **getsockname**还可以用于**获取**某个套接字的**协议族**
- 上面两个函数中的第一个参数**sockfd必须是已连接的套接字描述符**
- 当服务器进程通过accept的某个进程通过调用exec执行程序时，getpeername是唯一可以用来获取对端设备地址信息的函数
