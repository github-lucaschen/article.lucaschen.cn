---
title: 'Network and Load Balance'
description: '网络和负载均衡'
keywords: 'network,load balance'

date: 2024-03-17T12:50:00+08:00

categories:
  - Doc
tags:
  - Network
  - LoadBalance
---

记录不同的网络分层，和负载均衡。

<!--more-->

## OSI model

<table class="wikitable" style="margin: 1em auto 1em auto;font-size: 14px;">
  <caption><b>OSI model</b></caption>
  <tbody>
    <tr>
      <th colspan="3">Layer</th>
      <th>Protocol data unit (PDU)</th>
      <th>Function</sup></th>
    </tr>
    <tr>
      <th rowspan="4">Host
        <br>layers</th>
      <td style="background:#d8ec9b;">7</td>
      <td style="background:#d8ec9b;">Application</td>
      <td style="background:#d8ec9c;" rowspan="3">Data</td>
      <td style="background:#d8ec9c;">High-level protocols such as for resource sharing or remote file access, e.g. HTTP.</td></tr>
    <tr>
      <td style="background:#d8ec9b;">6</td>
      <td style="background:#d8ec9b;">Presentation</td>
      <td style="background:#d8ec9b;">Translation of data between a networking service and an application; including character encoding, data compression and encryption/decryption</td></tr>
    <tr>
      <td style="background:#d8ec9b;">5</td>
      <td style="background:#d8ec9b;">Session</td>
      <td style="background:#d8ec9b;">Managing communication sessions, i.e., continuous exchange of information in the form of multiple back-and-forth transmissions between two nodes</td></tr>
    <tr>
      <td style="background:#e7ed9c;">4</td>
      <td style="background:#e7ed9c;">Transport</td>
      <td style="background:#e7ed9c;">Segment, Datagram</td>
      <td style="background:#e7ed9c;">Reliable transmission of data segments between points on a network, including segmentation, acknowledgement and multiplexing</td></tr>
    <tr>
      <th rowspan="3">Media
        <br>layers</th>
      <td style="background:#eddc9c;">3</td>
      <td style="background:#eddc9c;">Network</td>
      <td style="background:#eddc9c;">Packet</td>
      <td style="background:#eddc9c;">Structuring and managing a multi-node network, including addressing, routing and traffic control</td></tr>
    <tr>
      <td style="background:#e9c189;">2</td>
      <td style="background:#e9c189;">Data link</td>
      <td style="background:#e9c189;">Frame</td>
      <td style="background:#e9c189;">Transmission of data frames between two nodes connected by a physical layer</td></tr>
    <tr>
      <td style="background:#e9988a;">1</td>
      <td style="background:#e9988a;">Physical</td>
      <td style="background:#e9988a;">Bit, Symbol</td>
      <td style="background:#e9988a;">Transmission and reception of raw bit streams over a physical medium</td></tr>
  </tbody>
</table>

### Layer 1: Physical layer

> 物理层：通过物理介质传输和接收原始比特流

物理层（Physical Layer）在局域网上传送数据帧（Data Frame），它负责管理电脑通信设备和网络媒体之间的互通。包括了针脚、电压、线缆规范、集线器、中继器、网卡、主机接口卡、路由器等。

### Layer 2: Data link layer

> 数据链路层：通过物理层连接的两个节点之间的数据帧传输

数据链路层（Data Link Layer）负责网络寻址、错误侦测和改错。当表头和表尾被加至数据包时，会形成信息框（Info Box）。数据链表头（DLH）是包含了物理地址和错误侦测及改错的方法。数据链表尾（DLT）是一串指示数据包末端的字符串。例如以太网、无线局域网（Wi-Fi）和通用分组无线服务（GPRS）等。

分为两个子层：逻辑链路控制（logical link control，LLC）子层和介质访问控制（Media access control，MAC）子层。

### Layer 3: Network layer

> 网络层：构建和管理多节点网络，包括寻址、路由和流量控制

网络层（Network Layer）决定数据的路径选择和转寄，将网络表头（NH）加至数据包，以形成分组。网络表头包含了网络资料。例如:互联网协议（IP）等。

### Layer 4: Transport layer

> 传输层：网络上各点之间数据段的可靠传输，包括分段、确认和复用

传输层（Transport Layer）把传输表头（TH）加至资料以形成分组。传输表头包含了所使用的协议等发送信息。例如:传输控制协议（TCP）等。

### Layer 5: Session layer

> 会话层：管理通信会话，即两个节点之间以多次来回传输的形式持续交换信息

会话层（Session Layer）负责在数据传输中设置和维护计算机网络中两台计算机之间的通信连接。

### Layer 6: Presentation layer

> 表现层：网络服务和应用程序之间的数据转换； 包括字符编码、数据压缩和加解密

表现层（Presentation Layer）把数据转换为能与接收者的系统格式兼容并适合传输的格式。

### Layer 7: Application layer

> 应用层：高级协议，例如资源共享或远程文件访问，例如 HTTP

应用层（Application Layer）是用户真正与电脑沟通的点，这一层只有在真正需要访问网络才有作用。以网络浏览器(IE，Chrome，Explorer)为例，您可以从系统中移除诸如 TCP/IP，NIC 卡等网络组件，但仍旧可以使用 IE 来查看原生的 HTML 文件。不过如果您想要查看那些必须使用 HTTP 来截取远程 HTML 文件，或是利用 FTP 或 TFTP 抓取文件时，事情就会一片混乱。这是因为 IE 或其他浏览器在回应这类请求时，会试图访问[应用层]。基本上，[应用层]会扮演真正的应用程序与下一层之间的接口，让应用程序能够穿越协议堆栈向下发送信息。这其实不算是分层式结构的一部分，因为浏览器不是真的位于[应用层]内，但它会在必须处理远程资源时，与[应用层]即其相关协议介接。 [应用层]还要负责识别及确认可用的通信伙伴，做好通信准备，并验证指定通信类型是否能得到足够的资源。这些任务非常重要，因为电脑应用有时需要的不只是桌面上的资源。通常会将几个网络应用的通信组件结合起来，例如:•网络浏览•文件传输•电子邮件•开启远程访问•网络管理活动•客户端/服务器处理•信息搜索。

## TCP/IP model

<table class="wikitable" style="margin: 1em auto 1em auto;font-size: 14px;">
  <caption><b>TCP/IP model</b></caption>
  <tbody>
    <tr>
      <th colspan="3">Layer</th>
      <th>Protocol</th>
    </tr>
    <tr>
      <th rowspan="2">Host
        <br>layers</th>
      <td style="background:#d8ec9b;">4</td>
      <td style="background:#d8ec9b;">Application</td>
      <td style="background:#d8ec9c;">HTTP、FTP、DNS</td></tr>
    <tr>
      <td style="background:#e7ed9c;">3</td>
      <td style="background:#e7ed9c;">Transport</td>
      <td style="background:#e7ed9c;">TCP、UDP、RTP、SCTP</td></tr>
    <tr>
      <th rowspan="3">Media
        <br>layers</th>
      <td style="background:#eddc9c;">2</td>
      <td style="background:#eddc9c;">Internet</td>
      <td style="background:#eddc9c;">IP</td>
    <tr>
      <td style="background:#e9c189;">1</td>
      <td style="background:#e9c189;">Network Access(link)</td>
      <td style="background:#e9c189;">Wi-Fi、MPLS</td></tr>
  </tbody>
</table>

数据封装情况：

![UDP_encapsulation](/imgs/posts/network-and-load-balance/UDP_encapsulation.svg.png)

## OSI & TCP/IP

<table class="wikitable" style="margin: 1em auto 1em auto;font-size: 14px;">
  <caption><b>OSI model</b></caption>
  <tbody>
    <tr>
      <th>Layer</th>
      <th colspan="2">OSI</th>
      <th colspan="2">TCP/IP</th>
    </tr>
    <tr>
      <th rowspan="4">Host
        <br>layers</th>
      <td style="background:#d8ec9b;">7</td>
      <td style="background:#d8ec9b;">Application</td>
      <td style="background:#d8ec9c;" rowspan="3">4</td>
      <td style="background:#d8ec9c;" rowspan="3">Application</td></tr>
    <tr>
      <td style="background:#d8ec9b;">6</td>
      <td style="background:#d8ec9b;">Presentation</td></tr>
    <tr>
      <td style="background:#d8ec9b;">5</td>
      <td style="background:#d8ec9b;">Session</td></tr>
    <tr>
      <td style="background:#e7ed9c;">4</td>
      <td style="background:#e7ed9c;">Transport</td>
      <td style="background:#e7ed9c;">3</td>
      <td style="background:#e7ed9c;">Transport</td></tr>
    <tr>
      <th rowspan="3">Media
        <br>layers</th>
      <td style="background:#eddc9c;">3</td>
      <td style="background:#eddc9c;">Network</td>
      <td style="background:#eddc9c;">2</td>
      <td style="background:#eddc9c;">Internet</td></tr>
    <tr>
      <td style="background:#e9c189;">2</td>
      <td style="background:#e9c189;">Data link</td>
      <td style="background:#e9c189;" rowspan="3">1</td>
      <td style="background:#e9c189;" rowspan="3">Network Access(link)</td></tr>
    <tr>
      <td style="background:#e9988a;">1</td>
      <td style="background:#e9988a;">Physical</td>
  </tbody>
</table>

## Load Balance

负载平衡（英语：load balancing）是一种电子计算机技术，用来在多个计算机（计算机集群）、网络连接、CPU、磁盘驱动器或其他资源中分配负载，以达到优化资源使用、最大化吞吐率、最小化响应时间、同时避免过载的目的。 使用带有负载平衡的多个服务器组件，取代单一的组件，可以通过冗余提高可靠性。负载平衡服务通常是由专用软件和硬件来完成。 主要作用是将大量作业合理地分摊到多个操作单元上进行执行，用于解决互联网架构中的高并发和高可用的问题。

### L4 Switch

L4 是传输层(Transport Layer)，此层级使用的协议是 TCP/UDP。使用 L4 的 load balancer 时，L4 不会对包进行任何处理，也不会与 client 端建立 TCP 连接，只做包转发，到后端真正的处理器中，由被转发的机器直接与 client 建立连接。

### L7 Switch

L7 是应用层(Application Layer)，此层使用 HTTP、telnet、FTP 等协议。使用 L7 的 load balance 时，L7 会去解析封包，并与 client 建立连接，在根据设置跟被代理的服务器建立 TCP 连接，此时的数据包与原本的 client 传过来是不一样的，同样的，收到返回的包时，也是要先解析再回传给 client。此处不能直接传输的原因是两者是不同的 TCP 连接。

### L4 & L7

|          | L4 Switch                | L7 Switch                                         |
| -------- | ------------------------ | ------------------------------------------------- |
| 基于     | IP + PORT                | 虚拟的 URL 或主机 IP                              |
| 类似于   | 路由器                   | 代理服务器                                        |
| 握手次数 | 1                        | 2                                                 |
| 复杂度   | 低                       | 高                                                |
| 性能     | 高；无需解析内容         | 中；需要算法识别 URL,Cookie 和 HTTP Header 等信息 |
| 安全性   | 低；无法识别 DDoS 等攻击 | 高；可以防御 SYN Cookie 和 SYN Flood 等           |
| 额外功能 | 无                       | 会话保持，图片压缩，防盗链                        |
