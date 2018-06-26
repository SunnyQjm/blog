---
layout:     post                    # 使用的布局（不需要改）
title:      Chapter 5 The Data Link Layer         # 标题
subtitle:                          #副标题
date:       2018-5-21 18:00              # 时间
author:     Ming.J                      # 作者
header-img: img/blog-header1.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - 计算机网络
    - 学习
    - 笔记
---

> ## Link Layer: Introduction and Services
> - 链路层实现相邻节点间的可靠数据传输
> - **nodes**(节点) => **主机/端系统** 和 **路由器**
> - **links**(链路) => 连接两相邻节点间的通信频道

- ### The Services Provided by the Link Layer（链路层所提供的服务）

  **链路层** 所提供的 **最基本的服务** 就是 **在两个相邻节点见得单个通信链路上转移datagram**
  - #### 成帧（***Framing***）
    - 在 **发送端** 将构造数据帧，将datagram放入数据域，加头加尾（尾部是用来检测及纠错的），**打包成帧**
    - 在 **接收端解开** 收到的 **数据帧**，将datagram传递给上层

  - #### 链路接入（***Link access***）
    - **MAC** (***Medium access control***, 介质访问控制) 协议详细规定了数据帧何时如何发送到链路上
    - MAC协议还用于当多个节点共享同一链路的时候，协调多个节点如何传递数据帧

  - #### 可靠数据传输（***Relilable delivery***）
    数据链路层可提供两相邻节点间的可靠数据传输（但不是必须的，大多数无线协议是不提供可靠数据传输的）

  - #### 流量控制（***flow control***）
    调节接收端发包的速率，防止接收端缓存溢出

  - #### 差错检测（***Error detection***）
    通过添加错误检测位，在接收端可以对收到的数据帧进行错误检测

  - #### 差错纠正（***Error correction***）

  - #### 半双工和全双工（***Half-duplex and full-duplex***）

- ### 链路层的实现
  {% img /img/computer_network/computer_network_66.png 网络适配器与其他主机组件以及网络协议栈的关系 %}
  数据链路层的实现是：硬件 + 软件

  - **大部分功能在硬件（网卡）中实现**：framing, link access, flow control, err dectection 等等
  - **部分功能由运行在CPU上的软件完成**：发送端从网络层接收datagram，以及接收端将解开frame得到的datagram传递给网络层等
  - **硬件实现的效率高，可以做纠错等比较复杂的操作而不太影响性能**
  - 现今很多人致力于将网络层甚至传输层的一部分工作，比如Checksum放到网卡中用硬件来实现，虽然这于网络协议栈分层的思想相悖，但是确实可以提高网络性能

> ## Error-Detection and Correction Techniques
> - 奇偶校验（***Parity checking***）
> - 校验和（***Checksum***）
> - 循环冗余校验（***Cyclic Redundancy Check***）

- ### 奇偶校验（***Parity checking***）
  - #### 奇校验和偶校验
    - 奇校验：数据位上1的个数个校验位上1的个数之和为奇数
    - 偶校验：数据位上1的个数个校验位上1的个数之和为偶数

  - #### One-bit even parity（单比特位偶校验）
    {% img /img/computer_network/computer_network_67.png 单比特位偶校验图示 %}
    - 存在的问题：当有偶数个bit为发送错误的时候，这种方式无法检测
    - **可以用来检错，但是准确率不高**

  - #### Two-dimensional even parity（二位偶校验）
    {% img /img/computer_network/computer_network_68.png 二位偶校验图示 %}
    - 即使偶数位比特位发送错误，也能检测出来
    - 通过二维校验位的定位，可以精确到哪一个bit位发生错误，进而改正这个错误（接收端如果能自行改正错误将降低网络的整体时延，提升网络的性能）
    - **可以检错，也可以纠错**

- ### 校验和（***Checksum***）
  通常发送端用校验和算法将让整个数据包的所有bit位参与计算，得到一个校验和（最简单的实现方式是将其看做一系列的数值，直接相加），在接收端再次计算这个校验和，并与收到的校验和逐bit比对

- ### 循环冗余校验（***Cyclic Redundancy Check***）
  {% img /img/computer_network/computer_network_69.png 循环冗余校验码格式 %}
  - #### 数据格式说明
    - **前d位为数据域**
    - **后r位为CRC码**

  - #### 生成多项式G
    - 生成多项式 **G** 共有 **r + 1位** （其中R位CRC码的长度）
    - 可能直接以 **二进制的形式** 给出
    - 也可能以 **多项式的形式** 给出（需要先转成二进制形式）

      Eg.: G = x<sup>4</sup> + x<sup>3</sup> + x  ==> G : 11010

  - #### 举个栗子
    {% img /img/computer_network/computer_network_70.png 计算CRC码的栗子 %}
    - 在data后面加上r位0用与计算
    - 其中里面每一步处理的时候执行的是 **异或运算**
    - 若最后的结果位数不够r为，则要在前面补上足够的0

> ## Multiple Access Protocols（多路访问协议）
> - 网络的两种链路类型
>   - point-to-point link (点对点链路) => 只有一个发送端和接收端
>   - brodcast link (广播链路) => 可能同时存在多个发送端和接收端在同一个链路上通信

- ### Introduction
  - #### 多路访问问题（***multiple access problem***）
    当在同一个通信链路上同时存在多个发送端和接收端的时候，他们之间要进行通信，由于同一时刻必然只有一个接收端和发送端，如何确保并协调这些节点的通信就是 **多路访问问题** => **多路访问协议**（***Multiple Access Protocol***）就是致力于解决这个问题

  - #### 碰撞的发生及处理
    在广播链路中，通常一个节点发送数据，该链路连接的所有节点都会收到一份copy，如果同一时刻有两个发送端在发送数据，则接收端无法解析这些数据，这就发送了 **碰撞**（***collide***） => 通常一旦碰撞发生，接收端就会选择丢弃这些包，这就相当于丢包发送了。这一段时间，两个发送端发送数据都不会被成功接收，链路并没有成功传输数据，**浪费了带宽资源**。而且通常不作处理的话，接入的节点越多，发生碰撞的几率越高，带宽就被大量的浪费

  - #### 多路访问协议分类
    - **信道分隔协议**（***channel partitioning protocols***）
    - **随机访问协议**（***random access protocols***）
    - **轮询协议**（***taking-turns protocols***）

- ### Channel Partitioning Protocols
  {% img /img/computer_network/computer_network_71.png 4个节点的TDM和FDM分隔信道图示 %}
  - 采用固定的资源分配方式，**资源独占独享**
  - **资源利用率低**
  - **不会发生冲突**
  - TDM 和 FDM

- ### Random Access Protocols
  > 使用随机访问协议时，一个 **正在传输的节点** 总是可以 **享用所有的带宽**。当 **发生碰撞** 时，所有 **参与碰撞的节点** 都会 **重传数据包**。但并不是立即重传，而是 **随机等待一段时间之后再重传**（***It waits a random delay before retransmitting th frame***）。而且每个参与碰撞的节点**产生时延都是独立且随机的**，这样就可以保证一部分节点可以错开其它节点，不发生冲突而成功传送数据包。

  - #### Sloted ALOHA （分时隙的ALOHA协议）
    - ##### 假设
      - 所有数据帧的长度都为L比特（***All frames consist of exactly L bits***）

      - 每个时隙的大小为L/R秒，也就是说一个在 **一个时隙内刚好可以传输一个数据帧** （***Time is divided int oslots of size L/R seconds, that is, a slot equals the time to transmit one frame***）

      - **节点只在时隙开始的时候传递数据包**（***Nodes start to transmit framges only at the beginings of slots***） => 也就是说在时隙内不会有其它未在传输的节点试图传递数据

      - 所有节点都是同步的，这样所有节点都能知道时隙什么时候开始（***The nodes are synchronized so that each node knows when the slots begin***）=> 前面的是书上的原话，同步和异步的定义在不同的场景下的定义可能不一样。这里所说的同步应该是对时间轴进行同步，所有实现的时候，只要时隙一开始，所有节点都能知道，并单独决定是否发送数据帧。

      - 一旦两个或更多的数据帧在同一时隙内发生了 **冲突**（***collide***），所有的节点都能在时隙结束前知道发生了冲突。（***If two or more frames collide in a slot, then all the nodes detect the collission event before the slot ends***）

    - ##### 每个节点所执行的操作
      - 当节点 **有一个新的数据帧要发送** 时，它会 **等待下一个时隙的开始**，并试图 **在下一个时隙内发送整个数据帧** （***When the node has a fresh frame to send, it waits until the begining of the next slot and transmit the entire frame in the slot.***）

      - 如果 **没有发送冲突** ，则节点 **认为数据包已成功传输且不用考虑重传**，如果还有数据帧未发的话，这个时候节点就可以准备发送下一个数据帧。（***If there isn't collision, the node has successfully transmited its frame and thus need not consider restransmitting the frame. The node can prepare a new frame for transimission, if it has one***）

      - 如果 **发生了冲突**，节点能够在时隙结束前获悉，**节点会在接下来的连续时隙内以概率p重传数据帧**，直到这个数据帧被成功发送且没有发送冲突。（***If there is a collision, the node detects the collision before the end of the slot. The node restransmits its frame in each subsequent slot with probability p until the frame is transimited without a collision***）

    - ##### 以概率p发送数据帧
      这是在发送冲突以后，节点会重传因为冲突而未传输成功的数据帧。但是为了避免下一时隙的大概率冲突，Slotted ALOHA协议选择让每个节点以概率p重传，也就是说**发送冲突的每个节点在下一时隙有概率p的可能重传数据包，有 (1 - p) 的可能选择等待**（值得注意的是，选择等待的节点在没有重传成功之前，都会在下一时隙开始时以概率p重传发送冲突的数据包） 。而且每个节点在未重传成功之前，都会每次以概率p发送，且是各自独立的，这也就实现了随机访问协议中 **当冲突发生的时候，每个节点会独立的随机等待一段时间** 的特性

    - ##### 举个栗子
      {% img /img/computer_network/computer_network_72.png Slotted ALOHA协议执行示例 %}

      上面假设三个节点各有一个数据包需要发送
      - 1时刻，3个节点都试图发送数据，发送冲突
      - 2时刻，3个节点都有需要重传的数据包，但是选择等待
      - 3时刻node3选择等待，但是另外连个节点选择发送，发送冲突
      - 4时刻只有node2选择发送，发送成功（至此，node2已经没有需要发送的数据帧）
      - 5时刻node1和node3都选择等待
      - 6时刻只有node1选择发送，发送成功（至此，node1已经没有需要发送的数据帧）
      - 7时刻，node3成功发送数据帧

      可以看出，发送3个数据帧，却用了7个时隙，浪费了带宽

    - ##### Slotted ALOHA 协议的优点
      - 单节点的瞬时带宽可达到最大带宽
      - 时隙的分配高度分散，只给需要传输数据的节点分配时隙（当系统中每个时隙只有一个节点需要发送数据的时候，该协议可以工作的很好）
      - 实现简单

    - ##### 缺点
      - 有可能发生碰撞进而浪费时隙（而且参与的节点越多，发生碰撞的可能性越大）
      - 需要严格的时钟同步
    - 当有 **大量的节点参与** 的时候，**时隙（带宽）的最大利用率** 大概只有 **1 / e ≈ 37%**

  - #### Aloha (pure ALOHA)
    {% img /img/computer_network/computer_network_73.png Pure ALOHA协议执行示例 %}

    纯的ALOHA协议没有时钟同步。当有一个数据帧要发送的时候就立即发送，当发生冲突的时候就立即以概率p重传，以概率（1  - p）等待传输一个数据帧所需要的时间。=> 冲突发生的概率更大了，当有 **大量的节点参与** 的时候，**带宽的最大利用率** 大概只有 **1 / 2e ≈ 18.5%**

  - #### Carrier Sense Multiple Access (CSMA, 载波侦听多路访问)
    - ##### 文明交流所必备的两个素质
      - ***Listen before speaking*** ：如果别人在啊讲话，则等别人说完再发表言论。在网络世界中，就是当你有一个数据帧需要发送的时候，而这个时候其它节点正在发送数据帧，则当前节点等待一段时间后再看是否还有节点在传输，仍有则继续等待，直到信道空闲的时候，发送数据。=> **载波侦听**（***carrier sensing***）

      - ***If someone else begins talking at the same time, stop talking*** ：如果在发送数据的过程中，发现其它节点也在发送数据，则停止发送数据。=> **冲突检测**（***collision dection***）

    - ##### CSMA的大致原理
      发送数据帧之前先侦听信道，如果信道为忙则等待一段时间后再尝试发送，如果为闲，则立即发送。

    - ##### CSMA仍然存在冲突
      当 **一个节点A发送数据到信道中，到另一个节点B可以侦听到节点A正在发送数据之间存在一段时延**，如果在这短时间内，节点B刚好有数据发送，那么由于它还没有检测到A正在发送数据，就会认为当前信道是空闲的，接着B就开始发送数据，这样冲突就出现了。

    - ##### 不侦听信道，冲突发生时不处理
      {% img /img/computer_network/computer_network_74.png 不侦听信道，冲突发生时不处理 %}

    - ##### 侦听信道，冲突发生时停止发送
      {% img /img/computer_network/computer_network_75.png 侦听信道，冲突发生时停止发送 %}

    - ##### Nonpersistent CSMA => 不持续侦听信道
      - 当节点有数据需要发送的时候，信道为闲则发
      - 信道为忙，则等待一段时间后在尝试发送

    - ##### 1-persistent CSMA
      - 信道为闲，则发
      - 信道为忙，则持续侦听信道，知道信道为闲，然后以概率1发数据（当两个节点同时侦听到信道闲，并都以概率1发数据，就发生了冲突）

    - ##### p-persistent CSMA
      - 信道为闲，则发
      - 信道为忙，则持续侦听信道，知道信道为闲，然后以概率p发数据

    - ##### CSMA with collision detection (CSMA/CD, 带冲突检测的载波侦听多路访问协议)
      - 要求节点必须全双工工作（发送数据包的同时也尝试接收数据包），**当检测到冲突时**（如果发送数据包的时候收到其他节点发送的数据包就表示发送了冲突），**立即停止发送数据包** => **可以把冲突带来损失降低**
      - 目前为止，所有的无线传输都是半双工的，故不能采用CSMA/CD

    - ##### CSMA with collistion avoid (CSMA/CA, 带冲突避免的载波侦听多路访问协议)
      因为无线传输时半双工的，无法再发包的同时检测冲突，故一旦冲突发生，发包也不会停止，故应该尽量避免冲突

  - #### Taking-Turns Protocols（轮流协议）
    - ##### polling protocol (轮询协议)
      主节点对每个节点进行轮询，只要当被询问时才可以发送数据
      - 优点：没有冲突，没有空时隙
      - 缺点：会有询问延时，需要主节点，会有单点故障

    - ##### token-passing protocol（令牌传递协议）
      一个特殊的帧被作为令牌，在节点之间按特定的顺序传递，只有持有令牌的节点才可以发送数据（且每次发送的数据有上限）
      - 优点：不需要主节点，没有冲突，没有空时隙
      - 缺点：可能有些节点中途宕机，导致令牌无法传递，或者持有令牌后宕机，导致令牌丢失等等
  - #### Local Area Networks (LANs, 本地局域网)
    {% img /img/computer_network/computer_network_76.png LAN %}

> ## Link-Layer Address （链路层寻址）

- ### MAC Address
  {% img /img/computer_network/computer_network_77.png 每个连接到LAN的网络适配器都有一个唯一的MAC地址 %}
  - MAC地址是 **数据链路层的地址**，同时也是数据链路层所采用的编址方式
  - **长度为48 bits**，常用12个16进制的数字表示（2个一组，6组）
  - 每个网络适配器上有一个网路地址，网卡出厂的时候的MAC地址是全球唯一的（是由IEEE统一管理的），但是现在可以用软件修改网络适配器的MAC地址，这里仍然假设它是唯一且不变的。

- ### Address Resolution Protocol (ARP, 地址解析协议)
  > ARP 协议实现的是IP地址到MAC地址的映射

  {% img /img/computer_network/computer_network_78.png 每个节点都有网络层地址和链路层地址 %}

  - 当 **一个节点发送一个frame** 时，实际上所有在 **同一个子网内的节点都会收到** 一个copy，只是在链路层做了一层判断，若 **该frame的目的MAC地址与本节点的MAC地址匹配则解包，传递给上层，否则直接丢弃**。

  - **每一个** 藉由网络适配器在局域网内通信的 **节点都会在内存中维护一张 ARP table**（地址映射表），保存了 **当前局域网内所有已知主机的IP地址与MAC地址的映关系**。（且映射表中的每一个表项都有一个TTL，一旦过期便会移除该表项）

  - 当欲发送的frame的目标IP在本地ARP table中 **找不到对应的映射时** ，节点 **发送一个APR query package**，目的MAC地址为全1，作为广播，只有与目的IP匹配的设备会返回一个正常的数据帧，节点再记录下IP地址与返回消息中包含的目标节点的MAC地址的映射。
    {% img /img/computer_network/computer_network_79.png  ARP table %}

  - **ARP协议是即插即用的**

  - **MAC地址只在本局域网内有效**
    {% img /img/computer_network/computer_network_80.png 一个路由器连接两个子网 %}
    => **数据帧在穿越路由器的时候链路层首部的源MAC地址和目的MAC地址都要修改**

> ## Ethernet (以太网)
> - 它是互联网中有线网络通信的事实标准
> - 所采用的MAC (***Multiple Access Control***, 多路访问控制) 协议为 **1-persistent CSMA/CD** (***Carrier Sense Multiple Access with Collide Detection***, 带冲突检测的载波侦听多路访问协议)
> - 以太网相较于其他有线局域网最大的优势：**实现简单，价格便宜**

- ### Ethernet Frame Structure (以太网帧结构)
  {% img  /img/computer_network/computer_network_81.png 以太网帧结构2%}
  - **Preamble** (***8 bytes***) : 前序/序文，不算在以太网帧长度里面
      - **Dest. address** (***6 bytes***) : 目的端MAC地址节点A向B发一个数据包，在数据包到达B之前，B恰好要想A发一个数据包，此时因未收到A发的包，故认为信道空闲，B向A发一个包。在B收到A的包后，意识到发送了冲突，停止发送数据包。****
  - **Source address** (***6 bytes***) : 源端MAC地址
  - **Type** : 标识使用的是哪个网络层协议
  - **Data** : 数据域
  - **CRC** : 冗余校验码

  => **以太网frame的长度范围：64 bytes ~ 1518 bytes** (其中最大是因为协议规定了MTU，需要有最小长度的原因稍后会解释)

- ### CSMA/CD: Thernet's Multiple Access Protocol
  - 信道空闲时，立即发送，若信道为忙，则持续侦听信道 => **持续侦听**
  - 信道侦听过程中，一旦信道再次空闲，则以概率1发送数据包 => **1-persistent**
  - 在边发送数据的同时，边接收数据 => **发送数据过程中如果收到数据包则表示发送了冲突**
  - 一旦**发生冲突**，立即**停止发送数据包**，并**广播一个信号**（告知其它用户发送冲突了），同时**等待一段时间**（具体等待的时间由 **“二进制退避算法”** 求得）

- ### * 二进制退避算法
  {0， 1， 2， ···， 2<sup>m</sup> - 1}
  - 其中m是发生冲突的次数
  - 随机选择一个元素 x 512 bits times
  - **bit times**: 在当前网络环境下传输1 bit所需要的时间
  - **冲突10次以后集合不在扩大**，即集合最大为1024个元素。**冲突16次以后直接丢弃该包**，不再传输

- ### 为什么以太网帧的长度最小为64 bits？
  - #### 数据帧长度不够长所引发的问题
    {% img /img/computer_network/computer_network_82.png 数据帧长度不够长所引发的问题 %}
    节点A向B发一个数据包，在数据包到达B之前，B恰好要想A发一个数据包，此时因未收到A发的包，故认为信道空闲，B向A发一个包。在B收到A的包后，意识到发送了冲突，停止发送数据包。**如果A在B发的包到达A之前就已经将数据包发送完毕了，即A发的数据包不够长的情况下，A可能以为数据包发送成功了吗，而实际上是发送冲突了**

  - #### 为什么是64个字节
    {% img /img/computer_network/computer_network_83.png 为什么是64个字节 %}

- ### CSMA/CD efficiency
  {% img /img/computer_network/computer_network_84.png CSMA/CD efficiency %}
  - 当d<sub>prop</sub> -> 0，或d<sub>trans</sub> -> ∞，效率 -> 1
  - 数据包长度越长，节点间距离越短，效率越高
  - 节点数据↑，发送冲突的可能性↑，效率↓

- ### 不同的以太网标准
  {% img /img/computer_network/computer_network_85.png 不同的以太网标准 %}
  - T开头的都是双绞线
  - 所有非T开头的都是光纤
  - 同轴电缆的格式：10Base2

=> **以太网负载在30%以内的时候性能很好，基本不丢包，但超过30%就会发生一系列的问题**

> ## Link-layer Switches (链路层交换机)

- ### 几种常见网络设备区别
  - **中继器**（***repeter***）：物理层设备 => **将信号放大后转发**
  - **集线器**（***hubs***）：物理层设备 => **不能隔离冲突域**（集线器是一种特殊的中继器）
  - **网桥**（***Bridge***）：二层网络设备 => **可隔离冲突域**
  - **交换机**（***Switches***）：二层网络设备 => **可隔离冲突域**
  - **路由器**（***Router***）：3层网络设备 => **可隔离广播域**

  => 网桥和交换机的功能基本一致，不过 **网桥是二段口的，而交换机是多端口** 的，同时 **交换机还兼备“自学习”** 等功能

- ### 冲突域和广播域
  - **冲突域**：在同一个冲突域的两个设备同时通信会导致冲突
  - **广播域**：在同一个广播域中的设备可以互相广播

- ### 交换机（Switches）
  - 是 **链路层设备**，比Hub更智能，其重要作用 => **存储转发以太网帧**
  - **对主机透明**
  - **即插即用**
  - **自学习**
  - 允许多路同步传输 => **交换机隔离了冲突域**

- ### Switch table and self-learn
  > 交换机通过自学习填充交换表

  - #### self-learn（自学习）
    交换机通过自学习可以在 Switch table中记录下从哪个端口可以到达哪个主机（下面建立交换表的过程中会详细讲述自学习）

  - #### Switch table
    {% img /img/computer_network/computer_network_86.png 交换表 %}
    - 交换表初始时为空

    - 当收到一个frame时，记录下发送端的MAC地址以及从哪个端口进来的，并去查询交换表中有无到目的主机MAC地址对应的记录，若有则直接转发到对应端口，若无，执行下一步

    - 若交换表中无对应的记录，则向所有的端口广播一个消息，目的MAC地址设置为目的主机，目的主机收到之后会回发一个数据包，交换机记录下其MAC地址以及从哪个端口可达。

- **交换机可以将一个子网分隔成多个局域网**

- ### hubs, routers, switches的比较
  {% img /img/computer_network/computer_network_87.png hubs, routers, switches的比较 %}
