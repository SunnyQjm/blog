---
layout:     post                    # 使用的布局（不需要改）
title:      Chapter 3 Transport Layer         # 标题
subtitle:                          #副标题
date:       2018-5-9 18:00              # 时间
author:     Ming.J                      # 作者
header-img: img/blog-header1.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - 计算机网络
    - 学习
    - 笔记
---

> ## Introduction of Transport-Layer Servicers （传输层提供的服务）
> - 传输层将网络层所提供的主机到主机之间的 **逻辑通信**（***logical comminication***）扩展为进程到进程间的通信 => **!!最主要的的功能**
> - **传输层只在端系统（end systems / hosts）中才有**，在路由器和二层交换机上没有
> - Inetnet的传输层协议：**TCP、UDP**

- ### 传输层 VS 网络层
  - **网络层** 完成的是 **主机到主机** 之间的逻辑通信， **传输层** 完成的是不同主机间 **进程到进程** 的逻辑通信 => 可以说，**传输层扩展了网络层的配送域**

    PS: 并不是说传输层可以单独完成不同主机间进程到进程通信，肯定是要基于网络层及之下各层所提供服务的支持。但是以应用进程的视角来看，确实它只要把通信的消息传给传输层，传输层会帮他把消息投递到目的主机的传输层，目的主机进程就从它的传输层中获取消息。
  - **网络层提供的是不可靠的传输**（***best-effort delivery service***）， 而 **传输层可以** 在网络层不可靠的基础上引入一些可靠数据传输的机制，来向上 **提供可靠的传输**（如：TCP）

  - 网络层和传输层都不能提供最短延迟和最小带宽的保障
- ### Internet 传输层协议
  - #### TCP
    - 可靠数据传输 （***reliable data transfer***）
    - 保证数据有序分发
    - 拥塞控制（***congestion control***）
    - 流量控制（***flow control***）
  - #### UDP
    - 不可靠
    - 无序分发

> ## Multiplexing and DeMultiplexing （分解与复用）
> 传输层通过复用收集各进程的消息，通过分解将消息分发给各个进程

- 各层的PDU（某一层协议所处理的最小数据单元）
  {% img  /img/computer_network/computer_network_01.jpg 五层网络协议栈模型 %}
- Socket 是数据穿梭于网络进程和网络之间的大门，同时也就是传输层和进程之间的门户。与传输层直接通信的其实是进程中的Socket（一个进程可以由多个Socket）
- ### Demultiplexing（分解）
  传输层将收到的网络层投递给它的Segments，去除传输层头部，分发给正确的Socket
- ### Multiplexing（复用）
  传输层收集来自各个Socket的message，并加上传输层头部，生产Segment，再交由网络层

- 每个Socket对应于一个端口号，传输层在分发Segments的时候通过目的端端口号判断应该分发给哪个Socket
- ### UDP 的 Multiplexing and DeMultiplexing
  - 一个 **UDP Socket** 由一个二元组唯一标识（目的IP，目的端口号）
  - UDP协议在分发的时候就根据上面提到的二元组分发Segment给对应的Socket（实际上是去除传输层头部，然后将里面的Message发送给Socket）
  - **来自不同IP、端口（源IP，源端口），但是目的IP和端口号一致的Segment会被分发给同一个Socket**
- ### TCP 的 Multiplexing and DeMultiplexing
  - 一个 **TCP Socket** 有一个四元组唯一标识（源IP，源端口号，目的IP，目的端口号）
  - TCP协议在分发的时候就根据标识TCP Socket的四元组来分发Segments
  - **来自不同IP、端口（源IP，源端口）的Segment会被分发给不同的Socket**

> ## Connectionless Transport: UDP （无连接的传输层协议）——UDP）
> - UDP（User Datagram Protocol, 用户数据报协议）
> - 基于UDP的应用层协议：SNMP、RIP、DNS、NFS

- ### 为什么需要UDP？
  - UDP **不需要建立连接** （少了建立连接的开销以及带来的时延）
  - **实现简单** => 发送者与接受者之间由于是无连接的，所以不需要保存连接状态信息
  - **没有拥塞控制** => 不限速
  - **UDP头部较小**（***Small packet header overhead***）=> UDP由于需要实现的功能较少，头部的字段也比较少，只需要8 byte
- ### UDP的应用场景
  - 多媒体（在线视频）
  - 具有重复性操作的场合（如：DNS）
  - 一对多通信的场景

  {% img  /img/computer_network/computer_network_21.png 常见应用层协议所用的传输层协议 %}
- ### UDP所提供的服务
  - 传输层将网络层所提供的 **主机到主机** 之间的逻辑通信 **扩展到进程到进程** 间的逻辑通信
  - **校验和**（***Checksum***）
- ### UDP Segment Structure
  {% img  /img/computer_network/computer_network_23.png UDP Segment Structure %}
  - **首部** 只有四个字段，**只占8 bits** 结构异常简单
  - **Length指的是整个Segment的长度**，包括数据域，以字节为单位
  - **UDP的Checksum只能检错，不能纠错**

> ## Principles of Reliable Data Transfer（可靠数据传输原理）
> - **校验和** => 检验比特差错
> - **ACK反馈**
> - **序列号** => 解决冗余数据包问题（冗余数据包指的是接收端会收到一个数据包的多份拷贝）
> - **定时器超时重传** => 解决丢包问题
> - **重传**

{% img /img/computer_network/computer_network_24.png 传输层实现可靠数据传输示意图 %}

- ### rdt1.0
  => **假设信道完全可靠，无丢包，无比特差错**
  {% img /img/computer_network/computer_network_25.png rdt1.0协议有限状态机示意图 %}
  - 发送端传输层只要完成从应用层接收message，打包（给message加上传输层头部，构建Segment），并发送给可靠信道进行传输。而接收端传输层只需要从信道获取Segment，解包，然后分发给对应的Socket即可
- ### rdt2.0
  => **假设信道无丢包，但是有可能发生比特差错**
  => rdt2.0是一个典型的 **ARQ协议**，也是一种**停等协议**（***stop-and-wait protocol***）

  {% img /img/computer_network/computer_network_26.png rdt2.0协议有限状态机示意图 %}
  - #### ARQ (Automatic Repeat reQuest) protocols
    ***In a computer network setting, reliable data transfer protocols based on such retransmission are known as ARQ prorocols***

    在计算机网络中，**基于重传的可靠数据传输协议**，我们称之为ARQ协议

    - **ARQ协议的要求：**
      - Error detection（错误检测）
      - Receiver feedback（接收反馈）
      - Retransmission（重传）
    - **rdt2.0 协议就是一种ARQ协议**
  - #### 发送端
    - 从上层应用层获取消息，打包，发送到信道传输。接着进入等待状态，**等待接收端的ACK/NAK**。
    - 如果收到ACK，则表示接收端收到了正确无误的数据包，状态切换回等待消息的状态，如果有消息就继续发送，如果没有，则等待应用层的消息
    - 如果收到NAK，则表示接收端收到了数据包，但是数据包 **发生了比特差错，则重传**之前发送的数据包
  - #### 接收端
    - 接收端从下层接收到数据包之后进行校验，如果 **发生比特差错**，则 **向发送端发送一个NAK并丢弃该数据包**
    - 如果 **没有发生比特差错**，则解包后 **传递给上层**，并**向发送端发送一个ACK**
  - #### 存在的问题
    如果当**接收端返回的ACK/NAK也发生了比特差错**时，发送端并不知道此次发送是否成功，那么它该作何处理？
    - ##### 思路一：xxx -> 你说什么？-> 你说什么？-> 你说什么？-> ...
      简单粗暴，接收端收到一个回复不知道是ACK还是NAK就再次询问客户端到底有没有正确接收。怕不是要打起来一会儿。。。
    - ##### 思路二：引入比特校准机制
      添加最够的Checksum，让其既具备检错功能，也具备纠错功能。这要求信道传输数据包的时候不能丢失（在rdt2.0的假设中是具备的，但是实际中是会发生丢包的），而且可能因为要兼具纠错功能，Checksum会很长
    - ##### 思路三：瞎几把重传
      一旦发送端收到的回复发生比特差错，无法判断的时候，就直接重传。这是一个稍微可行的方案，但是由于在无法判断传输是否成功的情况就重传，客户端可能会收到一个数据包的多个拷贝(**产生冗余数据包（duplicate packets）问题**)，这个时候客户端就不知道该如何判断服务器发过来的是一个没有接收过的数据包还是一个冗余的数据包 =>
  - #### 解决冗余数据包问题 => 引入序列号 (***sequence number***) 机制
    发送端在发送数据包的头部多加一个Sequence number的字段，接收端在接收到一个数据包的时候将其序列号取出，与自己已经接收到的前一个数据包的序列号对比，就可以判断这个数据包是不是冗余的了。因为rdt2.0采用是一种停等协议的工作模式，所以用1 bit的序列号就可以解决这个问题

- ### rdt2.1
  => 在rdt2.0的基础上采用上面思路三 + 引入序列号的方式解决了冗余数据包问题

  => 还是采用了NAK + ACK的实现方案
  {% img /img/computer_network/computer_network_27.png rdt2.1 Sender %}
  {% img /img/computer_network/computer_network_28.png rdt2.1 Receiver %}
- ### rdt2.2
  => 在rdt2.1的基础上，去除了NAK，收到错误数据包之后不发送NAK，而将最近一次成功接收到的数据包的ACK发送回去

  => 这种方案就导致发送端可能接收到同一个数据包的多个ACK (**冗余ACK，duplicate ACKs**)

  {% img /img/computer_network/computer_network_29.png rdt2.2 Sender %}
  {% img /img/computer_network/computer_network_30.png rdt2.2 Receiver %}

- ### rdt3.0
  => **假设信道可能丢包，但是有可能发生比特差错**

  => 在rdt2.2的基础上，考虑丢包问题。

  => **定时器 + 超时重传 => 解决丢包问题**
  {% img /img/computer_network/computer_network_31.png rdt3.0 Sender %}
  接收端FMS(finite-state machine, 有限状态机)图同rdt2.2
- ### Pipelined Reliable Data Transfer Protocols（流水线型可靠数据传输协议）
  {% img /img/computer_network/computer_network_32.png 停等协议与流水线型协议的对比 %}
  rdt3.0其实已经是一个可用的协议了，但是采用停等协议大大限制了网络的传输性能（发送端必须接收到接收端返回的ACK才发送下一个数据包或者重传，增大了传输时延，降低了带宽利用率）。而相比之下，***流水线型可靠数据传输协议*** **允许已发送未确认的数据包可以有多个**
  - Go-Back-N (回退N)
  - Selective repeat（选择重传）
- ### Go-Back-N (GBN)
  回退N
  > - **定时器超时**，所有已发送未被确认的数据包都 **重传**
  > - **采用累积确认**：收到ack=10，则表示序列号为10及之前的数据包都已正确接收，接收端下一个期望收到的数据包的序列号为11
  > - 接收端收到乱序的数据包直接丢弃

  - #### Go-Back-N滑动窗口
    {% img /img/computer_network/computer_network_33.png 发送端视角看回退N的序列号窗口 %}
    - **base** : 最早的发送却没有被确认的数据包的序列号
    - **nextseqnum** : 最小的没有使用的序列号（实际上指的是发送下一个非重传数据包时所使用的序列号）
    - **N** : 发送端序列号窗口的大小（限制发送端已发送未被确认数据包的数量，在流量控制的时候可以遏制发送端发包的速率）
    - **[base, nextseqnum - 1]** : 已发送未被确认的数据包的序列号范围
    - **[0, base - 1]** : 发送且已被确认的数据包的序列号
    - **[nextseqnum, base + N - 1]** : 剩余可用的序列号范围（指的是该时刻在不溢出发送窗口大小N的情况下，可以继续使用的序列号）
    > ps: 如果序列号的取值是以取模的方式循环使用，上面的范围划分就略有变动，但是意思是差不多的
  - #### Go-Back-N FSM 图
    - ##### 发送端
      {% img /img/computer_network/computer_network_34.png Go-Back-N发送端FSM图 %}
      - **上层的调用**（***Invocation from above***）: 当上层调用rdt_send()发送数据包的时候，**发送端检查发送窗口是否已满**（即检查一发送未确认的数据包的个数是不是已经达到N了），如果满了就通知上层拒绝发送（接着上层肯能会一段时间后重试），没有满则生成一个Segment并发送
      - **收到一个ACK**（Receipt of an ACK）: 发送端采用 **累积确认**（***cumulative acknowledgment***）的方式，例如：收到ack=10，则表示序列号为10及之前的数据包都已正确接收，接收端下一个期望收到的数据包的序列号为11
      - **超时事件**（***A timeout event***）: 发送端开始发送数据包的时候回启动一个定时器，之后**每收到一个ACK会检查是否还有已发送未确认的数据包**。有则重启定时器，没有则停止定时器。**一旦定时器超时，所有已发送但未确认的数据包都将被重传**（最多重传N个）
    - ##### 接收端
      {% img /img/computer_network/computer_network_35.png Go-Back-N接收端FSM图 %}
      - **接收端收到** 一个序列号为n的 **有序的数据包**（***in-order packet***, 表示接收端最近一次传递给上层的数据包的序列号为n-1），则给发送端回一个对应序列号的ACK，同时将数据包解包够传递给上层
      - **接收端收到错误的数据包** 则 **直接丢弃**
      - **接收端收到正确但乱序的数据包**（就比如收到的数据包序列号为n，但是上一次传递给上层的数据包的序列号不是n-1），接收端的处理也是 **直接丢弃**。这虽然看起来很浪费，但却是比较明智的处理方式。试想，如果接收端收到一个序列号为n+1的数据包，但是上一次传递给上层的却是序列号为n-1的数据包，则很可能序列号为n的数据包在传输的过程中丢失了，那么必然导致发送端计时器超时，发送端会重发数据包为n和n+1的数据包，所以接收端完全没有必要缓存序列号为n+1的数据包（这样实现起来简单并且有效）。
  - #### 补充
    - 发送端既然采用的是累积确认，那么如果发送端收到一个对序列号为n的ACK，但是上次收到的是n-3的，对n-1和n-2的ACK可能丢失了。这个时候发送端不会理会那两个丢失的ACK。因为接收端是有序接收的，所以发送端收到对序列号为n的数据包的ACK就可以认为序列号为n以及比n小的数据包都已经成功被接收了

- ### Selective Repeat（SR）
  选择重传
  {% img /img/computer_network/computer_network_36.png 选择重传（SR）发送方与接收方的序号空间 %}
  - #### 发送端
    - **从上层收到数据**（***Data received from above***）：当发送端接收到上层的数据时，**获取下一个可用的序列号**，如果该序列好在发送端序列号窗口内（表示当前窗口未满），则打包发送。如果不在序列号窗口内，则和GBN一样，发回给上层或缓存起来。
    - **超时**（***Timeout***）：发送端会为每一个已发送未确认的数据包维护一个逻辑计时器，一旦某个计时器超时，就重发它所负责监视的那个数据包。
    - **收到ACK**（***ACK received***）：
      - 收到的ACK所确认的数据包的序列号 **落在滑动窗口内，则将其标记为已确认**
      - 收到的ACK所确认的数据包的序列号 **等于send_base**（就是滑动窗口的第一个序列号），则 **将滑动窗口右移至第一个未被确认的序列号处**。
  - #### 接收端
    - 如果**序列号在[recv_base, recv_base + N - 1]范围内被接收**（也就是说收到的数据包在接收端的滑动窗口内）：向发送端 **回一个对该序列号的ACK**。
      - **如果该序列号不等于recv_base**，则将该序列号标记为已接收，将数据包缓存起来
      - **如果该序列号等于recv_base**，则接收端 **滑动窗口右移**至第一个未标记的序列号处。并将刚才扫过的数据包（就是[old_recv_base, new_recv_base - 1]之间的数据包）一并上交给上层。
    - 如果**序列号在[recv_base - N, recv_base - 1]范围内被接收**（也就是说这个刚收到的数据包之前已经接收过并回复ACK了）：向发送端回一个该序列号的ACK

      => 这个ACK看起来是多余的，其实是必须的。因为接收端将ACK发出之后就可能导致滑动窗口右移的操作。如果发送给发送端的ACK没有到达或者出错了，接收端在计时器超时之后就会重传，而这个时候发送端的窗口已经右移了。如果不再对这个数据包回复一个ACK，则导致发送端该序列号一直没有被确认，发送端就会不断重发这个数据包，而且滑动窗口也会停滞不前。

    - **其他情况**（发生比特差错，收到右溢出滑动窗口的包等）：忽略这个数据包，直接丢弃
  - #### 滑动窗口大小的问题
    => **滑动窗口的大小应该小于或等于序列号空间大小的一半**

    当序列号范围有限时，如果接收端滑动窗口太大，可能导致接收端无法判断刚接收到的数据包是一个新的未接收过的数据包还是重复的已经接收过的数据包
    - 举个栗子：
      {% img /img/computer_network/computer_network_37.png 采用有限序列号范围时，接收端滑动窗口太大的栗子 %}
      - 图中，**1等待接收**（可能由于丢包，比特差错等，等待发送端重传。或者是由于所走路由的问题，比2,3到来的晚），**2,3已经接收**，**4可以接收**
      - recv_base = 1, N = 4。显然，序列号是采用 mod 6 的方式复用的
      - 这个时候 **收到一个序列号为4的数据包**，
        - 那么它 **可能是一个新到来的落在滑动窗口内的数据包**(***[1, 2, 3, 4] => [recv_base, recv_base + N - 1]***)
        - 也 **可能是上一轮滑动窗口中已经接收并回复ACK**，但是由于发送端没有正确接收到ACK而重发的数据包，因此 **这个数据包可能是之前已经接收过的冗余数据包**（***[3, 4, 5, 0] => [recv_base - N, recv_base - 1]***）
      - **对于第一种情况，我们需要缓存该数据包并回复ACK，而对于第二种情况我们只需要回复ACK即可**

    - 为了避免上述情况，**滑动窗口的大小应该小于或等于序列号空间大小的一半**

> ## Connection-Oriented Transport: TCP
> - 点对点（***Point-to-Point***）
> - 传输可靠、有序的字节流 => 对每个字节进行编号
> - 流水线型（***Pipelined***）
> - 发送端和接收端都有缓存（***send & receive buffers***）
> - 全双工通信（***full-duplex service***）
> - 面向连接的（***connection-oriented***）
> - 拥塞控制（***congestion control***）
> - 流量控制（***flow control***）

- ### TCP接收和发送缓存
  {% img /img/computer_network/computer_network_38.png TCP发送和接收缓存 %}
  => **注意：**在TCP连接的两端各自都有自己的发送和接收缓存（因为TCP是全双工的，所以两端的终端都有发送和接收数据的需求）
- ### MSS（Maximum Segment Size, 最大报文长度）
  ***MSS is the maximun amount of application-layer data***

  MSS指的是Segment中应用层数据的最大长度，并不包括传输层头部

- ### TCP Segment Structure
  {% img /img/computer_network/computer_network_39.png TCP报文结构 %}
  - #### 字段说明
    - ***Source port*** : 源端端口号（***16 bits***）
    - ***Dest port*** : 目的端端口号（***16 bits***）
    - ***Sequence number*** : 序列号（***32 bits***）
    - ***Acknowledgment number*** : 确认号（***32 bits***）
    - ***Header length*** : TCP Segment头部（传输层头部）的长度（***4 bits***）。**单位是4个字节，一行**
    - ***Unused*** : 目前未使用的字段域（***6 bits***）
    - ***Flag filed*** : 标志域（***6 bits***）
      - ***URG、PSH*** : 与urgent data pointer filed 并用，处理紧急数据包（实践中未用到） => Don't care now
      - ***ACK*** : 标识这个数据包是不是一个ACK
      - ***RST、SYN、FIN*** : 这三个字段域用于创建TCP连接过程中
    - ***Receive window*** : 该字段用于流量控制，指示的是发送数据包一端，接收窗口的剩余可用大小
    - ***Internet checksum*** : 校验和，用于检测数据包是否出错
    - ***Urgent data pointer*** : 指向紧急数据尾的指针（实践中未用）=> Don't care
    - ***Options*** : TCP Segment头部的可选字段，可以没有
    - ***Data*** : TCP Segment的数据域，包含了上层传递下来的message
  - #### 补充说明
    - **TCP首部的开销至少是20个字节**，通常默认情况下Options字段域是为空的
    - **Header length 的单位是4个字节**（一行32bits）
  - ### TCP的序列号和确认号
    - 序列号和确认号是TCP提供可靠数据传输最重要的两个重要字段
    - **TCP** 把data看做有序的字节流。它按字节编号，**给每一个字节分配一个序列号** => **一个Segment的序列号由其数据域第一个字节的序列号所代表**
    - **TCP同样也是使用累积确认的机制**，但是和之前讲的有点不一样：***The acknowledgment number that Host A puts in its segment is the sequence number of the next byte Host A is expecting from Host B*** （主机A在给主机B发送的ACK包的确认号是主机A期望从主机B获取到的下一个数据包的序列号）

      例如：发送端如果收到确认号为536的ACK包，则表示序列号为536之前的所有数据包均已被成功接收，并且接收端期望下一个收到序列号为536的数据包

- ### Reliable Data Transfer => (TCP)的可靠数据传输
  - TCP在IP不可靠服务的基础上实现了可靠数据传输
  - **流水线型协议** => 带宽利用率高
  - **采用累积确认**
  - **TCP采用单个重传计时器**
  - **重传的时机**：
    - 定时器超时
    - 收到3次冗余ACK
  - **快速重传** => 当收到3次冗余ACK时（***实际上是收到了4个相同的ACK***），就认为定时器超时，重传已发送未确认的编号最小的数据包
- ### TCP中使用了哪些机制来实现可靠数据传输
  - **引入序列号来解决冗余数据包的问题**
    => 接收端收到序列号与之前收到的数据包序列号相同时，就丢弃该包
  - **采用累积确认**
    => 发送端如果收到确认号为536的ACK包，则表示序列号为536之前的所有数据包均已被成功接收，并且接收端期望下一个收到序列号为536的数据包
  - **TCP采用单个重传计时器+快速重传+累积确认来解决丢包问题**
    => 当发送端定时器超时或者收到相同数据包的3次冗余ACK时，便重传已发送未被确认的数据包中编号最小的数据包
  - **Checksum** => 用来检测数据包是否出现差错

- ### TCP交互流程图中ACK和ack
  - **ACK** => 表示的是TCP Segment中的ACK标志域，如果ACK=1则表示这是一个ACK包，如果ACK=0则表示这不是一个ACK包
  - **ack** => 表示的是TCP Segment中的确认号字段
- ### Flow Control（流量控制）
  - **调节发送端发包的速率，防止接收端缓存溢出**
  - **控制已发送未被确认的数据包的数量 ≤ Receive Window size** （接收端剩余缓存空间的大小 => 接收端通过ACK返回给发送端）
- ### TCP Connection Management（TCP连接管理）
  - #### TCP 三路握手
    {% img /img/computer_network/computer_network_40.png TCP三路握手 %}
    - ① 客户端首先向服务端 **发送一个SYN Segment**，其中SYN = 1，并且 **随机生成一个起始的序列号 client_initial = 5 赋给seq**（将随机生成的序列号填入SYN Segment的序列号字段）=> **没有应用层data，但是占用一个序列号**
    - ② 服务端收到客户端发送的TCP SYN Segment后，服务端分配发送和接收buffer以及相关变量。**同时发送一个SYNACK Segment** 给客户端，SYN = 1，ACK=1（表示这是一个ACK包），ack = 6 （client_initial + 1）。同时 **随机生成一个服务端的初始化序列号 server_initial = 17 赋值给seq** （通知客户端我已经接收到你建立连接的请求，并且返回给客户端自己的初始化序列号）。 => **SYNACK 包也没有应用层data，但也同样占用一个序列号**
    - ③ 第三步客户端给服务端返回一个普通的ACK，普通ACK是 **不占用序列号** 的，但是 **可以捎带数据**（捎带数据指的是，客户端可以在建立连接的最后一个ACK中，放入一部分要发送的数据。则这个数据包即完成了作为ACK的职能，也携带了一部分数据。）
  - #### TCP 四路挥手
    {% img /img/computer_network/computer_network_41.png 客户端发起的双向关闭TCP连接示意图 %}
    - 不含data的正常ACK，序列号可以不写（因为不占用序列号）；
    - 在非ACK数据包中的ack无含义，不用写。
  - #### TCP状态转换图 => [点我](http://qjm253.cn/2018/01/04/super_c_d/#%E7%8A%B6%E6%80%81%E8%BD%AC%E6%8D%A2)

> ## Principles of Congestion control
> 拥塞控制

**调节发送端发包的速率，避免路由器缓存溢出**

> ## TCP Congestion Control（TCP拥塞控制）

{% img /img/computer_network/computer_network_42.png TCP拥塞窗口的演化（Tahoe and Reno） %}

=> TCP Tahoe（代表系统首次启动的情况）： **1~4时刻为慢启动阶段**，cwnd以指数方式增长，其中 **时刻4的时候达到了系统初始设置的慢启动阈值**，系统进入拥塞避免状态，cwnd的增加开始放缓速度，以线性方式增加。到时刻 **8的时候定时器超时**，**ssthesh = cwnd / 2， cwnd = 1 (MSS)**。系统 **进入慢启动阶段**。到时刻12的时候达到了系统初始设置的慢启动阈值，系统进入拥塞避免状态。

=> TCP Reno（代表系统运行时收到3次冗余ACK的情景）：时刻9系统收到3次冗余ACK，**cwnd = cwnd / 2**，系统 **进入快速恢复阶段**

- ### 慢启动（***slow start***）
  首次启动时，拥塞窗口（***cwnd***）的大小置为1 MSS（***Maximum Segment Size***），接着每收到一个ACK，**cwnd** 的值便翻一番，指数增长。当 **定时器第一次超时时**，**cwnd的值置为1，同时将慢启动阈值** （***ssthresh***）的值设置为 **cwnd/2**。接下来 **cwnd** 仍然以指数方式增长，但是 **当达到ssthesh时，进入拥塞避免状态**（需要注意的是，初始的时候，ssthesh会设置有一个初始值，当如果第一次启动时在没有超时之前，cwnd达到了预设的ssthesh，也会进入拥塞避免状态）。

- ### 拥塞避免（***Congestion Avoidance***）
  当**进入拥塞避免状态之后**，**cwnd** 不再以指数方式增长，而是 **以线性方式增长**。
  - 当定时器超时时，**ssthesh = cwnd / 2， cwnd = 1 (MSS)**。系统 **进入慢启动阶段**；

  - 当收到3次冗余ACK时，**cwnd = cwnd / 2**，系统 **进入快速恢复阶段**

- ### 快速恢复（***Fast Recovery***）
  每收到一个冗余ACK，**cwnd + 1**
  - 如果引发TCP进入快速恢复阶段的 **缺失报文段在定时器超时之前到达**，**进入拥塞避免状态**。

  - 如果在此期间，**定时器超时，则ssthesh = cwnd / 2, cwnd = 1 (MSS)，进入慢启动状态**
