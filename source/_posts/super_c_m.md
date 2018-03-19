---
layout:     post                    # 使用的布局（不需要改）
title:      高级C与网络编程复习（13）—— 线程（Threads）（第26章）         # 标题
subtitle:   线程（Threads）        #副标题
date:       2018-01-06 19:00:00           # 时间
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

- ### 用fork实现并发要求存在的问题
  - fork的代价是比较昂贵的
  - fork返回之后父子进程之间的信息传递需要进程间通信(IPC)机制。
- ### 使用线程可以有效的解决上面的问题
  - 线程的创建是轻量级的，线程也被称为轻权进程（lightweight process）。线程的创建可能比进程快10~100倍
  - 同一进程内的所有线程共享全局内存，这使得线程之间易于共享信息，然后随之而来的便是同步(synchronization)问题
- ### 同一进程内的线程共享：
  - 全局变量
  - 进程指令（process instruction）
  - 大多数数据（most data）
  - 打开的文件（open file）==> 即描述符
  - 信号处理函数和信号处置（Signal handlers and signal disposition）
  - 当前工作目录（Current working directory）
  - 用户ID和组ID（User and group ID）
- ### 线程独享以下内容：
  - 线程ID （thread ID）
  - 寄存器集合，包括程序计数器和栈指针 （Set of registers， including program counter and stack pointer）
  - 栈（用于存放局部变量和返回地址）  Stack（for local variables and return address）
  - errno ==> 全局动态变量（记录最近的一次错误）
  - 信号掩码(Signal mask)
  - 优先级(Priority)
