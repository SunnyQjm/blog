---
layout:     post                    # 使用的布局（不需要改）
title:      高级C与网络编程复习（8）—— UDP套接字编程（UDP Sockets Introduction）（第八章）         # 标题
subtitle:   UDP套接字编程（UDP Sockets Introduction）        #副标题
date:       2018-01-06 11:40:00           # 时间
time:
author:     Ming.J                      # 作者
header-img: img/hacker.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - C语言
    - 网络编程
    - 学习

    - 笔记
---

> ## 概述
> UDP编程和TCP编程有着本质的差异：UDP是无连接不可靠的数据报协议，非常不同于TCP提供的面向连接的可靠字节流
> [点我获取本文源码](https://github.com/SunnyQjm/unpv13e/tree/master/job/chapter8)

- ### UDP的适用场景
  - DNS（域名系统）
  - NFS（网络文件系统）
  - SNMP（简单网络管理协议）
- ### 典型的UDP C/S 程序的函数调用
  {% img /img/unp_35.png UDPC/S程序所用的套接字函数 %}

> ## recvfrom和sendto函数

```c
#include <sys/socket.h>

/**
* 类似read函数，通过sockfd指向的套接字读数据
*
* @param sockfd    目标描述符
* @param buff      接收缓存，用于存放收到的数据
* @param nbytes    指定接收的字节数
* @param flags     标志位，14章详细讨论，此处都设为0
* @param from      用于存放收到消息的源端地址
* @param addrlen   Value-Result参数，限定from大小，存放from实际大小
*
* @return 成功  ==>  返回收到的字节数
*         出错  ==>  返回-1
*/
ssize_t recvfrom(int sockfd, void *buff, size_t nbytes, int flags,
                 struct sockaddr *from, socklen_t *addrlen);

/**
* 类似write函数，通过sockfd指向的套接字写数据
*
* @param sockfd    目标描述符
* @param buff      发送缓存，用于存放要发送的数据
* @param nbytes    指定发送的字节数
* @param flags     标志位，14章详细讨论，此处都设为0
* @param to        要发送的目的端地址
* @param addrlen   to结构的大小
*
* @return 成功  ==>  返回发送的字节数
*         出错  ==>  返回-1
*/
ssize_t sendto(int sockfd, const void *buff, size_t nbytes, int flags,
               const struct sockaddr *to, socklen_t addrlen);
```
- recvfrom返回0是可以接受的（不像read函数返回0代表对端关闭），因为UDP是无连接的
- **recvfrom**调用的时候如果**不关心发送端的地址**，则**后两个参数from和addrlen可以置为NULL**。要注意的是，**如果from传的是NULL，addrlen也必须传NULL**（可能是由于recvfrom函数的内部实现导致的）
- recvfrom和sendto都可用于TCP，但通常不这么用

> ## UDP实现简单的回射C/S程序

{% img /img/unp_36.png 使用UDP的简单回射客户/服务器 %}

- ### 服务器程序
  ```c
  #include <unp.h>

  void dg_echo(int sockfd, SA *cliaddr, socklen_t clilen);

  int main(int argc, char **argv){
      int sockfd;
      struct sockaddr_in servaddr, cliaddr;

      //第二个参数指定为SOCK_DGRAM，标识创建一个UDP套接字
      sockfd = Socket(AF_INET, SOCK_DGRAM, 0);

      bzero(&servaddr, sizeof(servaddr));
      servaddr.sin_family = AF_INET;
      servaddr.sin_port = htons(9748);
      servaddr.sin_addr.s_addr = htonl(INADDR_ANY);

      //为套接字绑定众所周知的端口号，以及指定接收哪些网络接口的请求
      Bind(sockfd, (SA *) &servaddr, sizeof(servaddr));

      dg_echo(sockfd, (SA *) &cliaddr, sizeof(cliaddr));
  }

  void dg_echo(int sockfd, SA *cliaddr, socklen_t clilen){
      int n;
      socklen_t len;
      char mesg[MAXLINE];

      for( ; ; ){
          len = clilen;
          //接收客户号段发送的消息
          n = Recvfrom(sockfd, mesg, MAXLINE, 0, cliaddr, &len);

          //回射
          Sendto(sockfd, mesg, n, 0, cliaddr, len);
      }
  }
  ```
  - 这段程序和[TCP服务端回射程序](../../05/super_c_e/#TCP回射服务程序)的实现逻辑类似，不一样的是收发消息用的是Recvfrom和Sendto
  - **需要注意的是**：UDP是无连接的，所以如果要给客户回发消息，就必须记录下客户的地址信息，**Recvfrom的后两个参数就是用来记录客户的地址信息的**，回发消息给客户的时候，用这个记录的地址信息即可
  - UDP层中隐含排队，所有收到的包都在队列中排队，recvfrom每次从队头取出消息处理，没有消息则阻塞（消息队列长度是有限制的，可通过SO_RCVBUF套接字选项修改）
- ### 客户端程序
  ```c
  #include <unp.h>

  void dg_cli_my(FILE *fp, int sockfd, SA *servaddr, socklen_t len);

  int main(int argc, char **argv){
      int sockfd;
      struct sockaddr_in servaddr;

      if(argc != 2)
          err_quit("usage: ./udpcli01 <IPAddress>\n");

      //创建一个UDP套接字
      sockfd = Socket(AF_INET, SOCK_DGRAM, 0);

      bzero(&servaddr, sizeof(servaddr));

      servaddr.sin_family = AF_INET;
      servaddr.sin_port = htons(9748);

      Inet_pton(AF_INET, argv[1], &servaddr.sin_addr);

      dg_cli_my(stdin, sockfd, (SA *) &servaddr, sizeof(servaddr));
  }

  void dg_cli_my(FILE* fp, int sockfd, SA *servaddr, socklen_t len){
      int n;
      char sendline[MAXLINE], recvline[MAXLINE];

      //从控制台接收用户输入，并发送给服务器
      //接着接收服务器回射的消息，并输出到控制台
      while( (Fgets(sendline, MAXLINE, fp)) != NULL){
          Sendto(sockfd, sendline, strlen(sendline), 0, servaddr, len);
          n = Recvfrom(sockfd, recvline, MAXLINE, 0, NULL, NULL);
          recvline[n] = 0;
          Fputs(recvline, stdout);
      }

  }
  ```
  - 本段程序和[TCP回射客户端程序](../../05/super_c_e/#TCP回射客户端程序)的实现逻辑类似，同样的区别就在于对网络数据的读写操作不同
  - **需注意的是**：UDP是无连接的，所以客户端在调用Sendto的时候并不要求一定要有一个服务器在监听。即**客户端只是负责把数据发出去，不管是否有人收到**

> ## TCP和UDP回射程序对比

- ### 两个客户的TCP客户/服务器小节
  {% img /img/unp_37.png 两个客户的TCP客户/服务器小节 %}
  - 服务器每收到一个客户端的连接，便派生一个子进程，并将连接交付给子进程维护，父进程接着回去监听客户端的请求。在子进程中处理特定客户端的回射服务
- ### 两个客户的UDP客户/服务器小节
  {% img /img/unp_38.png 两个客户的UDP客户/服务器小节 %}
  - UDP服务器将收到的多个数据报置于套接字接收缓冲区中，再逐个取出，并根据Datagram报文中的发送端地址分别回复
- ### 对比
  - **UDP回射服务程序是永不终止的**，不像TCP中类似EOF的东西。而**TCP回射服务程序**，可以通过客户关闭连接导致服务端子进程中**读到EOF**，进而**子进程退出**。
  - **大多数TCP服务器是并发**的，而**大多数UDP服务器是迭代的**（TCP需要用连接标识不同的客户，并与之通信，所以一般做成并发。而UDP不需要维持连接，对收到的任何一个包进行处理都可以知道是谁发出的，对应处理即可，所以一个进程里面可以处理所有的请求）
  - **TCP通信必须要一个长期在线的服务端，并且服务端要先于客户端启动，而UDP通信则不用**
    - 可以看出，利用**UDP进行通信**的时候，并**不要求服务器在客户端之前启动**（即即便客户端比服务器先启动也是不会报错的，指示发出去的消息没有响应而已），只是服务器在启动之前，客户端无法获取到服务，一旦服务器启动，客户端变可以获取到服务。
    - 相比之下，**TCP进行进行通信**的的时候，**必须服务端先于客户端启动**。因为TCP是有连接的，其通信依赖于一个已经建立好的连接，一旦连接无法建立，变无法进行通信（虽然可以在连接失败时反复尝试，但服务器没启动之前，客户端的connect函数都是返回错误的）

> ## UDP程序栗子小节

- ### 从客户角度总结UDP客户/服务器程序
  {% img /img/unp_39.png 从客户角度总结UDP客户/服务器程序 %}
- ### 从服务器角度总结UDP客户/服务器程序
  {% img /img/unp_40.png 从服务器角度总结UDP客户/服务器程序 %}
- ### 服务器可从到达的IP数据报中获取的信息
  {% img /img/unp_41.png 服务器可从到达的IP数据报中获取的信息 %}
