---
layout:     post                    # 使用的布局（不需要改）
title:      高C考试说明         # 标题
subtitle:   高C考试说明        #副标题
date:       2018-01-06 23:00:00           # 时间
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

> ## 说明

- 最后一个高C课，老师旁边围了一圈，就有了如下信息

> ## 题型

- 简答：50
- 编程：30
- 实验：20 （代码分析，分析哪里错了...）

> ## 以下不用看
> 括号里是我加的，以防万一

- timewait不用看
- key managment不用看 （知道怎么创建一个秘钥管理套接字：socket(PF_KEY, SOCK_RAW, 0)）
- ioctl 不用看

> ## 老师说会考

- 前几章重点
- 大端小端代码
- traceroute、ping会考（研究生学长说不考，但是今年王老师出卷，其它先复习，有时间看看ping和traceroute的源码）
- HDR_INCL套接字选项（原始套接字设置该选项可以自己构造IP首部）
- BPF、DLPI、SOCK_PACKET（三种提供链路层访问控制的方式）
- 掌握netstat怎么用（用于显示各个端口的状态）
