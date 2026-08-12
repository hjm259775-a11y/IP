**VPN** (virtual private network）——虚拟专用网





## 技术背景

​	Internet网络不安全；

​	通过专线连接分支机构成本高；

​	PSTN拨号成本高，速率低；



## 作用：

==***利用共享公网构建专有私网，来达到跨越公网，访问私网主机的目的***==（就是让一个内网中的主机跨越公网，来访问另外一个内网中的主机）



可能有人会说，这不就是NAT吗，VPN相比于NAT，有更多优势，VPN只有在与目标内网建立连接之后才能访问（内网->内网，单个用户->内网都可以），而NAT只需要知道连接内网的外接口即可，相比之下，**VPN更安全**

 



*   部署简单快捷；

*   与私有网络一样提供安全性、可靠性和可管理性；

*   通过Internet互连，不受地理位置限制，成本低；

*   简化用户侧的配置和维护工作



！？强强？！



## 原理：

先浅谈一下：

![image-20260808001450525](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260808001450525.png)

如图，建立VLAN连接之后，1.0里的主机发东西给2.0里的主机时，在要到达公网时会在SIP和DIP的旁边放上100.1.1.1和100.1.1.2，这样以便在公网上进行传输，就是下面这张图





VPN又叫隧道技术：

定义：使用一种协议去封装另一种协议

![image-20260808004752574](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260808004752574.png)

​	载荷数据：被封装的原始数据

​	载荷协议：被封装在内层的协议（私网IP头部)

​	封装协议：对载荷协议的封装方式（标识用哪种VPN）

​	承载协议：再次封装的外层协议（公网IP头部)



## 分类

了解一下：



按使用场景：
(1)site-to-site vpn：站点（内网）到站点的VPN，用于连接不同分支机构的VPN，双方的公网地址必须是静态的。

​	IPsec VPN：一种网络层的安全保障技术，在公网上为两个私有网络提供安全通信通道，通过加密通道保证连接的安全。

​	GRE VPN：最简单的VPN，一般和IPsecVPN搭配使用
(2）access vpn：接入vpn，用于把单个移动用户接入到公司内网

​	L2TP VPN（此技术被淘汰）：隧道到传送PPP网络，二层VPN，用L2TP VPN构建access VPN

​	SSL VPN：SSL VPN是解决远程用户访问公司敏感数据，最简单最安全的技术



按工作层次（OSI参考模型）：
二层VPN:

​	L2TP VPN（此技术被淘汰）

​	==EVPN==

​	==VXLAN==

​	==BGP MPLS VPN==

三层VPN:
	IPSEC VPN

​	GRE VPN

​	==EVPN==

​	==VXLAN==

​	==BGP MPLS VPN==

七层VPN：

​	SSL VPN



按建设者分：
运营商：BGP MPLS VPN（收费）
用户自建：L2TP VPN（此技术被淘汰）、IPSEC VPN、GRE VPN、SSL VPN等



## GRE VPN

现在我们来学习下GRE技术：

Genric Routing Encapsulation，通用路由封装，标准的三层隧道技术，是一种点对点的隧道，在任意一种网络协议上传送任意一种其他网络协议的封装方法。



那GRE VPN就是直接使用GRE封装建立GRE隧道，在一种协议的网络上传送其他协议

虚拟的隧道接口(Tunnel)

隧道的两端必须得是在同一个网段





这个知识点需要知道：

![image-20260809023910941](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260809023910941.png)

左边IP头是公网的IP，协议为47代表之后让GRE协议处理

GRE头的以太网类型为0x0800代表下一步让IP协议处理

右边IP头是内网的IP







详细了解下工作过程：

**工作过程：**

------------------------

*   隧道起点找到私网路由，数据包发往Tunnel

*   数据包在Tunel口进行封装隧道使用的协议和公网IP头部

*   根据公网IP头部查找路由表，并转发

*   数据包在公网进行传输

*   查找公网路由并解除公网IP头部封装，交给GRE处理

*   隧道终点查找私网路由并转发至目的主机



举个例子：

![image-20260809024754621](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260809024754621.png)

首先需要了解：隧道其实就是将192.168.2.0和10.1.1.2对应起来，形成一个映射，当有数据包发来时，查看是否对应，如果是要发送给2.0的，就直接进隧道且下一跳变成10.1.1.2



当PC1开始发送数据时：SIP+DIP

发到R1时，==R1查看路由表==，发现目标网段有设置，就扔给10.1.1.1隧道

到了隧道，根据GRE提供的信息，看到10.1.1.2对应的是200.2.2.2，包装GRE头和公网SIP+公网DIP，变成公网SIP（100.1.1.1）+公网DIP（200.2.2.2）+GRE+SIP+DIP

==R1查看路由表==，根据公网SIP+公网DIP发送给200.2.2.2

发送给200.2.2.2后，==R2查看路由表==，发现是给自己的，把公网头部拆掉

拆完因为公网头协议为47代表之后让GRE协议处理，之后又因为以太网类型为0x0800让IP协议处理

R2看到内网IP，==R2查看路由表==，发给2.0网段



## GRE VPN优缺点

**优点：**

*   可以用当前最为普遍的IP网络作为承载网络：

*   支持多种网络层协议

*   支持组播和动态路由协议；

*   配置简单、部署容易：



**缺点：**

*   点对点隧道：

*   静态配置隧道参数；

*   布置复杂连接关系时，代价巨大；

*   缺乏安全性（传输公网的数据没有加密解密）

*   不能分割地址空间 (不能解决私网地址冲突的问题：两个隧道所对应的私网网段可能重复)





## 多Tunnel口冗余技术

要是这条隧道对应的公网寄了，就炸了，所以，我们需要冗余



作用：主隧道转发数据，备用隧道完全处于空闲状态；

同时需要开启Keepalive（保活机制）来检测隧道运行状态

![image-20260809164432193](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260809164432193.png)

了解一下





## 配置

先看下要求：

![image-20260809171219009](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260809171219009.png)

跳过配置IP地址，动态路由等（顺嘴一提，配置RIP需要宣告192.168.3.0网段）



```
[R2]interface Tunnel 0/0/0	
[R2-Tunnel0/0/0]ip address 192.168.3.1 24
设置隧道IP，注意两边隧道IP要在同一个网段

[R2-Tunnel0/0/0]tunnel-protocol ?
  gre        Generic Routing Encapsulation
  ipsec      IPSEC Encapsulation
  ipv4-ipv6  IP over IPv6 encapsulation
  ipv6-ipv4  IPv6 over IP encapsulation
  mpls       MPLS Encapsulation
  none       Null Encapsulation
[R2-Tunnel0/0/0]tunnel-protocol gre
设置类型为GRE VPN

[R2-Tunnel0/0/0]source 100.1.1.1
设置自己的隧道所对应的公网IP

[R2-Tunnel0/0/0]destination 100.2.2.2
设置隧道对面的公网IP，也就是自己的目的地



当然，R4也得配
```





```
[R2]ip route-static 192.168.2.0 24 Tunnel 0/0/0

还需要在边界设备上配置路由表，让他走隧道
```















