---
layout:     post                    # 使用的布局（不需要改）
title:      高级C与网络编程复习（1）—— Introduction（第一章）         # 标题
subtitle:   概述                       #副标题
date:       2018-01-02              # 时间
author:     Ming.J                      # 作者
header-img: img/hacker.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - C语言
    - 网络编程
    - 学习
    - 笔记
---


> Introduction
 - A Simple Daytime Client
 - A Simple Daytime Server

-----

> ### A Simple Daytime Client
> - {% link Steven源码地址 https://github.com/SunnyQjm/unpv13e/blob/master/unpv13e/intro/daytimetcpcli.c %}
>
> - {% link 笔者加注释源码地址 https://github.com/SunnyQjm/unpv13e/blob/master/job/chapter1/daytimecli.c %}
> - #### 功能：
>   - 实现向服务器发起一个TCP连接，请求时间信息，并将接收到的信息打印在控制台上

  {% codeblock lang:c %}
  #include <unp.h>

  int main(int argc, char** argv){
      /*sockfd 套接字描述符，客户端通过该描述符与服务器进行通信（它指示一个与服务器的连接）*/
      int sockfd, n;
      char recvline[MAXLINE + 1];         /*接收缓存*/

      /*服务器地址，用来保存服务器的地址信息（该结构体专门保存IPV4地址），包括ip和端口*/
      struct sockaddr_in servaddr;

      /*如果输入参数不够，则报错并退出*/
      if(argc != 2)
          err_quit("usage: daytimecli <IPAddress>");

      /*通过socket函数创建一个套接字，创建失败则退出*/
      if( (sockfd = socket(AF_INET, SOCK_STREAM, 0) ) < 0)
          err_sys("socket error");

      /*清0*/
      bzero(&servaddr, sizeof(servaddr));
      servaddr.sin_family = AF_INET;      /*指定协议族为网际协议族*/
      servaddr.sin_port = htons(13);      /*指定端口为13*/

      /*将用户从命令行输入的IP存到servaddr中，如果用户输入的格式不正确*/
      /*则该函数会返回小于0的错误信息，此时退出应用程序*/
      if(inet_pton(AF_INET, argv[1], &servaddr.sin_addr) <= 0)
          err_quit("inet_pton error for %s", argv[1]);

      /*连接到servaddr指向的服务器，连接失败则退出*/
      if(connect(sockfd, (SA *)&servaddr, sizeof(servaddr)) < 0)
          err_sys("connect error");

      /*读取服务器发回的信息*/
      while( (n = read(sockfd, recvline, MAXLINE)) > 0 ){
          recvline[n] = 0;

          /*输出到控制台*/
          if(fputs(recvline, stdout) == EOF)
              err_sys("fputs error");
      }

      if(n < 0)
          err_sys("read error");
      exit(0);
  }
  {% endcodeblock %}

  - #### struct sockaddr_in
    - 组成

      {% codeblock lang:c %}
      /**
      * 下面是三个struct sockaddr_in的主要成员，数据的具体类型不同的系统不同，
      * 可以大概理解成下面这样，具体的定义可以查看源码
      **/
      unsigned short sin_family;    /*源码中并不是直接表示成这样，用了几层宏定义，不过在*/
                                    /*笔者的电脑上，其最原始的定义为 unsigned short*/

      u_16 sin_port;                /*u_16表示无符号16位的数，范围为0~65535*/
      struct in_addr sin_addr;

      /**
      * struct in_addr 的结构如下
      **/
      struct in_addr{
        __be32 s_addr;      /*__be32 通常为 unsigned int(32位)*/
      }
      {% endcodeblock %}

    - 用于存储IPV4的地址信息，包括ip，端口，协议族（IPV4属于 AF_INET）
  - #### socket()
    - 百度百科--> {% link click_me https://baike.baidu.com/item/Socket/281150#4 %}
    - [click me for detail](../../04/super_c_d#socket函数)
    - 原型
      {% codeblock lang:c %}
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
      int socket(int domain, int type, int protocol);
      {% endcodeblock %}
    - 需注意的是，socket函数的后两个参数不能随意组合。比如在type = SOCK_STRAM 的时候 protocol ≠ IPPROTO_UDP。（***当第三个参数为0的时候，会自动选择第二个参数类型对应的默认协议***）
  - #### bzero
    - 原型

      {% codeblock lang:c %}
      /**
      * bzero为一个宏函数，实际上调用的是memset
      **/
      #define bzero(ptr, n)   memset(ptr, 0, n)
      {% endcodeblock %}

    - {% link void *memset(void *s, int ch, size_t n); https://baike.baidu.com/item/memset/4747579?fr=aladdin) ==> 以s所指为起始，**将紧接着的n位置成ch(0/1 %}**

    - **bzero(ptr, n)**  ==>  以ptr所指为起始，**将紧接着的n位置成 0** (可以达到清0的效果)
  - #### htons()
    - 百度百科--> {% link click_me https://baike.baidu.com/item/htons %}
    - 原型
      {% codeblock lang:c %}
      /**
      * 将一个无符号短整型从主机字节序转网络字节序
      **/
      u_short htons(u_short hostshort);
      {% endcodeblock %}
    - **网络字节序统一为 大端（big-endian）序**，而现在大部分主机采用的是小端系统，也有机器采用大端系统。为了统一，**调用这个函数之后均采用大端序**（协议统一）
  - #### inet_pton()
    - 百度百科--> {% link click_me https://baike.baidu.com/item/inet_pton %}
    - 原型：
      {% codeblock lang:c %}
      /**
      * 将“点分十进制” --> “二进制整数”
      *
      * @param af   address family(地址族)
      * @param src  指向一个字符串，这个字符串为一个点分十进制的串，例如："192.168.1.1"
      * @param dst  指向一个数据结构，用来存储转换后的结果
      *             如果af = AF_INET, 即为ipv4地址转换，则函数会将结果放在一个in_addr结构体中
      *             如果af = AF_INET6, 即为ipv6地址转换，则函数会将结果放在一个in_addr6结构体中
      *
      * @return   如果函数出错则返回一个负值，并将errno置为EAFNOSUPPORT。
      *           如果参数af指定的地址族和src格式不对，则返回0
      **/
      int inet_pton(int af, const char *src, void *dst);
      {% endcodeblock %}
    - inet_pton同时支持和IPV4和IPV6
  - #### connect()
    - 百度百科--> {% link click_me https://baike.baidu.com/item/connect%28%29 %}
    - [click me for detail](../../04/super_c_d#connect函数)
    - 原型：
      {% codeblock lang:c %}
      /**
      * 该函数用于建立与指定socket的连接
      * @param sockfd       一个未连接的socket的描述符
      * @param sockaddr     指向要连接的套接字的sockaddr结构体的指针
      * @param addrlen      上述sockaddr结构体的长度
      *
      * @return         成功则返回0, 失败返回-1, 错误原因存于errno 中
      **/
      int connect(int sockfd, const struct sockaddr * servaddr, int addrlen);
      {% endcodeblock %}
    - 为了书写简便，原书作者对上述函数的第二个参数做了一层宏定义：
      {% codeblock lang:c %}
      #define SA struct sockaddr
      {% endcodeblock %}
    - 所以在上面的daytimecli.c 中调用connect的时候，用SA简化了书写，实际上 SA = struct sockaddr
      {% codeblock lang:c %}
      connect(sockfd, (SA *)&servaddr, sizeof(servaddr)
      {% endcodeblock %}
  - #### read()
    - 百度百科--> {% link click_me https://baike.baidu.com/item/read%28%29 %}
    - 原型：
      {% codeblock lang:c %}
      /**
      * 从fd所指向的文件中传送count个字节到buf中
      *
      * @param fd      关联一个文件的描述符（可以是socket fd）
      * @param buf     指向一个数组的指针，用做缓存，存取从fd中读出的数据
      * @param count   读取的大小
      *
      * @return        返回值为实际读取到的字节数
      *                如果返回0，表示已到达文件尾或无可读取的数据。
      *                错误返回-1,并将根据不同的错误原因适当的设置错误码
      **/
      ssize_t read(int fd, void *buf, size_t count);
      {% endcodeblock %}
    - 如果read函数中传入的文件描述符为sockfd，则表示从网络中读取count字节的数据并存到buf中。
    - read函数是一个**阻塞函数**，如果没有读够count个字节，会一直在那边死等，下面两种情况下read函数和的阻塞状态会解除
      - 如果一个信号的到来，会导致主线程因为去执行信号的回调函数，而解除阻塞函数的阻塞状态。并将errno置成{% link EINTR https://baike.baidu.com/item/EINTR %}。表示因为信号中断而退出。（***不过现在的系统好像做了优化处理，即便定义了某些信号的处理函数，当该信号到来时，该回调会执行，但同时却不会引发中断错误***）
      - 还有就是收到EOF（文件结束指针）。如果是tcp socket，**则当对方关闭了写一端的时候，会向本机发送一个FIN，标识对方已经发完数据了。此时read的阻塞状态便会解除**。同样的，如果对方给调用close函数关闭了socket，read函数也会解除阻塞（关闭socket相当于写端和读端都关闭了）
    - 对比 [write](./#write)
  - #### fputs()
    - 百度百科--> {% link click_me https://baike.baidu.com/item/fputs/10942345 %}
    - 原型：
      {% codeblock lang:c %}
      /**
      * 向指定的文件中写入一个字符串
      *
      * @param ptr     指向待写入的字符串
      * @param stream  指向目标文件的一个文件指针（文件指针由fopen获得）
      *
      * @return        函数返回值为一般非负整数，如果返回EOF(常值，为-1)，则标识读到文件尾
      **/
      int fputs(const char* ptr, FILE* stream)
      {% endcodeblock %}
  - #### IPV6版本
    - 上面的程序是支持IPV，{% link 点击此处有IPV6版本的代码 https://github.com/SunnyQjm/unpv13e/blob/master/unpv13e/intro/daytimetcpcliv6.c %}

> ### 一些关于教材的扩展介绍

- #### 包裹函数（wrapper function）
  - linux系统内核的c语言函数名都是小写的，如果之后的代码中出现了**大写开头的函数**，则表示是原书作者**对内核函数做了一层包装**，加了一些错误判断等，使用起来更加方便，这些即作者所说的**包裹函数（wrapper function）**
  - 举个栗子：
    {% codeblock lang:c %}
    int Socket(int family, int type, int protocol){
      int n;
      if( (n = socket(family, type, protocol)) < 0 )
        err_sys("socket error");
      return(n);
    }
    {% endcodeblock %}
    - 上面这个函数便是作者对内核的socket函数做了一层封装，功能和socket函数时候一样的，只不过在出错的时候，这个函数已经帮你将错误打印出来了，如果不需要什么其它特殊处理的话。在使用Socket函数的时候变可以不需要错误处理了
    - 下面展示会了没有使用包裹函数和使用了包裹函数的区别
      {% codeblock lang:c %}
      //不使用包裹函数
      if( (sockfd == socket(AF_INET, SOCK_STREAM, 0) ) < 0)
        err_sys("socket error");

      //使用包裹函数
      Socket(AF_INET, SOCK_STREAM, 0);
      {% endcodeblock %}
  - 原书作者定义的包裹函数大致符合下列规则
    - **名字和被包裹的函数一致，只是首字母大写**
    - **函数的参数数量和意义和被包裹的函数一致**
    - **函数的行为与被包裹的函数保持一致**
- #### Unix errno 值
  - {% link error https://baike.baidu.com/item/errno %} 为一个**全局变量**
  - 当**Unix中的函数**执行过程中**有错误发生**，则**errno**就被**置为一个指明该错误类型的正值**，而函数本身通常返回-1

> ### A Simple Daytime Server
> - {% link Steven源码地址 https://github.com/SunnyQjm/unpv13e/blob/master/unpv13e/intro/daytimetcpsrv.c %}
>
> - {% link 笔者加注释源码地址 https://github.com/SunnyQjm/unpv13e/blob/master/job/chapter1/daytimeserv.c %}
> - #### 功能：
>   - 实现在13号端口上监听来自任意IP的请求，并向客户端输出时间信息
>   - 没做并发处理，一次只能向一个客户端提供服务

{% codeblock lang:c %}
#include <unp.h>
#include <time.h>

int main(int argc, int argv){
    int listenfd, connfd;
    struct sockaddr_in servaddr;
    char buff[MAXLINE];
    time_t ticks;

    //调用包裹函数，创建一个socket
    listenfd = Socket(AF_INET, SOCK_STREAM, 0);

    bzero(&servaddr, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    servaddr.sin_port = htons(13);

    //指定socket的地址为INADDR_ANY，标识监听来自所有地址的请求
    servaddr.sin_addr.s_addr = htonl(INADDR_ANY);

    //将socket绑定到指定的端口，只在指定的端口监听来自客户端的请求
    Bind(listenfd, (SA *) &servaddr, sizeof(servaddr));

    //调用Lisen可将socket转换成一个监听套接字，监听套接字可用于监听其他客户端的请求
    Listen(listenfd, LISTENQ);

    for( ; ; ){
        //accpet函数是一个阻塞函数，死等一个连接请求。
        //当监听到一个请求，就返回一个已连接描述符（该描述符用于与新连接的那个客户端通信）
        connfd = Accept(listenfd, (SA *) NULL, NULL);

        //获取当前系统的时间
        ticks = time(NULL);

        //将当前时间输出到buff数组中
        snprintf(buff, sizeof(buff), "%.24s\r\n", ctime(&ticks));

        //将buff中的数据发给客户端
        Write(connfd, buff, strlen(buff));

        //关闭socket连接
        Close(connfd);
    }
}
{% endcodeblock %}

- #### bind()
  - 百度百科--> {% link click_me https://baike.baidu.com/item/bind%28%29?fromtitle=bind&fromid=17200415 %}
  - [click me for detail](../../04/super_c_d#bind函数)
  - 原型：
    {% codeblock lang:c %}
    /****
    *  sockfd：   标识一未捆绑套接口的描述字。
    *  my_addr：  赋予套接口的地址。sockaddr结构定义如下：
    *             struct sockaddr{
    *               u_short sa_family;
    *               char sa_data[14];
    *             };
    *  addrlen：  name名字的长度。
    *  返回值：    成功返回0，失败返回-1.
    ****/
    int bind( int sockfd , const struct sockaddr * my_addr, socklen_t addrlen);
    {% endcodeblock %}
  - bind函数把一个本地协议地址赋予一个套接字，通常在connect或listen函数调用前使用
- #### listen()
  - 百度百科--> {% link click_me  %}
  - [click me for detail](../../04/super_c_d#listen函数)
  - 原型
    {% codeblock lang:c %}
    /**
    * 将一个未连接的套接字（主动套接字）转换成监听套接字（被动套接字），这样即可以用来监听来自客户端的请求了
    * @param  sockfd     一个未连接的套接字描述符
    * @param  backlog    等待连接队列的最大长度
    **/
    int listen( int sockfd, int backlog);
    {% endcodeblock %}
  - 函数的第二个参数指定的是系统内核允许在这个监听描述符上排队的最大客户连接数（内核为listen维护两个队列，第二个参数指定的至一般认为是已连接队列的上限）
    - **不是允许的最大并发数**
    - **在监听描述符上排队的客户** ==> 客户的请求被listen到了，但是还没有被accept处理，那么这个客户的请求便在该监听描述符上排队，等待被accpet
    - 通常情况下，accpet以后就调用新线程或新进程处理了，所以很快就可以再accept，所以一般在监听描述符上排队的客户数不会很多
- #### accept()
  - 百度百科--> {% link click_me https://baike.baidu.com/item/accept%28%29 %}
  - [click me for detail](../../04/super_c_d#accept函数)
  - 原型：
    {% codeblock lang:c %}
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
    {% endcodeblock %}
  - 上述函数的第二、三个参数为**值-结果（value-result）参数**
    - 即在函数调用的时候，可以通过这两个参数向函数内部传递内容
    - 同时在函数调用结束的时候，可以通过这两个参数获取到返回信息
    - accept函数就将新连接的地址信息保存在了后两个参数中（如果不需要可以直接传NULL）
  - **accpet为每个连接到本服务器的客户返回一个全新的描述符（唯一标识一个客户）**
- #### snprintf()
    - 百度百科--> {% link click_me https://baike.baidu.com/item/snprintf%28%29?fromtitle=snprintf&fromid=7320492 %}
  - 原型：
    {% codeblock lang:c %}
    /**
    * 向str指向的区域格式化输出size个字节的数据
    **/
    int snprintf(char *str, size_t size, const char *format, ...)
    {% endcodeblock %}
  - 用法和printf基本相同
  - 不同的是，**printf是向控制台打印，而snprintf是通过地址指针，向目标区域输出**
- #### write()
    - 百度百科--> {% link click_me https://baike.baidu.com/item/write/6391023 %}
  - 原型：
    {% codeblock lang:c %}
    /**
    * 将buf中count个字节的数据输出到fd标识的文件中
    **/
    ssize_t write (int fd,const void * buf,size_t count);
    {% endcodeblock %}
  - 对比[read](./#read)
- #### close()
  - [click me for detail](../../04/super_c_d#close函数)
  - 关闭与客户端的连接。该调用引发正常的TCP连接终止序列：**每个方向上（读方向，写方向）发送一个FIN，每个FIN又各自的对端确认**
