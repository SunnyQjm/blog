---
layout:     post                    # 使用的布局（不需要改）
title:      高级C与网络编程复习（11）—— 数据链路访问（Datalink Access）（第29章）         # 标题
subtitle:   数据链路访问（Datalink Access）        #副标题
date:       2018-01-06 17:00:00           # 时间
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
> 目前大多数操作系统都为应用程序提供访问数据链路层的功能，这种功能可提供如下能力

- **能够监视由数据链路层接收的分组**，使得诸如tcpdump之类的程序能够在普通计算机系统上运行，而**无需使用专门的硬件设备来监视分组**。如果使网络接口进入**混杂模式**(promiscuous mode)，**甚至可以监听**本地电缆上流通的**所有分组**
  - The ability to watch the packets received by the datalink layer, allowing programs such as tcpdump to be run on normal computer systems (as opposed to dedicated hardware devices to watch packets). (promiscuous mode)
- 能够**作为普通应用进程**而不是内核**的一部分运行某些程序**，例如：RARP
  - The ability to run certain programs as normal applications instead of as part of the kernel. For example, most Unix versions of an RARP server are normal applications that read RARP requests from the datalink (RARP requests are not IP datagrams) and then write the reply back to the datalink
- **实现数据链路层访问的三个常见方法**：
  - BSD的分组过滤器**BPF**
  - SVR4的数据链路提供者接口**DLPI**
  - Linux的**SOCK_PACKET**接口

> ## BPF
> 4.4BSD and many other Berkeley-derived implementations support BPF

{% img /img/unp_45.png 使用BPF截获分组%}

> ## DLPI
> SVR4 provides datalink access through DLPI(Datalink Provider Interface)

{% img /img/unp_46.png 使用DLPI、pfmod和bufmod捕获分组%}

> ## SOCK_PACKET and PF_PACKET
> Two methods under Linux:
> - SOCK_PACKET: original, widely available but less flexible
> - PF_PACKET: newer method

```c
/* newer systems*/
fd = socket(PF_PACKET, SOCK_RAW, htons(ETH_P_ALL));
/* older systems*/
fd = socket(AF_INET, SOCK_PACKET, htons(ETH_P_ALL));
```

> ## 相关库

- libpcap: 分组捕获函数和库
- libnet：分组构造与输出库

> ## 检查UDP校验和字段的栗子
> 见课本 图29-3
