---
layout:     post                    # 使用的布局（不需要改）
title:      高级C与网络编程复习（10）—— 原始套接字（Raw Socket）（第28章）         # 标题
subtitle:   原始套接字（Raw Socket）        #副标题
date:       2018-01-06 16:00:00           # 时间
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
> 原始套接字提供TCP和UDP所不提供的以下3个能力

- 有了原始套接字，进程可以**读与写ICMPv4、IGMPv4和ICMPv6分组**
  - Raw sockets let us read and write ICMPv4, IGMPv4, and ICMPv6 packets. (ping, mrouted)
- 有了原始套接字，进程**可以读写内核不处理其协议字段的IPv4数据报**
  - With a raw socket, a process can read and write IPv4 datagrams with an IPV4 protocol field that is not processed by the kernel.
- 有了原始套接字，进程还可以**使用IP_HDRINCL套接字选项**（IP header include），**自行构造IPv4首部**
  - With a raw socket, a process can build its own IPv4 header using the IP_HDRINCL socket option

> ## 原始套接字的创建

- ### 创建
  - 调用**socket函数创建**，而且函数的**第二个参数必须是SOCK_RAW**
    ```c
    //下面以创建一个IPv4原始套接字为例
    int sockfd;
    sockfd = socket(AF_INET, SOCK_RAW, protocol);
    ```
  - **protocol参数**通常不为0，是**形如IPPROTO_xxx**的某个常值
  - 只有**超级用户才能创建原始套接字**（防止普通用户自行构造IP数据报）
- **可**按以下方式**开启IP_HDRINCL选项**
  ```c
  const int on = 1;
  if(setsockopt(sockfd, IPPROTO_IP, IP_HDRINCL, &on, sizeof(on)) < 0)
    出错处理
  ```
- 可在原始套接字上调用bind，但比较少见
- 可在原始套接字上调用connect，但比较少见

> ## 原始套接字输出
> 原始套接字的输出遵循以下规则：

- 普通输出通过**调用sendto或sendmsg并指定IP地址完成**，如果套接字已连接，也可以调用write，writev，send
  - Normal output is performed by calling sendto or sendmsg and specifying the destination IP address
- 如果**IP_HDRINCL选项未开启**，进程发送数据的**起始地址是IP首部之后的第一个字节**，**IP首部由内核自行构造**，并把**socket中第三个参数**的值**设置进IP首部的协议字段**
  - If the IP_HDRINCL option is not set, the starting address of the data for the kernel to send specifies the first byte following the IP header because the kernel will build the IP header and prepend it to the data from the process
- 如果**IP_HDRINCL选项开启**，进程发送数据的**起始地址是IP首部的第一个字节**，进程调用输出函数输出的数据中必须包括IP首部，**整个IP首部自行构造**
  - If the IP_HDRINCL option is set, the starting address of the data for the kernel to send specifies the first byte of the IP header
  - IPv4标识字段可置为0，告诉内核设置该值
  - IPv4首部校验和字段总是由内核计算并存储
  - IPv4选项字段是可选的
- **内核**会对**超出外出接口MTU的原始分组**进行**分片**
  - The kernel fragments raw packets that exceed the outgoing interface MTU

> ## 原始套接字输入
> 内核把哪些接收到的IP数据报传递到原始套接字？遵循一下规则

- ### 规则
  - 接收到的**UDP和TCP分组绝不传递到任何原始套接字**(如果一个进程想要读取，必须在数据链路层读取)
    - Received UDP packets and received TCP packets are never passed to a raw socket. (must be read at the datalink layer)
  - **大多数ICMP分组**在**内核处理完**其中的ICMP消息后传递到原始套接字
    - Most ICMP packets are passed to a raw socket after the kernel has finished processing the ICMP message
  - **所有的IGMP分组**在**内核完处理完**其中的IGMP消息后传递到原始套接字
    - All IGMP packets are passed to a raw socket after the kernel has finished processing the IGMP message
  - **内核无法识别其协议字段的所有IP数据报**传递到原始套接字
    - All IP datagrams with a protocol field that the kernel does not understand are passed to a raw socket
  - 如果某个**数据报**是以**片段**的形式到达，那么在它的所有片段均到达且**重组之前**，**不传递任何片段到原始套接字**
    - If t`he datagram arrives in fragments, nothing is passed to a raw socket until all fragments have arrived and have been reassembled.
- ### 内核在分配一个IP数据报到raw socket时的处理
  > - When the kernel has an IP datagram to pass to the raw sockets, all raw sockets for all processes are examined, looking for all matching sockets. A copy of the IP datagram is delivered to each matching socket. The following tests are performed for each raw socket and only if all three tests are true is the datagram delivered to the socket
  > - 当内核分发IP数据报给原始套接字的时候，所有进程的所有原始套接字都将检查一遍，只要匹配，便获得该IP数据报的一个备份，下面是**三个判断条件**，只有当三个判断条件都为真时，内核才把接收的套接字传递给这个套接字

  - 如果该原始套接字调用socket创建的时候，第三个参数指定了protocol（即不为0），则IP数据报中的协议字段，必须和其匹配   ==> **协议匹配**
  - 如果该原始套接字已经调用了bind绑定到本地的某个IP地址，那么IP数据报中的地址字段，必须和其匹配                    ==> **目的地址匹配**
  - 如果该原始套接字已经调用connec连接了，则IP数据报的源端地址必须是connect所连接的对端设备的地址                  ==> **源端地址匹配**

> ## ping Program
> 原理：往目的IP发送一个ICMP回射请求，该节点则以一个ICMP回射应答响应。由于ICMP规则要求回射应答中返回来自回射请求的标识符，序列号，以及可选数据。可以在可选数据中存放时间戳，以便在收到应答的时候就算RTT。（一般系统发送ICMP都有默认的TTL值大小（例如linux/unix 默认是64或255），提取出应答包中的TTL对当前主机和目的主机之间的跳数有一定指导意义）

- ### 一个没有特性蔓延（creeping featurism）的ping程序
  - 自行阅读书上ping程序的源码
- ### ping程序的概貌
  {% img /img/unp_44.png 我们的ping程序汇总各个函数的概貌%}

> ## traceroute Program
> - 原理：一开始向目的节点发送一个TTL为1的

> - traceroute 可以用来测量本机到一个目的主机的路径

- 实现的原理是利用了IPv4的TTL字段或IPv6的跳限字段，一开始设置为1，然后中间节点会返回一个ICMP"time exceeded in transmit"（传输中超时）错误，接着逐渐增大TTL，从而逐步确定下一跳路由地址。直至目的节点返回一个ICMP"port unreachable"（端口不可达）错误，则表示到达目的节点（这要求目的节点没有在该端口上开启服务，即发送ICMP包时，目的端口号应该选择一个未被目的主机使用的端口号 ==> traceroute选择了一个大于30000值作为目的端口号，因为UDP协议要求端口号必须小于30000，所以目的主机如果接收到必然会会一个ICMP端口不可达错误）。
- 早期用IP_HDRINCL来达到修改TTL的目的，现在多用IP_TTL套接字选项来修改
