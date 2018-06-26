---
layout:     post                    # 使用的布局（不需要改）
title:      Chapter 1 Computer Network and the Internet         # 标题
subtitle:                          #副标题
date:       2018-5-5 18:00              # 时间
author:     Ming.J                      # 作者
header-img: img/blog-header1.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - 计算机网络
    - 学习
    - 笔记
---

> ## What is the Internet?
> - There are couple of ways to answer this question. First, we can describe the nuts and bolts of the Internet, that is, the basic hardware and software components that make up the Internet. Second, we can describe the Internet in terms of a networking infrastructure that provides services to distributed applications
> - 对于什么是因特网，可以从两方面来描述。首先，我们可以描述它的基本原理，即 **构成Internet的基本硬件和软件组件** 。其次，我们可以将其描述为 **一个网络基础设施，它为分布式应用程序提供服务**

- ### A Nuts-and-Bolts description
  > 数以百万计的计算机设备——**主机/终端（host/end systems）** 通过 **通信链路（communication links）** 和 **分组交换机（packet switches）** 相互连接在一起。

  - #### 主机/终端（host/end systems）
    广义所指，除了传统的PC、Linux工作站等还包括手机、TV等一众可以连接到互联网的设备
  - #### 通信链路（communication links）
    通信链路有多种类型，它们基于不同的物理介质，如：同轴电缆（coaxial cable），铜线（copper wire），光纤（fiber optics）等，采用不同的材质，会影响链路的 **传输速率（transmission rate）**

    - 传输速率（transmission rate）
      - transmission rate = bandwidth（传输速率=带宽）==> **发送端发包的速率**
      - ps: 吞吐量 ==> **接收端收包的速率**
  - #### 分组交换机（packet switches）
    A packet switch takes a packet arriving on one of its incoming communication links and forwards the packet on one of its outgoing communication links. (分组交换机的功能就是接收一个链路上发来的数据包，并将其转发给它的另一个出口链路上)目前Internet上的分组交换机主要有两种：**路由器（routers）** 和 **链路层交换机（link-layer switches）**

    - 路由器（routers） ==> 常在网络核心（network core）中使用
    - 链路层交换机（link-layer switches） ==> 常在接入网（access network）中使用
  - #### 路由（route / path）
    The sequence of communication links and packet switches traversed by a oacket from the sending end system and to the receiving end system is know as **route** or **path** (数据包从发送端到接收端所途经的通信链路和分组交换机组成所谓的路由) ==> **一个路由其实就是网络上的一条路径**，某些数据包通过这条路径从发送端传输到接收端
  - #### 互联网服务提供商（Internet Serveice Providers，ISPs）
    互联网服务提供商，即向广大用户综合提供互联网接入业务、信息业务、和增值业务的电信运营商
- ### A Sercices description
  > The Internet is an infrastructure for providing services to distributed applications ==> **Internet一个为分布式应用提供服务的网络基础设置**

  对于一个分布式的网络应用程序，程序之间要进行通信就需要一个传输媒介，而Internet就很好的扮演了这个角色。


- ### What is Protocol?
  A **protocol** defines the format and the order of messages exchanged between two or more communicating entities, as well as the actions taken on the transmission and/or receipt of a message or other event.

  一个协议定义了两个通信实体之间交换信息的格式和顺序，以及在消息和其它事件的发送或接收时所采取的响应动作

- ### The topology structure (拓扑结构)
  - #### 集中式
    单点故障；结构，管理简单；中心节点价格昂贵，维护困难；
    - Star => 星型
    - Tree => 树型
  - ### 分布式
    - Bus => 总线型
    - Mesh => 网状网 （适用于无线通信）
    - Ring => 环形

- ### Network structure(网络结构)
  - Network edge    （网络边缘）
  - Access network  （接入网）
  - Physical media  （物理介质）
  - Network core    （网络核心）

> ## The Network Edge (网络边缘)
> - end systems     （终端）
> - access network  （接入网）
> - links           （链接）

- ### Client and Server Programs (C / S)
  - C/S

    A **client program** is a program running on one end system that requests and receives a service from a **server program** running on another end system.

    客户端程序运行在一个终端上，并且向运行在另一个终端上的服务端程序请求并获取服务。

  - P2P

    同一个终端既可以作为客户端请求服务，也可以作为服务端程序提供服务

- ### Access network （接入网）

  => 主机（终端）与边缘路由器相连的那段网络（Physical link）称之为接入网

  => **接入网很大程度上决定了用户的所能真正享有的带宽**

- ### Physical Media （物理媒介）
  > - 传输的是Bits（一系列的比特流）；
  > - guided media => 导向型（有线）
  > - unduided media => 无导向型（无线）

  - Twisted Pair(TP) => 双绞线
  - Coaxial Cable => 同轴电缆
  - Fiber Optics => 光纤（ ***low bit error rate (比特差错率低)*** ）
  - Terrestrial Radio Channels => 地面无线信道
  - Satellite Radio Channels => 卫星无线信道

> ## The Network Core (网络核心)
> => 分组交换机以及连接分组交换机间的链路

电路交换网络和分组交换网络可以类比为饭馆订餐：电路交换网络就好比一个饭店，需要提前预定座位，并且饭店会在你预定之后再规定时间内一直为你保留预定的座位，你在既定时段的任何一个时间点去都是有位置的，期间有未预定的顾客进入，即便饭店有空位，也无法入座；而分组交换网络则好比不需要预定的饭店，先到先得，如果饭店满座了就需要等待。

- ### Circuit Switching (电路交换网络)
  > 典型的就是电话网络

  - **Dedicated allocation** => 固定分配，独占独享
  - **Resource reservation** => 资源预留（不共享，不允许别的连接使用）
  - **Multiplexing** => 多路复用(将网络资源切片)
    - frequency-division multiplexing => **频分多路复用** (FDM)
    - time-division multiplexing => **时分多路复用** (TDM)
    {% img  /img/computer_network/computer_network_03.png FDM和TDM图解 %}
  - 需要呼叫建立
  - 时延小，无资源竞争

  - 由于电路交换网络给每个Connection分配固定的带宽，分配的连接越多，意味着每个Conncetion所能享有的带宽就越少

- ### Packet Switching (分组交换网络)
  > 典型的，Internet网络采用的就是分组交换网络

  - Share network resources => 各个分组 **共享网络资源**
  - each packet uses full link bandwidth => **每个分组均能使用全部的链路带宽**
  - resources used as needed => **资源按需分配**
  - resource contention => **存在资源竞争** => 可能会导致丢包，拥塞，延时等一系列问题

  - Statistical Multiplexing => **统计复用** （异步时分多路复用）

    - 传统的 **TDM**（时分多路复用）是给每一个终端分配一个 **time-slot**（时隙），对于某个具体的终端，不管这个终端有没有在进行通信，分配给它的那部分带宽都将被占用，是同步的

    - 而 **统计复用**（异步时分多路复用）是把公共信道的时隙实行“ **按需分配** ”，即只对那些需要传送信息或正在工作的终端分配time-slot，这样就能使所有的时隙都能被充分的利用，可以使服务的终端数大于一个周期内时隙划分的个数，提高了媒质的利用率，从而起到了复用的作用。
  - **Store-and-forward** （存储转发）

    => 以太网交换机的控制器先将输入端口到来的数据包缓存起来，先检查数据包是否正确，并过滤掉冲突包错误 （增加了网络的时延，但是使得网络有了一定的检错功能）
  - 与电路交换相比
    - Packet switching allows more users to use network( **分组交换允许更多的用户使用网路** )
    - Great forbursty data ( **处理突发数据极为有效** )

- ### 两种网络的应用场景
  - 在 **带宽需求相对固定** 的情况下， **电路交换网络** 比较合适
    - 资源需求稳定
    - 固定电话的通信就是，语音要求的带宽非常固定，而且采用电路交换网络也可以保证通话的质量
  - 在 **带宽需求动态变化** 时，**分组交换网络** 比较适用
    - 典型的Internet网络的带宽需求就是动态变化的

> ## Delay, loss and throughputin Packet-Switched Neworks
> 在分组交换网络中的延迟，丢包和吞吐量

- ### Delay（延迟）
  {% img  /img/computer_network/computer_network_01.png 路由器A的节点传输延迟 %}
  - **Nodal procesing delay** => **节点处理延迟**（一般是很短的）
    - 差错检测 （check bit error）
    - 决定从哪个口转发出去 (determine output link)

  - **Queueing delay** => 排队延迟（可以为0，也可以很长，取决于当前网络的状态）
  - **Transimission delay** => 传输延迟（可以很小，也可以很长，取决于数据包的长度以及网络的带宽）
    {% img  /img/computer_network/computer_network_02.png 传输延迟计算方法 %}
    => 所以 **提高网络带宽只能降低传输延迟**
  - **Propagation delay** => 传播延迟 （延迟的大小由物理介质的长度决定，因为传播的速度很快，所以一般这个延迟也不会很大）
    {% img  /img/computer_network/computer_network_04.png 传播延迟计算方法 %}
    - 其中传播的具体速度的大小取决于是用那种物理介质传输的（光纤，双绞线等等）
    - 取值范围：2\*10<sup>8</sup> m/s ~ 3\*10<sup>8</sup> m/s (最快接近光速)
    - **与数据包的长度无关**

  > !!! ** d<sub>nodal</sub> = d<sub>proc</sub> + d<sub>queue</sub> + d<sub>trans</sub> + d<sub>prop</sub>**
  > (数据包在一个节点上的时延 = 节点处理延迟 + 排队延迟 + 传输延迟 + 传播延迟)

- ### Loss（丢包）
  当分组交换机（链路层交换机和路由器）有限缓存溢出的时候就会发生丢包
  - 可能会由前一个节点或源端节点重传，或者压根不重传
  - 在端系统的视角看来，就是一个数据包已经被发送到网络核心，当时却没有到达目的节点
- ### Throughput（吞吐量）
  > [百度百科](https://baike.baidu.com/item/%E5%90%9E%E5%90%90%E9%87%8F/157092)： 吞吐量是指对网络、设备、端口、虚电路或其他设施，单位时间内成功地传送数据的数量（以比特、字节、分组等测量）
  >- instantaneous throughput (瞬时吞吐量)
  >- average throughput (平均吞吐量)

  - 在Internet网络中端到端的**吞吐量** 通常和 **接收端收包的速率** 是一致的
  - 在Internet网络中的端到端吞吐量取决于网络中的**瓶颈链路（Bottleneck link）**
    {% img  /img/computer_network/computer_network_05.png 吞吐量受瓶颈链路的影响 %}

- ### 补充
  - 所有网络信号在物理介质上的速度近似为光速，可以认为是一样的 => 即 **网络信号在传输时并无快慢之分**
  - 之所以在 **不同的链路上有带宽差别**，是因为由于协议，技术等原因导致 **两个相邻的数据包之间存在不同大小的时隙**
  - 网络中一条链路上数据的传输永远是串行的（并不会出现类似“多车道”的情况）
  - Quality of service, Qos => 服务质量

> ## Protocol Layers and Their Service Models（协议分层以及他们的服务模型）
> 计算机网路的体系结构（architecture）是计算机网络各层及其协议的集合

  {% img  /img/computer_network/computer_network_06.png 两种网络协议分层模型 %}

- ### PDU (Protocol Data Unit, 网络协议数据单元)
  同一层的所有协议所处理的数据单元
- ### Five-layer Internet protocol stack (五层网络协议栈、TCP/IP协议栈)
  {% img  /img/computer_network/computer_network_01.jpg 五层网络协议栈模型 %}
  - #### 各层提供的功能
    - Applicaiton（应用层）: 仅为用户提供接口
    - Transport（传输层）：实现进程到进程间的通信
    - Network（网络层）：实现主机到主机之间的通信
    - Link（数据链路层）：实现相邻节点间的数据传输
    - Physical（物理层）：在物理介质上传输比特流
  - #### 各层的PDU
    - Applicaiton（应用层）: message（报文 / 消息）
    - Transport（传输层）：segment（报文段）
    - Network（网络层）：datagram（数据报）
    - Link（数据链路层）：frame（数据帧）
    - Physical（物理层）：bit（比特）
- **协议** 是对等实体之间的， **服务** 则是由下层通过向上层提供接口提供的
- ### Seven-layer ISO/OSI reference model
  {% img  /img/computer_network/computer_network_07.png OSI/ISO 七层网络协议栈模型 %}
