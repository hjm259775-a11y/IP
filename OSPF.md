# OSPF





RID---区分和标识不同路由设备的ID —— 32位二进制构成

​			1，手工配置---1，IP地址格式进行配置（点分十进制）；2，全网唯一

​			2，自动获取——使用设备的IP地址作为RID，优先选择环回接口的IP地址作为RID，如果存在多个环回接口，则选择环回接口中IP地址数值最大的作为RID；如果没有环回接口，则使用物理接口的IP地址作为RID，如果物理接口存在很多个，则优先选择物理接口中IP地址数值最大的作为RID。（**有环回选环回，没换回选直连，选大的**）



## OSPF的数据包



注意：OSPF是跨层封装的协议，没有传输层，所以，网络层的IP协议需要一个协议号来标定OSPF，协议号为89。（写上89，让网络层能认出来直接交给OSPF进程，避免给TCP，UDP，ICMP）



### 共同包头

------------------

首先，我们要知道，OSPF的数据包的头部都是一样的

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260916121837172.png" alt="image-20260916121837172" style="zoom:33%;" />

版本————OSPF的版本（用的v2版本，0000 0010）

类型————区分是哪种数据包（hello——1，DBD——2，LSR——3，LSU——4，LSACK——5）

报文长度——就是报文长度

RID————发出该数据包设备的RID

区域ID——发出这个数据包的接口所在的区域ID

校验和——对数据进行特定算法计算得出

校验类型——手工认证时（NULL——0——无认证，simple——1——明文认证，MD5——2——对比摘要值认证）

认证数据——

| AuType 值（验证类型） | 名称   | 含义     | Authentication 字段内容（认证数据） |
| :-------------------- | :----- | :------- | :---------------------------------- |
| **0**                 | Null   | 无认证   | 全 0（或忽略）                      |
| **1**                 | Simple | 明文认证 | 直接填 8 字节明文密码               |
| **2**                 | MD5    | 摘要认证 | 填 Key ID、MD5 摘要等信息           |

​	MD5是一种哈希算法，能把任意字节长度的数据算成一个128位的数据，通过比对二者MD5算出的值来进行验证（特点：输入相同，则输出相同；偷偷改一点结果就不一样，雪崩效应；不可逆性）

注意：在OSPF中，认证需要先比对认证类型，再比对认证数据。





### hello包

------------------

​	作用：周期性的发现，建立和保活邻居关系。



​		hello时间---10S（30s，网络类型不同导致的）

​		Dead time---4倍的hello时间



​		



<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260915172843577.png" alt="image-20260915172843577" style="zoom:67%;" />



*   <span style="color:red;">网络掩码</span>——发送该hello包接口IP对应的掩码。（邻居关系的建立需要核查参数，标红就是核查项，只在MA网络中生效）

*   <span style="color:red;">Hello时间</span>——如果双方hello时间或死亡时间对不上，则无法建立邻居关系。

*   <span style="color:red;">死亡时间</span>——如果双方hello时间或死亡时间对不上，则无法建立邻居关系。

*   可选项——8个标记位——每一个标记位置1时，都代表OSPF循某一种OSPF的特性。其中包含<span style="color:red;">特殊区域的标记位</span>（下图的E和N）

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260915194644825.png" alt="image-20260915194644825" style="zoom: 25%;" />



*   路由器的优先级——发出hello包接口的优先级——DR，BDR选举时使用的优先级。

*   指定路由器——DR接口的IP地址（没选举成功前用0.0.0.0）

*   备份指定路由器——BDR接口的IP地址（没选举成功前用0.0.0.0）

*   邻居——本地已知邻居的RID（通过看别人的邻居RID来发现邻居）





发现邻居的条件：

​	收到对方的Hello包里面的邻居列表存在自己



建立邻居关系的因素：（这五个参数对上就能建立邻居关系）

​	1，网络掩码

​	2，hello时间

​	3，死亡时间

​	4，特殊区域标记

​	5，认证









### DBD包

------------------

数据库描述报文---链路状态数据库---LSDB数据库---LSA的信息---仅是描述数据库中LSA的摘要信息。——菜单

​	作用：1，主从关系选举；2，共享LSDB摘要信息；3，确认包（一开始隐形确认，交流到后面都不交流LSA了，但是小弟收到老大后还是得回一个八位的后三位全置为0的数据包，序列号是老大给的，代表收到）



<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260915202103978.png" alt="image-20260915202103978" style="zoom:80%;" />

*   接口最大传输单元——MTU——网络接口一次能传输的最大数据包大小（开启MTU检测：`[r2-Gigabittheret0/0/0]ospf mtu-enable`）（**两端开启检测后MTU值不同会停留在Exstart状态**）

*   可选项——8个标记位——每一个标记位置1时，都代表OSPF循某一种OSPF的特性（和Hello包是一样的）

*   八位二进制，前五位保留着，没用，后三位（I置为1，代表此时的DBD是进行主从关系选举的DBD包，下面的LSA头部就不需要带东西了，就是我们所说的“未携带数据的DBD包”。；M置为1，代表该DBD包之后还会存在其他的DBD包。；MS置为1，则代表该发送数据包的设备为主。）

*   DBD序列号——可以表明发送DBD包的顺序，同时也作为隐形确认的依据均由主设备进行主导。（指定路由器发送数据包时贴上序列号，别的路由器回复时只能用这个序列号 ，这样指定路由器就知道他们有没有收到了，顺便给序列号来个递增，就知道顺序了）





>模拟DBD过程：（I，M，MS，序列号）
>
>小弟A：1，1，1，109（自己定的）
>
>小弟B：1，1，1，129（自己定的）
>
>比较RID后小弟B晋升老大
>
>小弟A：0，0，0，129（老大给的）（小弟先进贡）
>
>老大：0，0，1，130（自己递增的）
>
>小弟A：0，0，0，130（老大给的）



发的第一个DBD：

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260915214611854.png" alt="image-20260915214611854" style="zoom:80%;" />

可以看到，在没有选出指定路由器时，会认为自己是指定路由器，这个104也是自己给自己设置的

在确认老大之后，小弟会发送序列号为老大的数据包给老大看自己的LSA













### LSR包

------------------

链路状态请求报文

​	作用：根据DBD包，请求自己本地未知的LSA



<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260915224436757.png" alt="image-20260915224436757" style="zoom:80%;" />

以下三个为一组，一个LSR包可能有很多组

*   链路状态类型
*   链路状态ID
*   通告路由器



链路状态类型，链路状态ID，通告路由器---LSA的三元组，知道这三个就可以确定一条链路（以上这三个参数可以唯一的标识出一条LSA）





#### LSA信息与类别详解









##### 公共头部

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260917231000612.png" alt="image-20260917231000612" style="zoom:67%;" />

*   **链路状态类型**————OSPF V2中需要掌握的LSA类型一共有6种
*   **链路状态ID**————LSA的身份标识，但是，不同类型的LSA的LS ID的生成方式可能不同，就会导致该参数可能会重复，无法唯一标识LSA，不过也会携带重要信息
*   **通告路由器**————发出该LSA的设备的RID

*   LS age————LSA的老化时间————从一条LSA创造出来开始计时，之后，在整个LSA传递过程中始终累加计时。一般情况下，当一条LSA的老化时间达到1800s时，将重新发送一条新的LSA（就是把表重新发一次，原模原样，拓扑变化会立刻更新的，不会等30min，不用担心）（这才是OSPF每30min周期更新的原因，这个时间单独记录在每个LSA身上的，并非所有LSA一起更新）

​	最大老化时间为3600s，当一条LSA的老化时间达到最大老化时间时，将直接删除该LSA。

*   Len————整个LSA的长度

*   Options————可选项——8个标记位——每一个标记位置1时，都代表OSPF循某一种OSPF的特性。（就是hello包和DBD包里的可选项）

*   seq#————用来表示一条LSA的新旧关系。每发出同一条LSA时，序列号将加1。（32位二进制，但是用8位16进制表示，为什么第一位是8，具体可以搜棒棒糖结构）（取值范围80000001-7FFFFFFE）
*   chksum————校验和————完整性校验，也参与LSA新旧关系的判断，如果两条相同的LSA，序列号也相同，就可以比较校验和，认定校验和大的为新。





##### Router LSA

------------------------------

| 类别               | LS ID      | 通告路由器                      | 传播范围 | 携带的数据           |
| ------------------ | ---------- | ------------------------------- | -------- | -------------------- |
| Type 1 LSA——Router | 和右边一样 | 设备自己的RID（谁发的就是谁的） | 单区域   | 拓扑信息（域内信息） |

Type 1 LSA————网络中每一台设备发送并且只发送一条一类LSA，使用通告路由器的RID作为LS ID。携带的是拓扑信息（区域内传递的）

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260918150904078.png" alt="image-20260918150904078" style="zoom: 67%;" />

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260918164831297.png" alt="image-20260918164831297" style="zoom:67%;" />

已经能画出来了

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260918165344268.png" alt="image-20260918165344268" style="zoom: 33%;" />





>注：LINK——用来描述接口连接信息，每一个接口都可以通过**一条或者多条LINK**来进行描述
>
>Link也分类型，每个类型所对应的一系列数值计算方式都不同：
>
><img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260918152744558.png" alt="image-20260918152744558" style="zoom: 67%;" />
>
>*   Link ID————上图
>
>*   Data————上图
>
>*   Link Type————上图
>
>*   Metric————其实就是Cost开销值
>
>*   Priority————无用之人，死。。。
>
>
>
>接口是P2P时，类型就是P2P，Link ID就是指的自己对面那个设备的RID，Date指的是自己接口的IP地址
>
>接口是MA时，类型就是TransNet，Link ID指的老大的IP地址，Date指的是自己接口的IP地址
>
>接口是边缘接口时（连接的对端不需要建立邻居关系），类型就是StubNet，Link ID指的是这个接口所对应的网段，Date就是网段的掩码（**只要这个网段上只有一台 OSPF 路由器，它也是 stubnet**，不管这个接口是 P2P、Loopback 还是以太网。查到的）
>
>接口是虚链路时，类型就是Virtual，Link ID就是虚链路的对面的设备RID，Date就是这条虚链路自己的接口的IP地址
>
>
>
>Link count————Link数量











##### Network LSA

------------------------

| 类别                | LS ID      | 通告路由器          | 传播范围 | 携带的数据                     |
| ------------------- | ---------- | ------------------- | -------- | ------------------------------ |
| Type-2 LSA——network | DR接口的IP | MA网络中DR设备的RID | 单区域   | 拓扑信息的补充信息（域内信息） |

Type-2 LSA————用来补充1类LSA在描述MA网络时拓扑的补充信息。（因为需要补充的是公共信息，所以不需要所有设备都发送，仅仅一台设备发送就好，就由MA网络中DR所在设备作为通告者，并使用DR接口的IP地址作为LS ID）

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260919162149434.png" alt="image-20260919162149434" style="zoom:50%;" />

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260919162527223.png" alt="image-20260919162527223" style="zoom: 33%;" />

也能画出来了



图上说明这是由RID是3.3.3.3的DR发出的，补充右边的MA网络是24位掩码，该网段设备只有2.2.2.2和3.3.3.3




















##### Sum-Net LSA

------------------------

| 类别                           | LS ID                | 通告路由器                                                | 传播范围            | 携带的数据           |
| ------------------------------ | -------------------- | --------------------------------------------------------- | ------------------- | -------------------- |
| Type-3 LSA——Sum-Net（summary） | 传递的本条路由的网段 | ABR设备（如果发送到下一个ABR设备时，会切换为新的ABR设备） | ABR设备相邻的单区域 | 路由信息（域间信息） |



除了1类和2类LSA之外，剩余所有类型的LSA均传递的是路由信息。其余传递路由信息的LSA需要1类LSA和2类LSA进行验算（需要通过一类和二类来查找通告者ABR）



Type-3 LSA——Sum-Net————都是由ABR设备进行通告，使用传递的这条路由的目标网段作为LS ID。里面有路由的子网掩码和开销值，LSA中携带的开销值为通告者到达目标网段的开销值，

​	1，收到 Type-3 LSA 后，先看通告者，也就是 ABR 的 Router ID。

​	2，通过 Type-1 LSA 和 Type-2 LSA 计算出：我到这个 ABR 的最短路径开销是多少，下一跳是谁，出接口是哪个。

​	3，然后：

​			总开销 = 我到 ABR 的开销 + Type-3 LSA 里的开销

​			下一跳 = 我到 ABR 的下一跳

​			出接口 = 我到 ABR 的出接口

​	4，把这些写到路由表里

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260919211049874.png" alt="image-20260919211049874" style="zoom:67%;" />



以及区域的事，为什么是单区域，因为三类LSA依然只能一个区域内传播，即便要进行转发操作，也会修改通告路由器，同一个三类LSA绝对只会在这一个区域内传播，这跟他传播区域外的路由信息并不冲突









<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260919225447912.png" alt="image-20260919225447912" style="zoom:67%;" />

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260919224940752.png" alt="image-20260919224940752" style="zoom: 67%;" />

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260919225010921.png" alt="image-20260919225010921" style="zoom:67%;" />

可以看到，R1把区域0和区域1的路由信息都一股脑给区域2了，通告设备是自己，要是区域2想访问区域1，就先给R1，然后转发给R3就可以了













##### Sum-asbf LSA

------------------------

| 类别                         | LS ID                            | 通告路由器                                                   | 传播范围                   | 携带的数据   |
| ---------------------------- | -------------------------------- | ------------------------------------------------------------ | -------------------------- | ------------ |
| Type-4 LSA——Sum-asbr（asbr） | 所要去的那个重发布网段的ASBR设备 | 想去ASBR时要经过的下一个ABR设备（如果发送到下一个ABR设备时，会切换为新的ABR设备） | 单区域（除去ASBR所在区域） | ASBR位置信息 |

ABR发送的————这是对五类LSA传播时的验算用的，ASBR设备所在区域的ABR设备首先通告，通过下个ABR设备时将换成新的ABR设备。携带的是ASBR设备的位置信息，开销值也是通告者到达ASBR设备的开销值

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260920104445630.png" alt="image-20260920104445630" style="zoom:67%;" />

























##### External LSA

------------------------

| 类别                        | LS ID                | 通告路由器 | 传播范围           | 携带的数据       |
| --------------------------- | -------------------- | ---------- | ------------------ | ---------------- |
| Type-5 LSA——External（ase） | 传递的本条路由的网段 | ASBR设备   | 整个OSPF网络全区域 | 路由信息（域外） |

ASBR发送的，





<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260920095915505.png" alt="image-20260920095915505" style="zoom:67%;" />

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260920094305246.png" alt="image-20260920094305246" style="zoom:67%;" />

可以看到，这边多了一个板块，表示域外信息



我们展开看看

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260920094840033.png" alt="image-20260920094840033" style="zoom:67%;" />

*   Metric度量值（COST）—— 因为不同协议开销值的计算规则不同，所以，重发布之后，将舍弃原先网络的开销值，并赋予一个初始的种子度量值（seed-metric）。OSPF默认的种子度量值为1。

`[r4-ospf-1]import-route rip 1 cost 2`重发布的时候修改度量值



*   EType——开销值类型类型——类型1——————E位置0————所有设备到达域外该路由的开销值为到达通告者沿途开销值的累加值加种子度量值
    						类型2（默认）——E位置1————所有设备到达域外该路由的开销值均为种子度量值

    `[r4-ospf-1]import-route rip 1 type 1`重发布的时候修改开销值类型

*   ForwardingAddress————转发地址



*   Tag————标签————可以给LSA打上标签，方便以后进行抓取

`[r4-ospf-1]import-route rip 1 tag 666`





可能有人会问，那个人就是我，为什么不让四类和五类挤一挤放一块呢，因为二者是由不同的设备通告的，二者都有对方不知道的信息，所以只能分开













| 类别 | LS ID | 通告路由器 | 传播范围 | 携带的数据 |
| ---- | ----- | ---------- | -------- | ---------- |
|      |       |            |          |            |













### LSU包

------------------

链路状态更新报文

​	作用：真正携带LSA的数据包



<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260915232302476.png" alt="image-20260915232302476" style="zoom:80%;" />

LSA个数——LSA的个数

LSA————具体的信息





### LSACK包

------------------

 链路状态确认报文---确认包



<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260916120406354.png" alt="image-20260916120406354" style="zoom:80%;" />

LSA头部——把收到的LSA信息的头部再发回去，代表自己收到了



















顺便看下有意思的点：

![image-20260916134315775](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260916134315775.png)

下面的DIP有些是224组播开头：

​	有些是单播，因为像LSR、LSU、LSACK其实是两台设备之间的交流，单播正常，

​	而组播是因为会触发更新，所以需要定期发送LSU













## OSPF的状态机



Down状态---接口激活，发送hello之后进入到下一个状态;

init状态---当收到邻居的hello包中包含自己本地的RID，则进入到下一个状态；

TWO-WAY状态---标志着邻居关系的建立

（条件匹配）---条件匹配成功，则进入到后面的状态；条件匹配失败，则将停留在邻居关系，通过hello包进行周期保活即可。
Exstart（预启动）状态---通过使用未携带数据的DBD包进行主从关系选举。为主的可以优先进行LSA的选择，为从的先进入到下一个状态，发送数据库的摘要信息。并且，为主的设备可以主导DBD包的隐形确认。
Exchange（准交换）状态---通过发送携带数据的DBD包进行LSA摘要信息的共享

Loading状态--基于DBD包，使用LSR/LSU/LSACK获取未知的LSA信息

FULL状态---标志着邻接关系的建立。





添：

Attempt——尝试状态——一方指定对端的单播邻居后，等待对方指定本端时所处于的状态————NBMA网络类型下特有的状态







## OSPF接口网络类型

（具体可以看网络类型和数据链路层协议那一篇笔记）

*   P2P点到点网络

*   MA多点接入网络


​		BMA——支持广播的多点接入网络

​		NBMA——不支持广播的多点接入网络



​	以太网——因为以太网协议可以组件一个多点的网络环境，所以，不同的节点需要使用不同的MAC地址进行区分和标识



P2P网络——仅能存在两台设备的网络，不需要使用MAC地址也可以正常通信







好，到此为止了！😡

**OSPF在不同的网络环境下的工作方式是不一样的**



<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260916164645309.png" alt="image-20260916164645309" style="zoom:80%;" />

举上图例子看看



先查看OSPF接口的状态：



### BMA



<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260916164039341.png" alt="image-20260916164039341" style="zoom:67%;" />

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260916171705183.png" alt="image-20260916171705183" style="zoom:67%;" />



### PPP

（PPP接口开销值甚至48（华为设备默认遵循的是E1标准——2.048MBps））

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260916172038093.png" alt="image-20260916172038093" style="zoom:67%;" />



### 环回接口

（华为设备中，环回接口的开销值被设定为0）

​							华为设备中，环回接口对应的路由默认是按照主机路由进行学习的(就是你输入2.2.2.0/24，但实际在路由表是2.2.2.2/32)，想改吗？/奶龙笑.jpg/，将环回接口的接口网络类型改成Broadcast，`[r2-LoopBacko]ospf network-type broadcast`,就可以还原了

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260916203236088.png" alt="image-20260916203236088" style="zoom:67%;" />



### NBMA

（帧中继）

​	我们要知道，NBMA是一个没有广播和组播的网络类型，但是OSPF的邻居发现就是需要用到组播，这个时候。可以手动给他邻居，让他实现单播邻居，`[r1-ospf-1]peer 12.0.0.2`，`[r2-ospf-1]peer 12.0.0.1`，来实现OSPF的单播建邻

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260916210255586.png" alt="image-20260916210255586" style="zoom:67%;" />

​	



### P2MP

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260916213637640.png" alt="image-20260916213637640" style="zoom:67%;" />



**顺便一提：Broadcast接口可以和P2P建立邻居关系**（因为只看那五条），但是获取不到路由信息







| 网络类型       | OSPF接口的网络类型和工作方式                                 |
| -------------- | ------------------------------------------------------------ |
| BMA（以太网）  | 网络类型：Broadcast。工作方式：需要进行DR和BDR选举；hello时间为10S，死亡时间为40S；可以建立多个邻居关系 |
| P2P（PPP）     | 网络类型：P2P。工作方式：不需要进行DR和BDR选举；hello时间为10S，死亡时间为40S；只能建立一个邻居关系 |
| 环回接口       | 网络类型：P2P（只是华为写个P2P，装个样子的）。就算有工作过程也是装样子的 |
| NBMA（帧中继） | 网络类型：NBMA。工作方式：需要进行DR和BDR选举；hello时间为30S，死亡时间为120S；可以建立多个邻居关系，只能手工建立邻居关系。 |
|                | 网络类型：P2MP（没有对应的网络环境，无法由设备自动生成，必须由管理员手工更改）。不需要进行DR和BDR选举（每一台设备都是点到点，所以不需要），hello时间为30S，死亡时间为120S；可以建立多个邻居关系。（算是对NBMA的改进） |













## OSPF不规则区域



首先我们知道：OSPF区域划分的要求

​	1，区域之间必须存在ABR设备

​	2，区域划分必须按照星型拓扑





不规则区域指的是：

​	1，远离骨干的非骨干区域

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260917110438031.png" alt="image-20260917110438031" style="zoom:33%;" />

​	2，不连续骨干

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260917110357168.png" alt="image-20260917110357168" style="zoom: 33%;" />





<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260917114635486.png" alt="image-20260917114635486" style="zoom:80%;" />

解决这些问题有三种办法：

### 利用VPN技术









### 利用OSPF内置的虚链路

```
[r4-ospf-1-area-0.0.0.1]vlink-peer 2.2.2.2

[r2-ospf-1-area-0.0.0.1]vlink-peer 4.4.4.4

相当于R4这个不合法的ABR设备找了合法的ABR设备R2做担保，这么理解
实际是想让这条虚链路视为在区域0内
```





只能通过这个查看

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260917115452109.png" alt="image-20260917115452109" style="zoom:80%;" />

开销值为2

Transit Area: 0.0.0.1——穿越区域1





注意：

​	1，虚链路的配置是双向的

​	2，虚链路之间也会周期性的发送hello包进行保活，只不过，使用单播发送。

​	3，这条虚链路永远属于区域0

​	4，虚链路只能穿越一个区域（其实配置的时候也能看出来，写的是同区域的对面的RID）



缺点：

​	1，虚链路只能穿越一个区域

​	2，因为需要维护虚链路邻居的邻居关系，也会发送一些周期性的数据，都会额外的消耗穿越路资源。









### 利用多进程双向重发布

​	不同协议之间或者不同的进程之间都存在信息隔离。重发布就是将一种协议的路由按照另一种议的规则发布出去，打破这种信息隔离。



<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260917222312886.png" alt="image-20260917222312886" style="zoom:67%;" />

解决方案如图，同时配置两个进程的是R4，多线程双向重发布可以将一种协议的路由信息按照另一种路由协议的规则转化过去，从而实现打破信息隔离的状况，甚至可以左边运行RIP，右边运行OSPF



R3被称为ASBR————协议边界/进程边界/AS边界设备————同时运行多种不同的协议或者同时运行一种协议的不同进程

```
[r3-ospf-2]import-route ospf 1
在区域2里面导入OSPF进程1的路由
```



```
[r3-ospf-1]import-route ospf 2
在区域1里面导入OSPF进程2的路由
```

这样两边的路由就能互相传递了





注意：这样学来的路由标的是O_ASE，代表域外路由，优先级是150

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260917224445973.png" alt="image-20260917224445973" style="zoom:67%;" />









顺便一提，想要OSPF和RIP互通也是可以的

```
[r4-ospf-1]import-route rip 1
```



```
[r4-ospf-1limport-route rip 1 cost 2
设置开销值
```













## OSPF的工作过程











### OSPF 在 MA 中的 DR/BDR 选举

==指定路由器==——DR：

​	在一个MA网络中，如果所有的设备都建立邻接关系，则将造成大量的重复更新，所以，需要选择一台设备和其余设备建立邻接关系，其他设备之间，仅保留邻居关系即可，该指定设备即为DR设备。

==备份指定路由器==---BDR：

​	BDR设备和其余设备之间也需要建立邻接关系，作为DR设备的备份。（先选举DR，再选举BDR）

==其余路由器==——DROther

​	其余非DR和BDR的设备统称。只有DROther之间才会建立邻居关系。 

**注意**：DR，BDR实际上是接口的概念

**注意**：DR和BDR设备会额外监听224.0.0.6这个组播地址，其余所有设备监听224.0.0.5组播地址，为了避免重复更新。（想想，传输的时候经过交换机可以是会泛洪的，为了避免二次收到同样的数据包，要是结构变化，小弟会发一个224.0.0.6的数据包先让老大知道，再让老大去转发）





**DR和BDR的选举**

------------------

​	先选择DR设备，DR设备确定后，再选择BDR设备。



​	1，比较接口的优先级，选择优先级最大的作为DR
​		优先级---8位二进制-0-255，默认值为1
​		如果将一个接口的优先级修改为0，则代表该设备不参与DR和BDR的选举。

​	2，如果优先级相同，则比较设备的RID，优先选择RID最大的设备的接口成为DR。

​	3，之后选举BDR，和DR选举规则相同，在剩余接口中进行选择。

​		在OSPF中，DR和BDR的选举是非抢占。选举时间默认等同于死亡时间。



## OSPF优化





减少LSA的更新量方面，可以：

​	1，汇总——减少骨干区域的更新量

​	2，做特殊区域——减少非骨干区域的更新量（这个区域只学自己的，不学别的区域的）































## 配置











