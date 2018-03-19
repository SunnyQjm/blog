---
layout:     post                    # 使用的布局（不需要改）
title:      高级C与网络编程复习（9）—— 名字和地址转换（Name and Address Conversions）（第11章）         # 标题
subtitle:   名字和地址转换（Name and Address Conversions）        #副标题
date:       2018-01-06 14:00:00           # 时间
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
> 数值地址长而不容易记，手工键入容易出错。用名字代表数字地址的机制便应运而生
> [点我获取本文源码](https://github.com/SunnyQjm/unpv13e/tree/master/job/chapter11)

- 主机名和IPv4地址之间转换：gethostbyname、gethostbyaddr
- 服务名字和端口号之间转换：getservbyname、getservbyport
- 两个协议无关的转换函数：getaddrinfo、getnameinfo


> ## 域名系统（Domain Name System， DNS）
> - 主要用于主机名字和和IP地址之间的映射
> - **基于UDP**

- ### 简单名字（simple name）和全限定域名（Fully Qualified Domain Name， FQDN）
  - 简单名字：如 solaris或bsdi
  - 全限定域名：如 qjm253.top
    - 严格来说FQDN最后必须以.结尾，但通常用户会省略。例如上面个域名严格来讲为： qjm253.top.
- ### 资源记录
  - **A**：A记录把一个**主机名映射成一个32位的IPv4地址**
  - **AAAA**：AAAA记录把一个**主机名映射成一个128位的IPv6地址**
  - **PTR**：称为指针记录（pointer record）的PTR记录把**IP地址映射成主机名**（IPv4、IPv6）
  - **MX**：MX记录**把一个主机指定作为个给定主机的“邮件交换器”（mail exchanger）**
  - **CNAME**：CNAME代表“canonical name”（规范名字），**常用作为常用的服务指派CNAME记录**
- ### 解析器和名字服务器（Resolvers and Name Servers）
  - 名字服务器：存储了域名和IP地址的对应关系
  - 解析器：客户端通过调用解析器中的函数来实现与DNS名字服务器的交互（实现名字与IP地址的转换）
  {% img /img/unp_42.png 客户、解析器和名字服务器的典型关系 %}

> ## gethostbyname函数

- ### 原型：
  ```c
  #include <netdb.h>

  /**
  * 根据名字获取IPv4地址信息
  *
  * @param 待转换的名字，如：qjm253.top
  * @return 成功则返回非空指针，出错则返回NULL，并且设置h_errno
  */
  struct hostent *gethostbyname(const chat *hostname);

  /**
  * host entry 结构体，该结构中包含了所查找主机的所有IPv4地址信息
  */
  struct hostent{
    char   *h_name;             //正式主机名
    char  **h_aliases;          //主机别名
    int     h_addrtype;         //地址类型
    int     h_length;           //地址长度，单位：byte
    char  **h_addr_list;        //主机包含的IPv4地址列表
  }
  ```
  - hostent结构和它所包含的信息
  {% img /img/unp_43.png hostent结构和它所包含的信息 %}
- ### 错误：h_errno
  - 当调用gethostbyname出错时，便会设置h_errno的值为下列常值之一
  - #### 类型
    - HOST_NOT_FOUND
    - TRY_AGAIN
    - NO_RECOVERY
    - NO_DATA（等同于HOST_NOT_FOUND）
  - hstrerrno函数接收一个上述常值之一，返回错误的具体描述
- ### 栗子：
  - 本程序与课本略有不同，简单的支持一个地址解析以及默认解析IPv4
  ```c <br/>&ensp; ghbn.c
  #include <unp.h>
  int main(int argc, char **argv){
      char *ptr, **pptr;
      struct hostent *hptr;
      char str[INET_ADDRSTRLEN];
      int i = 0;

      if(argc != 2)
          err_quit("usage: ./ghbn <Domain name>");

      if( (hptr = gethostbyname(argv[1])) == NULL){
          err_msg("get hostbyname error for host %s : %s", argv[1], hstrerror(h_errno));
      } else {
          printf("official hostname: %s\n", hptr->h_name);
          for(pptr = hptr->h_aliases; *pptr != NULL; pptr++)
              printf("\talist: %s\n", *pptr);
          for(pptr = hptr->h_addr_list; *pptr != NULL; pptr++)
              printf("address: %s\n", Inet_ntop(AF_INET, *pptr, str, sizeof(str)));
      }


  }
  ```
  - 测试：
    ```c
    //我们先来测试一下正常的情况
    //可以很容易看出本博客是搭建在github上的，并且使用了一个作者自己的域名指向github上的域名
    输入：  ./ghbn qjm253.top
    输出：
          official hostname: sni.github.map.fastly.net
          	alist: qjm253.top
          	alist: sunnyqjm.github.io
          address: 151.101.77.147

    //再来测试一下域名不存在的情况
    输入： ./ghbn qjm253.top.c
    输出：
          get hostbyname error for host qjm253.top.c : Unknown host

    //测试一下域名存在，但是没有映射地址的情况
    //qjm253.cn是笔者的一个保留域名，目前还没有解析到任何地址
    输入： ./ghbn qjm253.cn
    输出：
          get hostbyname error for host qjm253.cn : No address associated with name

    ```
> ## gethostbyaddr函数

- ### 原型：
  ```c
  #include <netdb.h>

  /**
  * 本函数试图由一个二进制的IP地址和找到相应的主机名，与gethostbyname的行为相反
  *
  * @param addr   addr参数实际上不是一个char *类型的指针，而是一个指向in_addr结构的指针
  * @param len    len实际上就是in_addr结构体的大小
  * @param family 由于本函数只支持IPv4，所以family为AF_INET
  * @return 成功则返回非空指针，失败则返回NULL，并且设置h_errno
  **/
  struct hostent *gethostbyaddr(const char *addr, socklen_t len, int family);
  ```
- ### 栗子
  ```c <br/>&ensp; ghba.c
  #include <unp.h>

  int main(int argc, char **argv){
      char *ptr, **pptr;
      struct hostent *hptr;
      char str[INET_ADDRSTRLEN];

      struct sockaddr_in ia;

      if(argc != 2)
          err_quit("usage: ./ghbn <IPAddress>");

      Inet_pton(AF_INET, argv[1], &ia.sin_addr);
      if( (hptr = gethostbyaddr(&ia.sin_addr, sizeof(ia.sin_addr), AF_INET)) == NULL){
          err_msg("get hostbyname error for host %s : %s", argv[1], hstrerror(h_errno));
      } else {
          printf("official hostname: %s\n", hptr->h_name);
          for(pptr = hptr->h_aliases; *pptr != NULL; pptr++)
              printf("\talist: %s\n", *pptr);
          for(pptr = hptr->h_addr_list; *pptr != NULL; pptr++)
              printf("address: %s\n", Inet_ntop(AF_INET, *pptr, str, sizeof(str)));
      }


  }
  ```
  - gethostbyaddr的第一个参数虽然是in_addr的地址结构，但是传的时候不能直接用in_addr，要使用sockaddr_in里面的in_addr。（博主猜想是应用的头文件中可能包含了in_addr结构体的多种实现，而gethostbyaddr函数要求的in_addr地址结构必须时候sockaddr_in里面定义的那种，读者有兴趣可以自行验证）
  - 测试：
    ```c
    输入： ./ghba 127.0.0.1
    输出：
          official hostname: localhost
          address: 127.0.0.1
    ```
  - 上面测试的是本机的回环地址，是成功的。如果要测试其它地址则不成功，具体可[点我查看](https://www.cnblogs.com/wunaozai/p/3753731.html)
  
> ## getservbyname和getservbyport

- ### getservbyname
  - #### 原型：
    ```c
    #include <netdb.h>

    /**
    * 通过服务名获取服务信息
    *
    * @param servname   服务名，如：domain, ftp
    * @param protoname  协议名，如：TCP, UDP
    * @return 若成功则返回非空指针，若出错则返回NULL
    **/
    struct servent *getservbyname(const char *servname, const char *protoname);

    struct servent{
      char    *s_name;        //正式的服务名
      char   **s_aliases;     //服务别名列表
      int      s_port;        //网络字节序的端口号
      char    *s_proto;       //协议名
    }
    ```
  - #### 栗子
    ```c <br/>&ensp; gsbn.c
    #include <unp.h>

    int main(int argc, char **argv){
        struct servent *sptr;
        char **sptrs;

        if(argc != 3)
            err_quit("usgae: ./gsbn <servname> <protoname>");
        sptr = getservbyname(argv[1], argv[2]);
        if(sptr == NULL)
            err_quit("get error");
        printf("official name: %s\n", sptr->s_name);
        for(sptrs = sptr->s_aliases; *sptrs != NULL; sptrs++)
            printf("\t\talias: %s\n", *sptrs);
        printf("port: %d\n", sptr->s_port);
        printf("protocol: %s\n", sptr->s_proto);
    }
    ```
  - #### 测试
    ```c
    输入： ./gsbn domain udp
    输出：
          official name: domain
          port: 13568
          protocol: udp

    输入： ./gsbn ftp tcp
    输出：
          official name: ftp
          port: 5376
          protocol: tcp
    ```

- ### getservbyport
  - #### 原型
    ```c
    #include <netdb.h>

    /**
    * 通过端口号获取服务的信息
    * @param port       端口号
    * @param protoname  协议名，如：TCP, UDP
    * @return 若成功则返回非空指针，若出错则返回NULL
    */
    struct servent *getservbyport(int port, const char *protoname);
    ```
  - #### 栗子
    ```c <br/>&ensp; gsbp.c
    #include <unp.h>

    int main(int argc, char **argv){
        struct servent *sptr;
        char **sptrs;

        if(argc != 3)
            err_quit("usgae: ./gsbn <port> <protoname>");
        sptr = getservbyport(atoi(argv[1]), argv[2]);
        if(sptr == NULL)
            err_quit("get error");
        printf("official name: %s\n", sptr->s_name);
        for(sptrs = sptr->s_aliases; *sptrs != NULL; sptrs++)
            printf("alias: %s\n", *sptrs);
        printf("port: %d\n", sptr->s_port);
        printf("protocol: %s\n", sptr->s_proto);
    }
    ```
  - #### 测试
    ```c

    //shell 通常运行在514端口，并且采用的是tcp协议
    输入： ./gsbp 514 tcp
    输出：
          official name: shell
          alias: cmd
          port: 514
          protocol: tcp
    ```
> ## getaddrinfo函数
> 这是一个比以上函数更新的函数，**了解即可**
