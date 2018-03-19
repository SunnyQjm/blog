---
layout:     post                    # 使用的布局（不需要改）
title:      高级C与网络编程复习（3）—— 套接字编程简介（Sockets Introducrion）（第三章）         # 标题
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

> ## 套接字地址结构（Socket Address Structures）
  > - 大多数套接字函数（socket function）都 需要一个指向套接字地址结构（socket address structure）的指针作为参数
  > - 每个协议族（protocol suite）都定义它自己的套接字地址结构
  > - 这些结构的名字均以**sockaddr_开头**，并以对应每个协议族的唯一后缀结尾

- ### IPV4套接字地址结构
  ```c <br/>&ensp;网际（IPV4）套接字地址结构：<b>sockaddr_in</b>&nbsp;-->&nbsp;定义在&lt;netinet/in.h&gt;当中
  struct in_addr{
    in_addr_t     s_addr;             //32-bit ipv4 Address, network byyte ordered
                                      //32为的ipv地址，采用网络字节序
  }；

  struct sockaddr_in{
    uint8_t         sin_len;            //length of structure (16)
    sa_family_t     sin_family;         //AF_INET ==>本地址结构是IPV4的地址结构，他属于网际协议族
    in_port_t       sin_port;           //16-bit TCP or UDP port, 网络字节序

    struct in_addr  sin_addr;           //32-bit ipv4 address, 网络字节序
    char            sin_zero[8];        //unused ==> 保留位，未使用
  }
  ```
  - POSIX规范要求的数据类型
    {% img /img/unp_10.png POSIX规范要求的数据类型 %}
  - IPV4地址和TCP或UDP端口号都采用网络字节序（大端序）存储
  - IPV4地址存在两种访问方法（因为历史原因）
    - serv.sin_addr            (结构体)
    - serv.sin_addr.in_addr_t （通常是一个无符号的32为整数）
  - sin_zero字段未被使用，不过在填写这种结构的时候sin_zero通常被置为0。（**通常的做法是，在填写之前，用bzero将整个结构体清0，再填写，可以保证未填写的部分都是0**）
  - 套接字地址结构仅在主机上使用，虽然结构体中的某些字段（例如IP地址和端口号）用在不同主机之间的通信，但是**结构体本身并不在主机之间传递**
- ### 通用套接字地址结构
  ```c <br/>&ensp;通用套接字地址结构：<b>sockaddr</b>&nbsp;-->&nbsp;定义在&lt;sys/socket.h&gt;当中
  struct sockaddr{
    uint8_t       sa_len;
    sa_family_t   sa_family;      //Address family: AF_XXX value
    char          sa_data[14];    //protocol-specific address
  }
  ```
  - 前面提到过，大多数套接字函数都需要一个指向套接字地址结构的指针。但是套接字函数大多支持多个协议族，也就是说在调用的时候可能传入不同协议族的地址结构指针，那**套接字函数在定义的时候就必须要有一个类型，可以接收各个协议族对应的地址结构指针**。
  - 这种需求可以用**void \***来解决，实际上也更方便，如果用void \*来定义，**可以接收任意类型的指针**，而且**不用显示转换**。但是void \*实在ANSI C中提出的，而套接字函数是在ANSI C之前定义的，所以为了解决上述需求，采用了通用套接字，下面是一个套接字函数的栗子：
    ```c
    int bind(int, struct sockaddr *, socklen_t);

    //调用方式如下
    bind(sockfd, (struct sockaddr *) &serv, sizeof(serv));
    ```
    - 通用套接字地址结构**sockaddr**和其他协议族各自的地址结构**sockaddr_XX**规定的**最小**size是一样的，都是**16个字节**
    - 套接字函数在具体处理的时候**根据sa_family字段区分不同的地址结构**，对应不同的处理
- ### IPV6套接字地址结构
  ```c <br/>&ensp;IPv6套接字地址结构：<b>sockaddr_in6</b>&nbsp;-->&nbsp;定义在&lt;netinet/in.h&gt;当中
  struct in6_addr {
    uint8_t             s6_addr[16];         //128-bit IPV6 address
  };

  #define SIN6_LEN                  //require for compile-time tests

  struct sockaddr_in6 {
    uint8_t             sin6_len;         //length of this struct, 大小为28个字节
    sa_family_t         sin6_family;      //AF_INET6
    in_port_t           sin6_port;        //传输层端口，网络字节序

    uint32_t            sin6_flowinfo;    //flow information, undefined
    struct in6_addr     sin6_addr;        //IPV6 address, 网络字节序

    uint32_t            sin6_scope_id;    //set of interfaces for a scope
  }
  ```
> ## 值-结果参数（Value-Result Argument）
> - 一个参数，当函数调用时，其作为一个值从函数外传入函数内，当函数返回时，该参数又存储了函数执行的部分结果，这种类型的参数称为**value-result参数**
> - **value-result参数总是以引用/指针的方式传递**（只能用地址传递的方式，如果用值传递方式获取不到函数的返回信息）

- 上文曾提到过，当**往一个套接字函数传递地址结构的时候**，该结构总是以引用的方式传递（即传递地址结构的指针）。**该结构的长度也作为一个参数来传递**，不过其**传递的方式可能是传值，也可能是传指针**，具体的传递方式**取决于该结构的传递方向**：是从进程到内核，还是从内核到进程
- ### 从进程到内核传递套接字地址结构
  - 涉及的函数有：**bind、connect、sendto**
  - 传递结构长度的时候传值就好了
  - 举个栗子：
    ```c
    struct sockaddr_in serv;

    /*fill in serv{}*/

    connect(sockfd, (SA *) &serv, sizeof(serv));
    ```
  - 图示：
  {% img /img/unp_11.png 从进程到内核传递套接字地址结构 %}
- ### 从内核到进程传递套接字地址结构
  - 涉及的函数有：**accpet、recvfrom、getsockname、getpeername**
  - 传递结构长度的时候传入一个指向socklen_t的指针（而不是int，POSIX规范建议将socklen_t定义为uint32_t）
  - 举个栗子：
    ```c
    struct sockaddr_un   cli;
    socklen_t len;

    len = sizeof(cli);
    getpeername(unixfd, (SA *) &cli, &len);
    ```
  - 图示：
  {% img /img/unp_12.png 从内核到进程传递套接字地址结构 %}
  - 当**函数被调用时**，结构大小是一个值（value），它**告诉内核该结构的大小，这样内核在写该结构的时候不至于越界**；当**函数返回时**，结构大小又是一个结果（result），它**告诉进程内核在该结构中究竟存储了多少信息**。

> ## 字节排序函数（Byte Ordering）

- ### 大端和小端（big-endian and little-endian）
  {% img /img/unp_13.png 16位整数的小端字节序和大端字节序 %}
  - 小端：低字节存储在起始地址
  - 大端：高字节存储在起始地址
  - 举个例子：
    ```c
    0x0102

    //从左到右为内存增大方向

    //在大端系统中存储
    00000001 00000010
    //在小端系统中存储
    00000010 00000001
    ```
- ### 测试主机是大端还是小端的实践 --> [click_me](https://github.com/SunnyQjm/unpv13e/blob/master/job/chapter3/byteorder.c)
  ```shell
  ./byteorder

  x86_64-unknown-linux-gnu: little-endian
  ```
- ### 网络字节序
  - 不同的机器可能采用不同的存储方式（大端/小端），为了统一，便为网际协议约定了一个**网络字节序**
  - 网际协议使用**大端字节序**来传送多字节整数
  - 由于不同主机的差异性，便需要有一些函数来进行网络字节序和主机字节序的转换
- ### 字节转换函数
  ```c
  #include <netinet/in.h>

  /**
  * host to network short
  * 将主机字节序的16位短整型转换为网络字节序的16位短整型
  **/
  uint16_t htons(uint16_t host16bitvalue);

  /**
  * host to network long
  * 将主机字节序的32位整型转换为网络字节序的32位整型
  **/
  uint32_t htonl(uint32_t host32bitvalue);

  /**
  * network to host short
  * 将网络字节序的16位短整型转换为主机字节序的16位短整型
  **/
  uint16_t ntohs(uint16_t net16bitvalue);

  /**
  * network to host long
  * 将网络字节序的32位整型转换为主机字节序的32位整型
  **/
  uint32_t ntohl(uint32_t net32bitvalue);
  ```
  - 事实上，在64为的系统中，尽管长整数占64位，htonl和ntohl函数操作的仍然是32位值

> ## 字节操纵函数
> 字节操作函数和有两组，本书中只用到了bzero

- ### 源自Berkeley的函数（b开头的字节操纵函数）
  ```c
  #include <strings.h>

  /**
  * 将以dest为起始的目标串的前nbytes个字节置为0
  **/
  void bzero(void *dest, size_t nbytes);

  /**
  * 将dst中的前nbytes个字节拷贝到src串中
  **/
  void bcopy(const void *src, void *dst, size_t nbytes);

  /**
  * 比较ptr1和ptr2串的前n个字节
  * @return   若相等则返回0， 否则返回非0
  **/
  void bcmp(const void *ptr1, const void *ptr2, size_t nbytes);
  ```
- ### ANSI C函数（mem开头的字节操纵函数）
  ```c
  #include <string.h>

  /**
  * 把dest串的前len个字节置为c
  **/
  void *memset(void *dest, int c, size_t len);

  /**
  * memcpy类似bcopy，但是两个指针的位置时候是相反的
  *
  * PS：当dest串和src串重叠时，bcopy可正常处理，memcpy的处理结果不可知，此时改用memmove函数
  **/
  void *memcpy(void *dest, const void *src, size_t nbytes);

  /**
  * 比较两个串的前nbytes个字节
  * @rerurn    0    相等
  *            >0   第一个不相等字节，ptr1 > ptr2
  *            <0   第一个不相等字节，ptr1 < ptr2
  **/
  void *memcmp(const void *ptr1, const void *ptr2, size_t nbytes);
  ```
- 之前的文章提到过有一个叫[bzero宏](../../02/super_c_a#bzero)的东西。其实是因为笔者的系统不是源自Berkeley的，所有没有bzero函数。但是Steven考虑的比较全面，为没有bzero函数的系统定义了一个bzero宏，间接调用memset函数，但是同样可以实现bzero的效果。

> ## 地址转换函数
> - 地址转换函数在ASCII字符串和网络字节序的二进制值之间转换网际地址
> - **inet_aton、inet_addr、inet_ntoa**在点分十进制数串（例如“192.168.1.1”）与它长度为32位的网络字节序二进制值之间转换IPv4地址。
> - 两个比较新的函数，**inet_pton**和**inet_ntop**对于IPv4和IPv6都适用

- ### inet_aton、~~inet_addr~~ 和inet_ntoa函数
  ```c
  #include<arpa/inet.h>

  /**
  * 将strptr所指c字符串转换成一个32位的网络字节序二进制值，并通过addrptr指针来存储
  **/
  int inet_aton(const char *strptr, struct in_addr *addrptr);

  /**
  * 将strptr所指c字符串转换成一个32位的网络字节序二进制值, 并返回
  **/
  in_addr_t inet_addr(const char *strptr);

  /**
  * 将一个32位的网络字节序二进制IPv4地址转换成相应的点分十进制数串。
  **/
  char *inet_ntoa(struct in_addr inaddr);
  ```
  - **inet_addr 被废弃**
    - inet_addr函数出错时返回INADDR_NOE(通常是一个32位均为1的值)
    - 这意味着该函数不能处理“255.255.255.255”，因为它的二进制值和INADDR_NONE的值是一样的，被用来指示函数执行失败
  - **inet_ntoa函数**返回值所指向的串驻留在静态内存当中，着意味着该函数**是不可重入的**
  - 上面这些函数至支持IPv4，如果要支持IPv6，使用下面介绍的两个函数
- ### inet_pton 和 inet_ntop 函数
  ```c
  #include <arpa/inet.h>

  /**
  * 尝试转换由strptr所指的字符串，并通过addrptr指针存放二进制结果
  *
  * @return   1 ==> 转换成功
  *           0 ==> 输入的不是有效表达式
  *          -1 ==> 转换出错
  **/
  int inet_pton(int family, const char*strptr, void *addrptr);

  /**
  * 与inet_pton进行相反的转换，从数值格式（addrptr）转换到表达式格式（strptr）。
  * @param len     指定目标存储单元的大小，以免该函数溢出调用者的缓冲区
  * @param strptr  用来存储目标串，如果执行成功，返回值即为这个指针
  *               （不能传递空指针，调用这必须为目标存储单元分配内存，并指定其大小）
  * @return  NULL   ==> 失败
  *          strptr ==> 成功
  **/
  const char *inet_ntop(int family, void *addrptr, char *strptr, size_t len);
  ```
  - 两个函数的**family参数可以是AF_INET或AF_INET6**。如果以不被支持的地址族作为family参数。这两个函数都返回一个错误，并将errno置为EAFNOSUPPORT

> ## sock_ntop和相关函数

- 在使用inet_ntop的时候，对于IPv4和IPv6的调用方法不同（这**使得我们的代码和协议相关了**）：
  ```c
  //IPv4
  struct sockaddr_in addr;
  inet_ntop(AF_INET, &addr.sin_addr, str, sizeof(str));

  //IPv6
  struct sockaddr_in6 addr6;
  inet_ntop(AF_INET6, &addr6.sin6_addr, str, sizeof(str));
  ```
- 所以Steven针对上述问题写了下面这么个封装函数
  ```c
  #include "unp.h"

  /**
  * 同时支持IPv4和IPv6版本的inet_ntop
  *
  * @return 成功  ==> 非空指针
  *         出错  ==> NULL
  **/
  char *sock_ntop(const struct *sockaddr, socklen_t addrlen);
  ```
- sock_ntop的实现和其它相关函数此处不做详细讨论，有兴趣可以查看Steven的《Unix 网络编程 卷1》

> ## readn, writen 和 readline 函数

```c
#include "unp.h"

/**
* 从一个描述符中读n字节
**/
ssize_t readn(int filedes, void *buff, size_t nbytes);

/**
* 往一个描述符里写n个字节
**/
ssize_t writen(int filedes, const void *buff, size_t nbytes);

/**
* 从一个描述符中读文本行，以字节为单位
**/
ssize_t readline(int filedes, void *buff, size_t maxlen);
```
- 上面三个函数对read和write操作可能发生的EINTR错误做了处理，是比较安全的读写函数
