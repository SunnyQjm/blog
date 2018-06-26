---
layout:     post                    # 使用的布局（不需要改）
title:      Chapter 2 Applicaiton Layer         # 标题
subtitle:                          #副标题
date:       2018-5-6 18:00              # 时间
author:     Ming.J                      # 作者
header-img: img/blog-header1.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - 计算机网络
    - 学习
    - 笔记
---

> ## Principles of Network Applications
- **只有端系统中才有应用层和传输层**

- ### 网络应用体系架构（Network Application Architectures）
  - client-server （C / S）
  - Peer-to-peer (P2P)
  - Hybird of C/S and P2P (混合架构)
  > ps: 与网络体系架构（5层架构、OSI/ISO七层架构...）做一下区分

  - #### C/S
    - ##### 特点
      - **服务端要一直在线**，且要 **先于客户端启动**
      - 服务端通常 **要有一个固定的，长期不变的唯一地址**（IP Address）
      - 客户端之间并不直接进行通信，**所有的客户端都直接与服务端通信**
      - 易于管理
    - ##### 缺点
      - 单点故障
      - 接入客户端过多的时候，容易导致服务器过载（所以服务端通常采用分布式集群，且性能一般都比较强大）
      - 维护、升级成本大
  - #### P2P
    - 不需要一直在线的服务端
    - 任意两个主机之间都可以直接通信
    - **高可扩展性** （***设备增加不会系统影响整体的性能***），但是不易管理

- ### Process Communication（进程通信）
  > **同一主机的进程之间** 通过 **进程间通信（innerprocess communication）** 进行通信，而不同主机的进程之间通过交换数据报（message）进行通信 => 发送进程将构造message并将其发送到网络当中，接收进程从网络中接收message，并解析它

  - #### Client and Server Processes （客户端和服务端进程）
    ***In the context of a communication session between on pair of processes, the process that initiates the communication (that is, initially contacts the other process at the begining of the session) is labeled as the client, The process that waits to be contacted to begin the session is the server***

    - 通常我们将 **发起通信的一端标记为客户端**

    - 不只是C/S架构中有 **服务端和客户端的概念，P2P架构中同样也有**，不同的是在C/S架构中一个进程扮演的角色是固定的，而 **P2P架构中同一个进程既可以是客户端，也可以是服务端**，甚至可以同时扮演两个角色

  - #### The Interface Between the Process and the Computer Network (进程和网络之间的接口) => Socket
    {% img  /img/computer_network/computer_network_08.png 进程和Socket关系图解 %}

    - **Socket作为Process和Network之间的门户** ，发送和接收message都需要经过Socket。可以把Process比作一个工厂或仓库，而Socket就是工厂或长裤的大门

    - 一个Socket只属于一个Process，而一个Process可以持有多个Socket

    - 我们用 IP+Port 标识一个进程

- ### Transport Services Available to Applications （传输层给应用层所提供的服务）
> 同时也可以说是理想情况下，应用层需要传输层给它提供哪些服务

  - #### Reliable Data Transfer (数据的可靠传输)
    传输层需要向应用层提供进程到进程的可靠通信
  - #### Throughput （最小带宽的保障）
    有些网络应用（比如网络电话）是对带宽比较敏感的，被称为 **带宽敏感型应用（bandwidth-sensitive-applications）**。这些应用对吞吐量有一定要求，要求吞吐量至少高于某个速率才能正常工作这样子。所以传输层就需要能为应用层提供保证吞吐量的服务
  - #### Timing （延迟保障）
  - #### Security （安全保障）

- ### Transport Services Providede by the Internet
  > 互联网所提供的传输层服务

  - #### TCP Services
    > 提供数据的可靠传输，不提供最小带宽，最短时延的保障，不提供安全保障

    - connect-oriented (面向连接)
    - reliable transport (可靠传输)
    - flow control (流量控制)
    - congestion control (拥塞控制)
  - #### UDP services
    > 不提供可靠数据传输，不提供最小带宽，最短时延的保障，不提供安全保障

  - #### 一些经典应用程序所采用的应用层协议和传输层协议
    {% img  /img/computer_network/computer_network_09.png 一些经典的应用程序所采用的应用层协议和传输层协议 %}
    - TCP: Telnet, FTP, SMTP, HTTP
    - UDP: SNMP, DNS, RTP, RIP

> ## The Web and HTTP

- ### HTTP
  > HTTP——HypreText Transfer Protol（超文本传输协议）

  - HTTP是应用层协议（Application protocol）
  - HTTP采用的是C/S架构
  - 基于TCP
  - HTTP是无状态的(stateless) => 不保存用户端的状态信息，所以HTTP协议也被称为无状态协议（stateless protocol）
- ### Non-Persistent and Persistent Connections
  - #### RTT (Round-trip time, 往返时间)
    **定义**：一个很小的数据包（小到可以近乎忽略数据包的长度）从客户端到服务端往返一次所需要的时间
    {% img  /img/computer_network/computer_network_02.jpg RTT %}

  - #### 一次HTTP请求的响应时间
    {% img  /img/computer_network/computer_network_10.png 一次HTTP请求的响应时间 %}
    - total = 2*RTT + transimit time
    - 总的响应时间 = 2倍的RTT + 收到整个返回的对象所需要的时间（文件长度/带宽）
  - #### Non-Persistent HTTP （非坚持的HTTP）
    每建立一个TCP连接，只能传一个对象（object => HTML file, JPEG...）

    - 由于接收每个对象都要单独建立连接，所以 **接收每个对象所需要的响应时间都包含至少两个RTT**
    - **服务器负担重**：传送每个对象都需要服务器维护一个新的连接，而连接的建立和维护是有一定开销的
  - #### Persistent HTTP （坚持的HTTP）
    一个连接建立可以传第一个Object
    - 这个接收第一个对象的时候需要至少两个RTT，紧接着后面的每个对象都少了建立连接的RTT了。
- ### HTTP Message Format
  - #### HTTP Request Message (HTTP请求消息格式)
    {% img  /img/computer_network/computer_network_11.png HTTP请求消息的通用格式 %}

    - **method field** (方法域)：GET, POST, HEAD, PUT, DELETE ...
    - **URL**: 请求位置
    - **Version**：HTTP协议的版本号
    - **Header lines**: 包含了请求头信息，请求头中包含一系列的键值对
    - **Entity body**: 数据域，当使用GET请求的时候，这个域是为空的，当使用POST请求的时候，这个域包含了请求所携带的数据
    - **HTTP Request message都是用ASCII text表示的**
    - 需要注意的是，请求中携带数据，不一定要使用POST方法， **GET方法也同样可以携带数据**。虽然GET请求的Entity body域是空的，但是GET方法可以在URL后面包含数据。
  - #### 一个HTTP请求的简单例子
    ```
    GET /somedir/page.html HTTP/1.1
    Host: www.someschool.edu
    Connection: close
    User-agent: Mozilla/4.0
    Accept-language: fr
    ```
    - 其中第一行分别是请求方法，URL和HTTP版本
    - 后面四行是请求头
      - Host: 服务器主机的域名
      - Connection: close => 要求服务器在发送完所请求的数据之后，不保持连接
      - User-agent => 通常用来标识浏览器内核版本，服务端可以通过这个请求头对不同的浏览器返回不同的文件，这样就可以实现同一个URL针对不同的浏览器返回不同数据的功能
      - Accept-language => 告诉服务器前端所期望服务器返回数据的语言
  - #### HTTP Response Message （HTTP返回消息格式）
    {% img  /img/computer_network/computer_network_12.png HTTP返回消息的通用格式 %}

    - **status line**
      - **version**: HTTP协议的版本号
      - **status code**: 本次请求的状态码
      - **parase**: 对状态码的简单描述
    - **Header lines**: 包含了返回头信息，返回头中包含一系列的键值对
    - **Entity body**: 数据域，如果有的话包含了服务器返回的数据

  - #### HTTP Response Status Code (HTTP 返回消息状态码)
    > [状态码列表](http://tool.oschina.net/commons?type=5)

    - 200 OK: 本次请求成功，并且所请求的object对象已经在Response消息中返回
    - 301 Moved Permanently: 所请求的Object已经被移到另一个位置，在Response Message的Location 头中包含了Object的新位置，正常情况下浏览器会自动访问新地址
    - 400 Bad Request: 服务器无法理解客户端的请求
    - 404 Not Found: 所请求的object在服务器上不存在
    - 505 HTTP Version Not Supported: 请求的HTTP协议版本不被服务端支持

  - #### 一个HTTP回复消息的简单例子
    ```
    HTTP/1.1 200 OK
    Connection: close
    Date: Sat, 07 Jul 2007 12:00:15 GMT
    Server: Apache/1.3.0 (Unix)
    Last-Modified: Sun, 6 May 2007 09:23:24 GMT
    Content-Length: 6821
    Content-Type: text/html

    (data data data data data ....)
    ```
    上面的消息中包含：an initial status line，six header lines，the entity body
    - 第一行表示服务端HTTP协议版本号为1.1，客户端本次请求的状态码为200，状态码的描述为OK（表示客户端请求的服务器存在，并且服务器上存在客户端所请求的文件）
    - **Connection: close** => 告知客户端，服务器发送完这个Object对象之后就会关闭连接
    - **Date** => 表示服务器创建这个message的时间
    - **Server** => 包含了服务器的一些信息，与User-Agent类似
    - **Last-Modified** => 表示这个Object的最后修改时间，这个在浏览器进行缓存处理的时候极为有用
    - **Content-Length** => 返回Object的数据长度
    - **Content-Type** => 返回Object的类型

- ### User-Server Interaction: Cookies (用户服务器交互：Cookies)
  - #### Cookies 技术四大组件
    - A cookie header line in the HTTP response message (HTTP返回消息中的一个cookie头行：例如 **Set-cookie: 1678**)
    - A cookie header line in the HTTP request message（HTTP请求消息中的一个cookie头行：例如 **Cookie: 1678**）
    - A cookie file kept on the use's end system and managed by use's browser (一个cookie文件，持久化在客户终端，并且由客户端浏览器统一管理)
    - A back-end database at the Web site (网站上的后端数据库 => 服务器上记录用户的状态信息，通过请求中的cookie，从数据库中取出对应的信息).
  - #### 交互示例
  {% img  /img/computer_network/computer_network_13.png 用户通过Cookies与服务器交互示意图 %}
    - 用户一开始向服务器发送一个正常的HTTP请求
    - 服务器在返回的消息中，通过 **Set-cookie** 头示意客户端浏览器保存一个Cookie。客户端浏览器如果打开了Cookie支持，就会将收到的Cookie存起来，持久化到Cookie文件里面
    - 之后对同一网站的请求，浏览器都会在每个请求消息中自动加入 **Cookie** 头，并将之前持久化保存起来的Cookie信息取出，填上去
    - 服务器收到带Cookie头的请求就会去数据库查询相关信息，

  - 使用Cookie技术，可以 **在无状态的HTTP上层创建了一个可以保持用户状态的用户会话层**

- ### Web Caching（Web缓存）
  > **Web Caching**（Web缓存）也称之为 **proxy server**（代理服务器）。是一个可以代替源端Web服务器响应客户端请求的这么一个网络实体
  > - WEB缓存在同一时刻既作为Client（客户端），也作为Server（服务端）

  - #### Web Caching 的工作流程
    {% img  /img/computer_network/computer_network_14.png Web缓存工作示意图 %}
    1. 首先，客户端浏览器建立一个到Web缓存的TCP连接，并将请求发送给Web缓存
    2. Web缓存收到客户端的请求之后，检查本地是否有客户端要请求的object的本地缓存，如果有就将其放入HTTP Response Message中返回给客户端，否则转第三步
    3. 如果Web Caching中没有客户需要的Object，Web缓存就根据客户端请求中的目标地址，向源Web服务器发起一个请求，获取到客户需要的Object
    4. 接着Web缓存将从源服务器中收到的Object缓存到本地，并奖客户端需要的Object放入HTTP Response Message返回给客户端
  - #### 采用Web缓存的好处
    1. WEB缓存可以大大地减少客户端请求的响应时间，特别是当客户端与源服务器的瓶颈带宽远小于客户端与WEB缓存之间的瓶颈带宽的时候。=> **缩短响应时间，提高响应速度**
    2. Web缓存能够大大减少一个机构的接入 链路到闲特网的通信量。通过减少通信量，该机构(如一家公司或者一所大学)就不必急 于增加带宽，因此降低了费用。此外，Web缓器能从整体上大大减低因特网上的 Web 流量，从而改善了所有应用的性能。 => **减少源服务器的负载，降低机构的出口带宽费用，改善整个因特网的性能**
    {% img  /img/computer_network/computer_network_15.png 添加本地Web缓存之后的系统 %}

    3. 可以提供另一条路径去访问远程服务器（VPN）。客户端访问不到源服务器但是可以访问Web缓存，而Web缓存可以访问到源服务器，那客户端就可以通过WEB缓存的代理，间接访问源服务器
  - #### 采用Web缓存可能存在的缺陷
    - 信息的滞后性，客户端获取到的可能不是源端最新的版本
    - 解决办法：通过 **Conditional GET(有条件的GET)**，当Web缓存收到客户端的请求的时候，就向源服务器发送一个有条件的GET，询问客户端请求的Object与Web缓存本地的缓存的对象相比是否有更新。如果有更新，就从源服务器下载Object，更新本地的缓存，并将最新的Object返回给客户端；如果没有更新，Web缓存就直接将本地缓存的Object返回给客户端。

- ### Conditional GET (有条件的GET)
  - #### 什么是有条件的GET
    - The Request message use the **GET** method （用GET方法请求）
    - The Request message includes an **If-Modified-Since:** header line （请求的时候带上If-Modified-Since头）
  - #### 实例
    1. 一开始Web缓存本地没有缓存，收到一个客户端的请求之后就转而去请求源服务器
      ```
      GET /fruit/kiwi.gif HTTP/1.1
      Host: www.exotiquecuisine.com
      ```
    2. 接着源服务器会返回类似如下的返回消息
      ```
      HTTP/1.1 200 OK
      Date: Sat, 7 Jul 2007 15:39:29
      Server: Apache/1.3.0 (Unix)
      Last-Modified: Wed, 4 Jul 2007 09:23:24
      Content-Type: image/gif

      (data data data data ...)
      ```
    3. **Web缓存收到源服务器返回的消息之后**，就会将Object缓存下来，同时也 **会把对应的Last-Modified头信息（Object最后修改时间）保存下来**。一段时间后，比如过了一周，又有一个客户端向Web缓存请求这个Object，由于过了一段时间了，这个Object可能被修改/更新过了，所以 **Web缓存需要向源服务器发送一个Conditional GET**， 以确定是否要从源服务器更新本地的缓存。Conditional GET请求如下：
      ```
      GET /fruit/kiwi.gif HTTP/1.1
      Host: www.exotiquecuisine.com
      If-Modified-Since: Wed, 4 Jul 2007 09:23:24
      ```
    4. 源服务器收到一个Conditional GET之后，会检查目标Object的最后修改时间和GET请求中 **If-Modified-Since:**字段的时间孰大孰小，如果时间一致，表示不用更新，会返回如下信息（如果时间不一致，则正常返回要请求的Object）：
      ```
      HTTP/1.1 304 Not Modified
      Date: Sat, 14 Jul 2007 15:39:29
      Server: Apache/1.3.0 (Unix)

      (empty entity body)
      ```
    5. Web缓存的 **Conditional GET请求** 如果收到 **源服务器返回的304状态码**，就认为本地缓存的Object是最新的，直接将其放入HTTP Response Message 当中返回给客户端

> ## File Transfer: FTP （文件传输协议）

  {% img  /img/computer_network/computer_network_16.png FTP协议工作示意图 %}

- ### 特点
  - 基于TCP
  - 采用C/S架构
  - 是应用层协议
  - 控制流和信息流分离，**控制信息的传输采用带外传输**（out-of-bound）的方式
- ### 两种连接
  {% img  /img/computer_network/computer_network_17.png FTP控制连接和数据连接 %}
  - FTP协议在 **21端口上传递控制信息**，在 **22端口上传递数据**（文件）
  - 初始时客户端通过向服务器的21号端口建立一条TCP连接（control connection），并且向服务端发送identifycation，username，password等信息。接着服务端向客户端建立一条TCP连接用于传输数据（data connection）。即便在同一个Session，每传递一个新的文件都需要新开一个data connection
  - **控制连接**（control connection）始终处于连接的状态，并且是维持用户状态的（state），**数据连接** 在每传输完一个文件之后就关闭，同时传输多个文件会开启多个数据连接（**Non-Persistent Connection**）

> ## Electronic Mail in the Internet (因特网中的电子邮件)
> - SMTP（简单邮件传输协议）=> TCP
> - POP3
> - IMAP

- ### 对比SMTP和HTTP
  - #### 同：
    - 两种协议都是用于从一个主机向另一个主机传输文件
    - 持久HTTP和SMTP都采用持久连接
    - 都是应用层协议
    - 都基于TCP
  - #### 异：
    - **HTTP** 是一种 **拉式协议（pull protocol）**，它由想接受文件的一端发起连接；而 **SMTP** 是一种 **推式协议（push protocol）**，它由想发送文件的一端发起连接
    - **SMTP** 的message需要使用 **7-bits ASCII** 格式，对于不是7-bits ASCII格式的文字或者包含比特流，都需编码为 7-bits ASCII 格式之后传送。而相比之下，HTTP就没有这种限制
    - 对于包含文本和图形的文档
      - HTTP是把每个Object放到一个Response message中发送
      - 而SMTP是把所有的Object放在一个报文中发送
  -  SMTP是一种推式协议，所以不能用来从邮件服务器上获取邮件，而应该采用其它协议来获取邮件
- ### 收发邮件所用的协议
  - #### 发邮件
    - **SMTP** (Simple Mail Transfer Protocol)
  - ### 收邮件
    - **POP3** (Post Office Protocol——Version 3) => 邮局协议，端口110
      - "download and delete" mode
      - "download and keep" mode
    - **IMAP** (Internet Mail Access Protocol) => 互联网邮件访问协议
    - **HTTP** (HyperText Transfer Protocol) => 超文本传输协议

> ## DNS
> - Domain Name System, 域名系统
> - **DNS 协议基于UDP**，PORT 53

- ### DNS Servers 组成
  - ***A distributed databse implemented in a hierarchy of DNS Servers*** (一个由分层的DNS服务器实现的分布式数据库)
  - ***An application-layer protocol that allows hosts to query the distributed database*** (一个使得主机能够查询这个分布式数据库的应用层协议)
- ### Services Provided by DNS (DNS所提供的服务)
  - 实现域名（***hostname***）和IP地址的转换 => 最主要的功能
  - 支持给主机设置别名（***Host aliasing***）=> 可以给复杂难记的域名指定一个简单易记的别名，别名指向的原域名我们称之为 **规范主机名**(***canonical hostname***)。我们可以用这个别名通过DNS服务器查到其规范主机名，进而查到对应的IP
  - 支持给邮件服务器设置别名 (***Mail Server aliasing***)
  - 支持负载均衡 (***Load distribution***) => 对同一个域名的访问/请求，可以由多个服务器分担
- ### DNS服务为什么不采用集中式设计（centralized design）
  - 单点故障 （***A Single point of failure***）
  - 通信容量限制（***Traffic volume***）=> 单个服务器无法处理上亿个甚至更多的请求
  - 远距离的集中式数据库（***Distant centralized databse***）=> 单个DNS服务器无法临近所有主机，距离越远，所产生的时延也就越大
  - 维护困难（***Maintenance***）=> 单个DNS服务器将为互联网的所有主机保存数据，将会导致中央数据库非常的庞大，难于维护。
- ### DNS Server 是一个分布式、分层的数据库
  {% img  /img/computer_network/computer_network_18.png DNS Server 分层 %}
  - Root DNS Servers => 根域名服务器（全球只有13个，大部分在北美）
  - Top-level domain (TLD) => 顶级域名服务器
  - Authoritative DNS Servers => 权威域名服务器
  - Local DNS Server => 本地域名服务器（通常每个ISP会有一个本地的域名服务器）
- ### 两种查询方式
  - #### 交互式查询（Interactive Queries）
    {% img  /img/computer_network/computer_network_19.png 交互式查询方式示意图 %}
  - #### 递归式查询（Recursive Queries）
    {% img  /img/computer_network/computer_network_20.png 递归式查询示意图 %}
- ### DNS Caching (DNS缓存)
  DNS服务器会对查询过的域名对应有缓存，并且该缓存会有一个存活时间，被访问的越频繁越不容易失效
  - DNS Caching 可以减少客户端查询DNS服务器所导致的延迟
  - DNS Caching 可以降低高级域名服务器的负载

- ### DNS Records
  一个DNS记录是如下的四元组
  ```c
  // 域名，值，类型，存活时间（Time-to-live）
  (Name, Value, Type, TTL)
  ```
  - #### 记录的类型
    - **A记录**：Name -> hostname, Value -> IP

      例：(relay1.bar.foo.com, 145.37.93.126, A)
    - **NS记录**：Name -> ，则 Name 是个域 (如foo. com) ，而 Value 是个知道如何获得该域中主机IP地址的权威DNS服务器的主机名

      例：(foo.com, dns.foo.com, NS)
    - **CNAME记录**：则 Value 是别名为 Name 的主机对应的规范主机名

      例：（foo.com, relat1.var.foo.com, CNAME）
    - **MX记录**：，则 Value 是个别名为 Name 的邮件服务器的规范主机名

      例：（foo.com, mail.bar.foo.com, MX）

> ## Peer-to-peer Applicaions（P2P应用程序）
> - 共享资源
> - 节点间课直接通信
> - 非集中式

- ### C/S 架构中存在的问题
  - 难于扩展（由于服务端负载有限额，当客户端达到一定额度之后，就很难在扩展了）
  - 单点故障
  - 需要超级节点作为管理者（服务器的性能要足够好）
  - 网络边缘的资源未得到充分的利用
- ### P2P 的特点
  - 资源利用率高
  - 可扩展性好
  - 可靠性/健壮性好（无单点故障，文件有多份拷贝）
  - 不需要超级节点管理，节点自管理
  - 隐私不可控（BT）
  - 动态性更强
  - 搭便车问题
- ### Robustness（健壮性）
  系统受到干扰之后迅速恢复回原来状态的能力。
- ### Flooding（洪泛）
  每次受到邻居节点消息之后都无条件的转发给其它邻居，导致洪泛发生。
