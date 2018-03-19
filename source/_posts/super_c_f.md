---
layout:     post                    # 使用的布局（不需要改）
title:      高级C与网络编程复习（6）—— I/O复用（I/O Multiplexing）（第六章）         # 标题
subtitle:   I/O复用（I/O Multiplexing）        #副标题
date:       2018-01-05 10:00:00           # 时间
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

> ## I/O复用概述
> 进程需要一种预先告知内核的能力，使得内核一旦发现进程指定的一个或多个I/O条件就绪（也就是说输入已准备好，或这描述符已能过承接更多的输出和），他就通知进程。这种能力称为**I/O复用**

- ### 应用场合
  - 当**客户处理多个描述符**（通常是交互式输入和网络套接字）时，必须使用I/O复用
    - When a client is handling multiple descriptors (normally interactive input and a network socket)
  - 一个**客户同时处理多个socket套接字**是可能的，不过比较罕见
    - It is possible, but rare, for a client to handle multiple sockets at the same time
  - 如果一个**TCP服务器既要处理监听套接字，又要处理已连接套接字**
    - If a TCP server handles both a listening socket and its connected sockets
  - 如果一个**服务器既要处理TCP，又要处理UDP**
    - If a server handles both TCP and UDP
  - 如果一个**服务器要处理多个服务或者多个协议**
    - If a server handles multiple services and perhaps multiple protocols (e.g., the inetd daemon)
- I/O复用的引用场景不仅仅局限于网络编程

> ## I/O模型
> - blocking I/O                                 ==> 阻塞式I/O
> - nonblocking I/O                              ==> 非阻塞式I/O
> - I/O multiplexing(select and poll)            ==> I/O复用
> - signal driven I/O (SIGIO)                    ==> 信号驱动式I/O
> - asynchronous I/O (the POSIX aio_functions)   ==> 异步I/O

- ### 阻塞式I/O模型
  {% img /img/unp_24.png 阻塞式I/O模型 %}
- ### 非阻塞式I/O模型 ==> 轮询（polling）
  {% img /img/unp_25.png 非阻塞式I/O模型 %}
- ### I/O复用模型
  {% img /img/unp_26.png I/O复用模型 %}
- ### 信号驱动式I/O模型
  {% img /img/unp_27.png 信号驱动式I/O模型 %}
- ### 异步I/O模型
  {% img /img/unp_28.png 异步I/O模型 %}
- ### 各种I/O模型的比较
  {% img /img/unp_29.png 5种I/O模型的比较 %}
  - 同步I/O操作（synchronous I/O operation）：导致请求进程阻塞，知道I/O操作完成
  - 异步I/O操作（asynchronous I/O operation）：不导致请求进程阻塞
  - 上述5种模型中，前4种是同步的，最后一种是异步的

> ## select函数
> select函数允许进程**指示内核等待多个事件中的任何一个发生**，并只在有**一个或多个事件发生或经历一段指定的事件后才唤醒它**
> - Allows the process to instruct the kernel to wait for any one of multiple events to occur and to wake up the process only when one or more of these events occurs or when a specified amount of time has passed.
> - (readable, writable, expired time)

- ### 原型
  ```c
  #include <sys/select.h>
  #include <sys/time.h>

  /**
  * 调用select函数可告知内核对哪些描述符（就读、写或异常条件）感兴趣以及等待多长时间
  *
  * @param maxfdl      最大文件描述符
  * @param readset     读监听集（当监听集内任意描述符可读，会导致select解除阻塞状态）
  * @param writeset    写监听集（当监听集内任意描述符可写，会导致select解除阻塞状态）
  * @param exceptset   异常监听集（当监听集内任意描述符出现异常，会导致select解除阻塞状态）
  * @param timout      最长等待时长（如果该时间过去了，select会跳出）
  *
  * @return     正常返回就绪描述符的总数
  *             0 ==> 超时返回
  *            -1 ==> 出错
  */
  int select(int maxfdl, fd_set *readset, fd_set *writeset, fd_set *exceptset,
             const struct timeval *timeout);

  struct timeval {
    long tv_sec;        //秒
    long tv_usec;       //微秒
  }

  //fd_set的数据结构，实际上是一long类型的数组，每一个bit都能与一打开的文件句柄
  //（不管是socket句柄，还是其他文件或命名管道或设备句柄）建立联系
  typedef struct{
  /*XPG4.2requiresthismembername.Otherwiseavoidthename
  fromtheglobalnamespace.*/
  #ifdef__USE_XOPEN
  __fd_maskfds_bits[__FD_SETSIZE/__NFDBITS];
  #define__FDS_BITS(set)((set)->fds_bits)
  #else
  __fd_mask__fds_bits[__FD_SETSIZE/__NFDBITS];
  #define__FDS_BITS(set)((set)->__fds_bits)
  #endif
  }fd_set;
  ```
  - #### timeout
    - NULL: 忙等
    - 值为0：立即返回
    - 不为0：至多等待一段时间后返回
  - #### fd_set
    - fd_set通常是一个整数数组，其中每个元素中的每一bit对应一个文件描述符
    - 四个操作宏
      ```c
      //将监听集清0
      void FD_ZERO(fd_set *fdset);

      //将一个描述符对应的bit置1
      void FD_SET(int fd, fd_set *fdset);

      //将一个描述符对应的bit置0
      void FD_CLR(int fd, fd_set *fdset);

      //判断一个描述符对应的bit位是否被置1
      void FD_ISSET(int fd, fd_set *fdset);
      ```
    - 举个栗子：
      ```c
      假设fd_set 只有8位 ==> fd_set fs

      执行: FD_ZERO(&fs)
      结果: 00000000

      执行: FD_SET(1, &fs)
      结果: 01000000

      执行：FD_SET(3, &fs)
      结果: 01010000

      执行: FD_CLR(1, &fs)
      结果: 00010000

      执行: FD_ISSET(3)
      返回: 1

      执行: FD_ISSET(1)
      返回: 0
      ```
   - select函数中的三个监听集，如果对哪个不感兴趣，直接设置为NULL就行了
   - select函数中的三个监听集参数均为[Value-Result参数](../../04/super_c_c#值-结果参数（Value-Result-Argument）)
 - #### maxfdpl
   - **maxfdl**指定待测试后描述符的个数，它的**值是待测试的最大描述符加1**（因为数组的下标是从0开始的）
   - FD_SETSIZE 常值是fd_set中描述符的总数，通常是1024，不过一般用不到那么大。指定maxfdp1可以提高select的效率
- ### 描述符就绪条件
  {% img /img/unp_30.png select返回某个套接字就绪的条件小节 %}

> ## str_cli函数修订版

- ### 各种条件下的处理
  {% img /img/unp_31.png str_cli函数中由select处理的各种条件 %}
  - 如果对端TCP发送数据，那么该套接字可读，并且read返回一个大于0的值
  - 如果对端TCP发送一个FIN，那么该套接字可读，并且read返回0（EOF）
  - 如果对端TCP发送一个RST，那么该套接字可读，并且read返回-1，而errno中含有确切的错误码
- ### 修订版的str_cli函数
  ```c
  /**
   * 改进后的str_cli函数，可以在服务器停止后马上返回
   */
  void str_cli(FILE* fp, int sockfd){
      int     maxfdpl;
      fd_set  rset;
      char    sendline[MAXLINE], recvline[MAXLINE];

      //首先清0
      FD_ZERO(&rset);
      for( ; ; ){
          //在调用select之前，设置监听集，告知select感兴趣的描述符
          FD_SET(fileno(fp), &rset);
          FD_SET(sockfd, &rset);

          //待测试的描述符数应该比我们要监听的最大描述符要大1（因为c语言数组是从0开始计的）
          maxfdpl = max(fileno(fp), sockfd) + 1;

          //一直阻塞，直至上述两个描述符至少其中一个可读（没有指定超时时间，所以会一直等待）
          Select(maxfdpl, &rset, NULL, NULL, NULL);

          if(FD_ISSET(sockfd, &rset)){        //socket 可读
              if(Readline(sockfd, recvline, MAXLINE) == 0)
                  err_quit("str_cli: server terminated prematurely");
              Fputs(recvline, stdout);
          }

          if(FD_ISSET(fileno(fp), &rset)){    //用户从控制台输入了数据
              if(Fgets(sendline, MAXLINE, fp) == NULL)
                  return;
              Writen(sockfd, sendline, strlen(sendline));
          }
      }
  }
  ```
  - [点我查看示例源码地址](https://github.com/SunnyQjm/unpv13e/tree/master/job/chapter6)
  - rset作为一个[Value-Result参数](../../04/super_c_c#值-结果参数（Value-Result-Argument）)使用
    - 在**select调用之前**，将感兴趣的描述符对应的bit**置位**。这样在调用select的时候，**告知Unix内核当哪些描述符准备好时应该通知用户**
    - 在**select调用之后**，Unix内核将哪些描述符准备好了记录**在rset当中**。故在select解除阻塞之后，可以用FD_ISSET**判断是哪些描述符准备好了**。(这样处理即便是多个描述符同时准备好了，也可以处理)

> ## 批量输入和缓存（Batch Input and Buffering）

- ### 交互式输入：
  {% img /img/unp_32.png 停等方式的时间线：交互式输入 %}
- ### 批量输入：
  {% img /img/unp_33.png 填充客户与服务器之间的管道：批量方式 %}
  - 以第五章最初的回射函数为例
  - TCP是全双工通信的
  - 我们假设客户以网络能接受的最快速度发送，而服务器以网络能提供的最快速度应答
  - 在时刻7的时候，网络管道就已经充满了
  - 如果在时刻8客户输^D(EOF)，则会导致客户端调用close函数关闭连接，而此时仍然还有请求在去的路上，也还有应答在回来的路上，这些消息客户端将无法再收到。
  - 实际上**这个时候客户只是不发数据了，但是仍然还需要接收数据**。所以我们**需要一个能够关闭TCP连接中其中一半的连接的方法** ==> shutdown函数

> ## shutdown函数

- ## close函数的两个限制
  - close把描述符的引用计数减1，仅在该计数变为0时才关闭套接字。==> 而**shutdown可以不管引用计数就激发TCP的正常终止序列**
  - clse终止读和写两个方向。==> **shutdown可以有选择的终止读、写或读写均关闭**
- ## 原型
  ```c
  #include <sys.socket.h>

  /**
  * howto = SHUT_RD   ==> 关闭读一半
  *         SHUT_WR   ==> 关闭写一半
  *         SHUT_RDWR ==> 读写均关闭
  * @return 成功返回0，出错返回-1
  */
  int shutdown(int sockfd, int howto);
  ```
- 调用shutdown关闭一半连接的图示：
  {% img /img/unp_34.png 调用shutdown关闭一半TCP连接 %}
  - 在调用shutdown关闭读一半以后，四路挥手的前两步已经完成，但此时客户端仍然可以读
  - 直到服务器发送FIN，完成四路挥手

> ## pselect函数
> 是select新的衍生函数，了解一下用法

```c
#include <sys/select.h>
#include <signal.h>
#include <time.h>


int pselect(int maxfdpl, fd_set *readset, fd_set *writeset, fd_set *exceptionset,
            const struct timesepc *timeout, const sigset_t *sigmask);
```

- 其它参数的功能和select的一致
- 最后一个参数是一个信号的掩码集，表示pselect在阻塞期间，禁止掩码集内的信号被递交
