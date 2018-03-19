---
layout:     post                    # 使用的布局（不需要改）
title:      高级C与网络编程复习（5）—— TCP C/S程序示例（TCP Client/Server Example）（第五章）         # 标题
subtitle:   TCP C/S程序示例（TCP Client/Server Example）        #副标题
date:       2018-01-05 08:00:00            # 时间
author:     Ming.J                      # 作者
header-img: img/hacker.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - C语言
    - 网络编程
    - 学习

    - 笔记
---

> ## 简单回射程序概述

- 客户从标准输入读入一行文本，并写给服务器
- 服务器从网络输入读入这行文本，并回射给客户
- 客户从网络输入读入这行回射文本，并显示在标准输出上
{% img /img/unp_23.png 简单的回射客户/服务器 %}

> ## TCP回射服务程序

```c <br/>&ensp;tcpserv01.c
#include <unp.h>

void str_echo(int);

int main(int argc, char** argv){
    int listenfd, connfd;
    pid_t pid;
    socklen_t clen;
    struct sockaddr_in cliaddr, servaddr;

    listenfd = Socket(AF_INET, SOCK_STREAM, 0);

    bzero(&servaddr, sizeof(servaddr));
    servaddr.sin_family = AF_INET;
    servaddr.sin_port = htons(9748);

    //指定服务端socket的地址为通配地址
    //表示接收来自本机各个网络接口的连接请求
    servaddr.sin_addr.s_addr = htonl(INADDR_ANY);

    Bind(listenfd, (SA *) &servaddr, sizeof(servaddr));
    Listen(listenfd, LISTENQ);

    for( ; ; ){
        clen = sizeof(cliaddr);
        connfd = Accept(listenfd, (SA *) &cliaddr, &clen);
        //接受到来自客户端的请求之后，fork一个进程，在子进程中为客户提供服务
        //父进程则关闭本进程内该已连接描述符（引用计数减1）
        //然后再返回继续accept，可以达到并发的效果
        if( (pid = Fork()) == 0 ){      //子进程执行
            Close(listenfd);
            str_echo(connfd);
            Close(connfd);
            exit(0);
        }
        Close(connfd);                  //父进程执行
    }
}


/**
 * 为客户端提供服务
 * 从客户端接收一个字符串，并将字符串回射回客户端
 */
void str_echo(int connfd){
    ssize_t n;
    char buf[MAXLINE];

again:
    while((n = read(connfd, buf, MAXLINE)) > 0)
        Writen(connfd, buf, n);
    if(n < 0 && errno == EINTR)
        goto again;
    else if(n < 0)
        err_sys("str_echo: read error");
}
```

- 50 ～ 55: 这里用到的是系统的read函数，没有对信号中断错误处理，需要自己处理。（收到客户的FIN或EOF将导致read返回）

> ## TCP回射客户端程序

```c
#include <unp.h>

void str_cli(FILE*, int);

int main(int argc, char** argv){
    int sockfd;
    struct sockaddr_in servaddr;

    if(argc != 2)
        err_quit("usage: ./tcpserv01 <IPAddress>");
    sockfd = Socket(AF_INET, SOCK_STREAM, 0);

    bzero(&servaddr, sizeof(servaddr));
    servaddr.sin_port = htons(9748);
    servaddr.sin_family = AF_INET;

    //调用inet_pton函数将用户输入的点分十进制串转化成网络字节序的32为IPv4地址
    Inet_pton(AF_INET, argv[1], &servaddr.sin_addr);

    Connect(sockfd, (SA *) &servaddr, sizeof(servaddr));

    str_cli(stdin, sockfd);
    Close(sockfd);
}

/**
 * 用fgets读取用户的一行输入，发送给服务器，再从服务器接收一行回复并打印到控制台
 */
void str_cli(FILE* fp, int sockfd){
    char sendbuf[MAXLINE], recvline[MAXLINE];
    while(Fgets(sendbuf, MAXLINE, fp) != NULL){
        Writen(sockfd, sendbuf, strlen(sendbuf));
        if(Readline(sockfd, recvline, MAXLINE) == 0)
            err_quit("str_cli: server terminated prematurely");
        Fputs(recvline, stdout);
    }
}
```

> ## 正常启动

- ### 先启动服务器
  ```c
  //后台启动服务器
  ./tcpserv01 &
  [1] 10186

  //我们查看一下端口的状态，用管道过滤，只显示9748端口的信息
  netstat -a | grep 9748

  Proto Recv-Q Send-Q Local Address           Foreign Address         State
  tcp        0      0 0.0.0.0:9748            0.0.0.0:*               LISTEN
  ```
- ### 接着直接在本机连接服务器
  ```c
  //连接到本机的回环地址
  ./tcpcli01 127.0.0.1

  //再开一个shell，查看当前的端口状态状态
  netstat -a | grep 9748
  Proto Recv-Q Send-Q Local Address           Foreign Address         State
  tcp        0      0 0.0.0.0:9748            0.0.0.0:*               LISTEN
  tcp        0      0 localhost:9748          localhost:32890         ESTABLISHED
  tcp        0      0 localhost:32890         localhost:9748          ESTABLISHED
  ```
- 0.0.0.0 ==> 代表通配地址
- \*  ==> 代表通配端口
- **客户端**在**收到**三路握手的**第二个分节**的时候，**connect函数就返回**了。而**服务器**在**收到**三次握手的**第三个分节**的时候**accept才返回**（之前分析过，服务器在收到第二个分节的时候还处于SYN_RECV状态，只有在收到第三个分节的时候才进入ESTABLISHED状态，此时相应的socket才被扔进监听套接字的已完成队列。而accept只在已完成队列中取socket，所以accept必定是在收到第三个分节之后才返回）

> ## 正常终止

- 状态转换图：
  {% img /img/unp_4.png TCP状态转换图%}
- 进程终止：
  - 关闭本进程打开的所有的描述符
  - 向父进程发送一个SIGCHLD信号

```c
./tcpcli01 127.0.0.1
hello
hello         //服务器回射回来的
good bye
good bye      //服务器回射回来的
^D            //Ctrl + D 相当于输入EOF

//在客户端程序退出后马上查看端口状态（由于是本机测试，一定要快，要不然抓不到）
netstat -a | grep 9748
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:9748            0.0.0.0:*               LISTEN
tcp        0      0 localhost:33146         localhost:9748          TIME_WAIT
```
- **键入EOF**后，客户端的**fgets返回NULL**，导致**str_cli结束**，最终导致**main函数执行到exit而终止**
- **进程终止的部分任务是关闭进程打开的所有的描述符**，因此客户端打开的套接字由内核关闭，这导致**客户TCP向服务器发送一个FIN，服务器回一个ACK**。至此，四路挥手的前两步完成。**服务器处于CLOSE_WAIT状态，而客户端处于FIN_WAIT_2状态**
- 服务器TCP收到FIN后，readline函数返回0，导致str_echo退出，接着main函数执行到exit，进而导致服务端子进程退出。
- 同样的，服务端子进程所打开的所有描述符随之关闭。这导致服务器向客户发送一个FIN，客户回一个ACK，至此，四路挥手结束，连接完全终止。**客户套接字进入TIME_WAIT状态**。
- 进程终止的另一部分内容是：在服务器子进程终止时，给父进程发送一个SIGCHLD信号。由于我们在代码中没有捕获该信号，而该信号的默认处理为忽略，所以就导致**子进程进入僵死状态**
  ```c
  //我们调用ps命令来验证一下
  //因为笔者在测试的时候打开了客户端两次，所以有两个僵死的子进程
  ps -a
    PID TTY          TIME CMD
   2720 pts/0    00:00:21 hexo
  10186 pts/1    00:00:00 tcpserv01
  11146 pts/1    00:00:00 tcpserv01 <defunct>         //僵死进程
  13389 pts/1    00:00:00 tcpserv01 <defunct>         //僵死进程
  18996 pts/1    00:00:00 ps
  ```

> ## POSIX信号处理
> **信号**（signal）就是告知某个进程发生了某个事件的通知，有时也称为**软件中断**（software interrupt）。通常是异步的
> - 每个信号关联一个处置（deposition），或称行为（action）。在信号发生时执行

- ### 类型
  - 一个进程发给另一个进程（可以是自身）
  - 由内核发给进程
- ### 三种处置
  - **自定义信号处理函数**，然后用sigaction设置给信号
    - SIGKILL和SIGSTOP不能被捕获
    ```c
    //信号处理函数原型
    void handler(int signo);
    ```
  - SIG_IGN ==> 忽略信号
  - SIG_DEF ==> 默认处理
- ### signal函数
  - 原型：
    ```c
    void (*signal(int signo, void (*func)(int)))(int);

    //定义新类型，来化简上面的原型
    typedef void Sigfunc(int)

    /**
    * 为一个信号设置处理函数
    * @param signo      信号
    * @param func       信号处理函数
    * @return           指向信号处理函数
    */
    Sigfunc *signal(int signo, Sigfunc *func);
    ```
  - POSIX规定设置信号的处置必须调用sigaction，上面的signal是对signation的封装，更容易使用

> ## 处理SIGCHLD信号

- ### 僵死状态
  - 僵死（zombie）状态的目的是维护子进程的信息，以便父进程在以后某个时候获取。这些信息包括子进程的进程ID、终止状态以及资源利用信息。如果一个进程终止，而该进程有子进程处于僵死状态，那么它的所有僵死子进程的父进程ID将被重置为1（init进程）。继承这些子进程的init进程将清理它们
- ### 处理僵死进程
  ```c
  //首先定义如下信号处理函数
  void sig_chld(int signo){
    pid_t pid;
    int stat;
    pid = wait(&stat);
    printf("child %d terminated\n", pid);
    return;
  }

  //在上面的server端程序的Listen之后添加下面这行
  Signal(SIGCHLD, sig_child);
  ```
  - [点我查看源码](https://github.com/SunnyQjm/unpv13e/blob/master/job/chapter5/tcpserv01.c)
  - 在执行了上面的处理之后，再测试，就观测不到僵死进程了
- ### 处理被中断的慢系统调用
  - 适用于慢系统调用的基本规则：当阻塞于某个**慢系统调用**的一个**进程捕获某个信号且相应信号处理函数返回**时，该系统调用**可能返回一个EINTR错误**
  - 有些系统上会发生，有些系统做了处理，不会发生。但是为了便于移植，还是建议用类似于下面的方法处理这种错误
    ```c
    for( ; ; ){
      clilen = sizeof(cliaddr);
      if( (connfd = accept(listenfd, (SA *) &cliaddr, &clilen)) < 0){
        if(errno == EINTR)
          continue;
        else
          err_sys("accept error");
      }
    }
    ```
> ## wait和waitpid函数

```c
#include <sys/wait.h>

/**
* 用于在父进程中清理已终止子进程（解除子进程的僵死状态）
*
* @param  wait函数通过这个参数返回子进程的终止状态
* @return 成功则返回被清理子进程的ID，错误则返回0或-1
*/
pid_t wait(int *statloc);

/**
* 用于在父进程中清理已终止子进程（解除子进程的僵死状态）
* @param pid      用于指定清理那个子进程，如果传入-1则表示等待第一个终止的子进程
* @param statloc  函数通过这个参数返回子进程的终止状态
* @param options  可选项
* @return 成功则返回被清理子进程的ID，错误则返回0或-1
*/
pid_t waitpid(pid_t pid, int *statloc, int options);
```
- 调用wait函数的时候如果没有已经终止的子进程，不过仍然有一个或多个子进程在执行，那么wait函数将阻塞到其中任意一个子进程终止为止
- waitpid的options可选项如果制定为WNOHANG，则告知内核在没有已终止子进程时不要阻塞
- ### wait函数和waitpid的区别
  - waitpid可以指定终止哪个子进程，而wait不能
  - waitpid可以实现在没有已终止子进程时不要阻塞，而wait不能
- 根据上述区别的第二点我们改进之前的信号处理函数
  ```c
  //由于wait不能实现在没有已终止子进程时不要阻塞，所以在下面的循环中不能调用wait，否则可能会阻塞主线程
  //经过下面的修改之后，就可以支持一次调用清理多个进程的要求

  void sig_chld(int signo){
    pid_t   pid;
    int     stat;
    while( (pid = waitpid(-1, &stat, WNOHANG)) > 0)
      printf("chihld %d terminated\n", pid);
    return;
  }
  ```
- 具体可以参考课本5.10介绍的同时开五个连接请求的情况
