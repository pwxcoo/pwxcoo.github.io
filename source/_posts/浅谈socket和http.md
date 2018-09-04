---
title: 浅谈socket和http
date: 2018-06-05 22:24:24
categories: base
tags:
- socket
- network programming
- computer network
---

今天试了 socket 和网络编程相关的东西。计算机网络是上个学期的课，忘了好多的说。。太久不用果然是学一点新的东西，旧的都忘了。。还有一部分原因也是自己当初学得仅仅停留在老师的期末考范围里，考纲以外的东西基本没做太多了解。。网络的知识点太多了。。

正好趁这次机会好好看了一下 socket 和网络编程相关的一些东西，跟之前比起来有了很多新的体会和认识。


## 计算机网络协议分层
物，数，网，传，会，表，应！

### TCP/IP
- 数据链路层：ARP，RARP
- 网络层：IP，ICMP，IGMP
- 传输层：TCP，UDP，UGP
- 应用层：Telnet，FTP，SMTP，SNMP

### OSI
- 物理层：EIA/TIA-232, EIA/TIA-499, V.35, V.24, RJ45, Ethernet, 802.3, 802.5, FDDI, NRZI, NRZ, B8ZS 
- 数据链路层：Frame Relay, HDLC, PPP, IEEE 802.3/802.2, FDDI, ATM,  IEEE 802.5/802.2 
- 网络层：IP，IPX，AppleTalk DDP 
- 传输层：TCP，UDP，SPX 
- 会话层：RPC,SQL,NFS,NetBIOS,names,AppleTalk,ASP,DECnet,SCP 
- 表示层:TIFF,GIF,JPEG,PICT,ASCII,EBCDIC,encryption,MPEG,MIDI,HTML 
- 应用层：FTP,WWW,Telnet,NFS,SMTP,Gateway,SNMP 

#### 应用层 
1. 主要功能 ：用户接口、应用程序 
application 
2. 典型设备：网关 
3. 典型协议、标准和应用：TELNET, FTP, HTTP 

#### 表示层 
1. 主要功能 ：数据的表示、压缩和加密 
presentation
2. 典型设备：网关 
3. 典型协议、标准和应用：ASCLL、PICT、TIFF、JPEG、 MIDI、MPEG 

#### 会话层 
1. 主要功能 ：会话的建立和结束 
session
2. 典型设备：网关 
3. 典型协议、标准和应用：RPC、SQL、NFS 、X WINDOWS、ASP 

#### 传输层 
1. 主要功能 ：端到端控制 
transport 
2. 典型设备：网关 
3. 典型协议、标准和应用：TCP、UDP、SPX 

#### 网络层 
1. 主要功能 ：路由，寻址 
network
2. 典型设备：路由器 
3. 典型协议、标准和应用：IP、IPX、APPLETALK、ICMP 

#### 数据链路层 
1. 主要功能 ：保证误差错的数据链路 
data link 
2. 典型设备：交换机、网桥、网卡 
3. 典型协议、标准和应用：802.2、802.3ATM、HDLC、FRAME RELAY 

#### 物理层 
1. 主要功能 ：传输比特流 
physical
2. 典型设备：集线器、中继器 
3. 典型协议、标准和应用：V.35、EIA/TIA-232 

## Socket, HTTP和WebSocket区别
### HTTP
HTTP 是短连接，无状态的。是应用层的协议，用的 80 端口。跟 Socket 相比较：
- 传输速度慢，数据包大（更上层的协议当然需要更多首部和一些杂七杂八的信息）
- 安全性差
适用于**对传输速度，安全性要求不是很高，且需要快速开发的应用**。

当然好处也明显，直观，标准，方便**异构系统的交互**。

### Socket
Socket 的意思是指端到端的通信。操作系统中也有 Socket 的概念，指的是文件系统间的通信。网络协议中的 Socket 的指的是端到端的进程之间的通信。

Socket 其实是对 TCP 和 UDP 的封装一层抽象接口，而非协议，所以是在传输层，也就是 HTTP（应用层）的下面一层，是比 HTTP 更底层的协议。

Socket 有两种类型：
- 如果封装的是 TCP，就是流式 Socket，面向连接的 TCP 服务应用，安全，效率低
- 如果封装的是 UDP，就是数据报式 Socket，应用于无连接的 UDP 服务应用，不安全，但效率高（PS: UDP 貌似都是应用于实时通信这些地方，貌似 QQ 的文件传输用的也是 UDP，怪不得有时候总是出错。。）

### WebSocket
如果要问 WebSocket 和 Socket 有什么关系的，就像卡巴斯基和巴基斯坦的关系 —— 有基巴关系。。貌似当初命名者跟那个 JavaScript 蹭 Java 热点一点，蹭了一波 Socket 的热点。。

说是没什么关系，但是其实还是有一点联系，可以理解为 WebSocket 是 Socket 的上层实现，WebSocket 是有当初设计 Socket 的通信理念在里头的，Socket 是传输层的 API，而 WebSocket 是应用层的协议。

WebSocket 跟 HTTP 一样，是应用层的协议，可以实现浏览器与服务器的全双工通信，我之前做过一个网页的社交聊天的应用，用的就是 WebSocket 技术，具体在这里就不展开讨论了。很多语言已经已经对 WebSocket 做了封装，提供了很好的 library，可以直接拿来用。


## 详谈Socket
### TCP/UDP
因为是 Socket 是对 TCP 和 UDP 的封装，所以看一下 TCP 和 UDP 的报文格式。

![tcpandudp](https://i.loli.net/2018/06/05/5b16b06f295d2.jpeg)

通讯过程:

![tcpProcess](https://i.loli.net/2018/06/07/5b18919f2343f.png)


### Socket 的通讯过程
服务器端：

1. 申请一个socket
2. 绑定到一个IP地址和一个端口上
3. 开启侦听，等待接收连接

 客户端：

1. 申请一个socket
2. 连接服务器(指明IP地址和端口号)

服务器端接收到连接请求后，产生一个新的socket(端口大于1024)与客户端建立连接并进行通信，原监听socket继续监听。


## 关于HTTP的番外篇
HTTP 的 Header 里有一个 Connection 的 key，里面的参数通常为 keep-alive，这个值并不是表示为长连接的 HTTP，只是表示这个 TCP 多保存一会，让多个 HTTP 连接复用同一个 TCP，以减少握手挥手所造成的损失。

还有一些 HTTP 头部信息的文档可以看 [HTTP —— MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Connection)。


## References
- [TCP/IP每层及OSI七层对应协议](http://thinking80s.iteye.com/blog/566438)
- [http webservice socket的区别 —— 陈建忠](https://www.cnblogs.com/111testing/p/6581062.html)
- [C# Socket网络编程精华篇 —— 微冷的雨(cnblog)](http://www.cnblogs.com/weilengdeyu/archive/2013/03/08/2949101.html)
- [TCP/IP、Http、Socket的区别 —— pk_zsq(CSDN)](https://blog.csdn.net/Pk_zsq/article/details/6087367)
- [HTTP协议头部与Keep-Alive模式详解 —— zfrong（CSND)](https://blog.csdn.net/zfrong/article/details/6070608)
- [HTTP —— MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Connection)
- [网络协议图](http://www.colasoft.com.cn/download/network-protocol-map-2017.pdf) ：一个很大很全的关于网络协议的 PDF