---
layout:     post                    # 使用的布局（不需要改）
title:      高级C与网络编程复习（12）—— 路由套接字和秘钥管理套接字（Routing Sockets and Key Managment Sockets）（第18章，第19章）         # 标题
subtitle:   路由套接字和秘钥管理套接字（Routing Sockets and Key Managment Sockets）        #副标题
date:       2018-01-06 18:00:00           # 时间
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
> AF_ROUTE domain: **唯一支持的套接字是原始套接字**(raw socket)

- ### 一个路由套接字上支持以下的三种操作
  1. 通过往路由套接字写数据来**向内核发送消息**（例如: 添加或删除路由）
  2. 通过从路由套接字读数据来**接收内核发送的消息**（例如：内核在接收到一个ICMP重定向消息的时候，就通过这种方式通知进程）
  3. 进程可以使用**sysctl函数倾泻出路由表或列出所有已配置的接口**
- 前两种需要超级权限，最后一种操作任何进程都可以执行

- 前两种可以复合起来使用，比如：进程通过写一个路由套接字往内核发送一个消息，请求内核提供关于某个给定路径的所有信息，又通过这个这个路由套接字接收内核的应答

> ## 数据链路套接字地址结构

```c <br/>&ensp; <net/if_dl.h>
struct sockaddr_dl{
  uint8_t       sdl_len;           //数据链路层地址是可变长度的，这个成员即为记录地址结构的长度
  sa_family_t   sdl_family;        //AF_LINK
  uint16_t      sdl_index;         //网络接口的正值索引，if > 0
  uint8_t       sdl_type;          //IFT_ETHER等
  uint8_t       sdl_nlen;          //名字的长度
  uint8_t       sdl_alen;          //链路层地址的长度
  uint8_t       sdl_slen;          //链路层选择器的长度
  char          sdl_data[12];      //数据域:         0 ~ sdl_nlen - 1  ==>  name
                                   //         sdl_len ~ sdl_alen - 1  ==>  link-layer address
}
```
> ## 创建一个路由套接字

```c
int sockfd;
sockfd = socket(AF_ROUTE, SOCK_RAW, 0);
```



> ## Key Managment 概述

- PE_KEY domain: **唯一支持的套接字是原始套接字**(raw socket)

> ## 创建一个秘钥管理套接字

```c
int sockfd;
sockfd = socket(PF_KEY, SOCK_RAW, 0);
```
