---
layout:     post                    # 使用的布局（不需要改）
title:      高级C与网络编程复习（7）—— 套接字选项（Socket Options）（第七章）         # 标题
subtitle:   套接字选项（Socket Options）        #副标题
date:       2018-01-05 14:00:00             # 时间
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

> ## 获取和设置影响套接字的三种方式

- getsockopt和setsockopt函数和
- fcntl函数 (file control)
- ioctl (IO control)

> ## getsockopt 和 setsockopt 函数
> 这两个函数仅用于套接字

```c
#include <sys/socket.h>

/**
* 获取一个打开的套接字的选项
*
* @param sockfd      必须指向一个打开的套接字描述符
* @param level       级别
* @param optname     选项名
* @param optval      指向一个变量的指针，用于接收函数的结果，其长度由最后一个长度限定
* @param optlen      这是一个Value-Result参数，传入时限定optval的最大长度，防止缓存溢出
*                    函数执行结束时，可以通过这个参数知道内核究竟往optval写了多少数据
* @return 成返回0，出错返回-1
*/
int getsockopt(int sockfd, int level, itn optname, void *optval,
               socklen_t *optlen);

/**
* 设置一个打开的套接字的选项
*
* @param sockfd      必须指向一个打开的套接字描述符
* @param level       级别
* @param optname     选项名
* @param optval      指向一个变量的指针，用于向函数传递要设置的值，其长度由最后一个长度限定
* @param optlen      指示了optval的长度
* @return 成返回0，出错返回-1
*/
int setsockopt(int sockfd, int level, int optname, const void *optval,
               socklen_t optlen);
```
- ### 套接字选项汇总 ==> 课本图7-1和7-2
  - 标志 ==> 表示这个选项是一个二元选项。0表示关闭，非0表示开启

> ## IPv4套接字选项
> 下面几个选项的等级（level）均为IPROTO_IP

- ### IP_HDRINCL
  - 可以为一个原始套接字设置该选项，设置以后可以自己构造IP首部（即在往里面写数据的时候是从IP包的首部起始位置开始写）
- ### IP_OPTIONS
  - 该选项允许我们在IPv4中设置IP选项
- ### IP_RECVDSTADDR
  - 该套接字选项导致**所收到的UDP数据报的目的IP地址**由recvmsg函数作为辅助数据返回
- ### IP_RECVIF
  - 该套接字选项导致**所收到的UDP数据包的接收接口索引**有recvmsg函数作为辅助数据返回
- ### IP_TOS
  - 该套接字选项允许我们为TCP、UDP或SCTP套接字**设置IP首部中的服务类型字段**
- ### IP_TTL
  - 我们可以使用本选项设置或获取系统改用在从某个给定套接字发送的单薄分组上的默认TTL值
