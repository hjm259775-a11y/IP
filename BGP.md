# BGP



EGP——GP
IGP——OSPF，RIP



AS —— 自治系统 —— A5号 —— 16位二进制构成 —— 1->65534（64512-65534私有AS号）

————现在也有了32位二级制拓展AS号





BGP————边界网关协议————BGP V4（针对于IPv4）

​						    BGP V4 + ————MP-BGP （支持多种地址族，IPV4和IPV6都可以）





重发布也可以实现AS之间路由传递的效果，但是：

​	1，多点的重发布必然会出现选路不佳的情况

​	2，ASBR设备的归属问题





RIP————无类别的**距离矢量型协议**

BGP————无类别的**路径矢量型协议**

​	路径矢量和距离矢量的区别：

​		1，距离指RIP始终以路由器为单位衡量开销

​		    路径指BGP始终将一个AS作为单位来看待

​		2，距离矢量本身是一种算法的概念，但是，路径矢量不牵扯算法，因为BGP仅是将IGP生成的路由传递到其他的AS即可。








IGP————选路佳，收敛快，占用资源少

## BGP的特点：

### 1，可控性 

———— 因为AS之间需要传递大量的路由信息，所谓可控，就是可以方便干涉选路，更容易做策略。

​		在BGP中，直接舍弃了开销值，取而代之的是设计了一系列的路由的路径属性。



### 2，可靠性

（因为没办法周期更新，所以需要保证自己的传输足够可靠，直接采用TCP（179号端口）协议作为传输层的协议。当然，肯定存在触发更新）

​		TCP的问题点：

​			1，TCP速度较慢，效率较低；

​			2，一旦选择了TCP协议，则将无法支持组播或者广播（无法自动发现和建立邻居关系）





​	BGP中可以实现非直连建邻————BGP的非直连建邻是建立在IGP的基础上的。（一个AS可能有多个BGP设备，他们需要建邻以传递路由信息）



​	邻居关系可以分为两类：

​		1，IBGP对等体（对等体就是邻居）————如果建立邻居的双方设备在同一个AS中则建立的关系称为IBGP对等体

​		2，EBGP对等体————如果建立邻居的双方设备在不同的AS，则建立的关系为EBGP对等体



​	**注意**：==一般情况下，IBGP对等体之间采用非直连建邻，IBGP对等体之间交互的数据包中的TTL值被设置为255。EBGP对等体之间采用直连建邻，EBGP对等体之间交互的数据包中的TTL值被设置为1.==



### as-By-AS

————BGP始终将一个AS作为单位来看待

​	注意：因为AS内部的情况无法评估，所以，在BGP中，几乎不存在负载均衡。如果到达同一个目标网段如果存在多条路径，则将选择其中最优的一条加表。



## BGP的数据包

————周期性发现，建立以及保活邻居关系



在BGP中，发现邻居，有网络管理员手工指定实现



### open报文

————在BGP中，建立邻居关系，由open报文来完成



建立BGP对等体关系，实际上就是参数的协商

影响邻居关系建立的参数：

​	1，AS号（OPEN报文中）（在发送open报文时，会携带自己本地的AS号，会和对端指定的AS号做对比，如果相同，则可以正常建立邻居关系；如果不同则邻居关系建立失败。）



​	2，Route-id（OPEN报文中） ———— 1，手工配置---必须按照IP地址的格式创建

​					   2，自动生成---优先选择环回接口中地址最大的IP作为RID，如果没有环回接口，则将选择物理接口中最大的IP作为RID。（和OSPF一样，有环回选环回，没换回选物理）

​	发送OPEN报文，里面有自己本地的RID，RIP不一样就建立邻居关系，一样就无法建立邻居关系



​	3，认证（TCP报文中）————在TCP建立阶段，会将认证口令携带在TCP的option字段中，如果口令不同，则邻居关系建立失败。



​	4，IP地址

**注意**：==在手工建立邻居关系时，会比较自己收到的open报文中的源IP和自己本地指定的IP地址，要是不一样，则将影响邻居关系的建立。==







以上是影响邻居建立的，下面讲些OPEN报文里带着的东西

​	Holdtime——保活时间（死亡时间）——默认为180s————如果邻居双方保活时间不同，依然可以正常建立邻居关系，但是执行时，会选用二者之间时间较短的作为保活时间

​	router-refresh————open报文中会携带自身是否支持router-refresh功能。（第五种数据包）





### Keeplive报文

————在BGP中，保活邻居关系，由keeplive报文完成

默认周期发送时间为保活时间的三分之一（默认就是60s）



keeplive除了完成保活工作之外，还会临时的充当OPEN报文的确认包。（确认的是OPEN报文里面的参数我认可了，并非确认收到，确认收到是TCP的事）





### update报文

更新包————真正携带路由信息的数据包





携带目标网段，掩码，路径属性



在BGP的update报文中，存在撤销路由条目的字段，如果某些路由失效，可以直接将这些路由放在该字段下面，对端将直接删除这些路由。





### notification报文

————告警机制



哪个环节出问题了就会发送这个报文，里面会写哪个环节出问题了，因为什么出问题了





### router-refresh报文

————发送这个报文给别人，让别人把路由表信息再给我发一次（路由刷新）



当两台 BGP 路由器已经建立邻居关系后，如果其中一台不想重置会话，但又想重新获取对方完整的路由信息，就可以发送 ROUTE-REFRESH报文。







## BGP的状态机



————BGP可以实现邻居关系建立和传递路由分开执行。所以，BGP的状态机仅描述BGP邻居关系建立的状态变化



<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260927134234759.png" alt="image-20260927134234759" style="zoom:80%;" />

idle————空闲状态————在配置完BGP之后，将会进入到一个检查阶段。（检查对端的IP地址本地是否路由可达）

​			检查通过则进入到下一个状态，没通过就停留在这个状态

Connect————建立TCP链接的阶段

​			会话建立成功————进入到OpenSent状态

​			会话建立失败————进入到Active状态

Active————反复尝试建立TCP连接，尝试成功就进入到OpenSent状态，反复尝试失败就退回到idle状态

OpenSent————发送Open报文协商参数。（在接收对方发来的opensent报文后，用keeplive确认，进入到下一个状态）

OpenConfirm————等待对方发来keeplive报文进行参数确认，收到了就进入下一个状态

Established——————建立完成阶段————标志着对等体关系的建立



<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260927135618190.png" alt="image-20260927135618190" style="zoom:80%;" />

最下面这条线回退到idle





## BGP的工作过程



1，基于IGP实现路由可达

2，手工指定邻居关系，邻居之间通过单播发送消息，通过三次握手建立TCP的会话通道，之后，BGP传输的可靠性均由TCP来保障。

3，使用open报文和keeplive报文进行参数协商完成邻居关系的建立————生成一张表————**邻居表**

4，使用update报文共享路由信息。同时收集对等体发送的路由信息，将所有自己发的以及收到的路由信息记录在一张表中————**BGP表**。

5，将BGP表中的路由信息添加到**路由表**，因为到达同一个目标网段可能存在多条路由，需要选择其中最优的一条加载到路由表中。通过比较路径属性来判断最优

6，收敛完成后，BGP依然会使用keplve报文进行周期保活，保活时间180S，发送周期60S

7，如果出现错误信息，则使用notification报文进行告警

8，如果出现结构突变，则将使用update报文进行触发更新。









## BGP的选路原则



![image-20260928214444459](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260928214444459.png)

0，丢弃所有不可用的路由

1，优选Preferred-Value属性值最大的路由。

2，优选Local_Preference属性值最大的路由。

3，本地始发的BGP路由优于从其他对等体学习到的路由，本地始发的路由优先级：优选手动聚合>自动聚合>network>import>从对等体学到的。

4，优选AS_Path属性最短的路由。

5，优选Origin属性最优的路由。Origin属性值按优先级从高到低的排列是：IGP、EGP及Incomplete。

6，优选MED属性值最小的路由。

7，优选从EBGP对等体学来的路由（EBGP路由优先级高于IBGP路由）。

8，优选到Next_Hop的IGP度量值最小的路由。

9，优选Cluster_List最短的路由。

10，优选Router ID（Orginator_ID）最小的设备通告的路由。

11，优选具有最小IP地址的对等体通告的路由。





```
[r4-bgp]display bgp routing-table 1.1.1.0 24

查看1.1.1.0/24
```

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260929084729829.png" alt="image-20260929084729829" style="zoom: 67%;" />

甚至能看到因为什么导致的选路不占优













<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260928223344963.png" alt="image-20260928223344963" style="zoom:67%;" />



| 属性     | 传播范围       | 默认值            | 评判标准   |
| -------- | -------------- | ----------------- | ---------- |
| PV       | 不传递（本地） | 0（0-65535）      | 越大越优   |
| LP       | IBGP对等体     | 100               | 越大越优   |
| 路由类型 |                |                   |            |
| AS_PATH  | BGP对等体      |                   | 越短越优   |
| OGN      | BGP对等体      | 根据发布方式相关  | i  > e > ? |
| MED      | BGP对等体      | 继承IGP路由开销值 | 越小越优   |
|          |                |                   |            |
|          |                |                   |            |
|          |                |                   |            |



<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260929105311056.png" alt="image-20260929105311056" style="zoom:80%;" />

**1，优选PV值最大的路由**
	PV属性是本地干涉选路最方便的属性————PV属性不能传递（华为私有属性）

`[r4-bgp]peer 3.3.3.3 preferred-value 100`仅具有本地意义，将3.3.3.3传来的路由的PV值改为100（3.3.3.3传来的所有的路由）

```
[r4]ip ip-prefix pv permit 10.0.0.0 24
抓流量

[r4]route-policy pv permit node 10
[r4-route-policy]if-match ip-prefix pv
[r4-route-policy]apply preferred-value 100
[r4]route-policy pv permit node 20
路由策略

[r4-bgp]peer 3.3.3.3 route-policy pv import
应用策略


抓10.0.0.0/24网段的数据，将PV值改为100（传给自己的路由）
```



**2，优选LP值最大的路由**

LP-- 本地优先级---在AS内部进行选路最方便的属性--IBGP对等体

`[r3-bgp]default local-preference 110`仅限于通过 IBGP 学来的路由，而且这条命令影响的是"它发给别人的时候带 110"。



```
[r3]ip ip-prefix pv permit 10.0.0.0 24
[r3]route-policy lp permit node 10
[r3-route-policy]if-match ip-prefix lp
[r3-route-policy]apply local-preference 110
[r3]route-policy lp permit node 20
[r3-bgp]peer 4.4.4.4 route-policy lp export

抓10.0.0.0/24网段的数据，将LP值改为110（发给别人的路由）
```





**3，手工聚合 > 自动聚合 > network > import > 从对等体处学来的**



`[r4-bgp]aggregate 172.16.0.0 22`手工聚合（聚合路由（尤其是自动聚合）可能造成大范围的路由黑洞。为了防止环路，设备会在本地生成一条指向 NULL0 的汇总路由（你的配置里就有 ip route-static 172.16.0.0 22 NULL 0），这条防环路由所依赖的下一跳就是本机回环接口，因此显示为 127.0.0.1。）所以在BGP表里的下一跳会写127.0.0.1



``



`[r4-bgp]network 172.16.0.0 22`本地宣告



`[r4-bgp] import-route static`把本机路由表里的静态路由，引入（注入）到 BGP 进程中，让 BGP 能把它们宣告出去。



``在别的地方敲





**4，优选AS_PATH属性值最短的路由信息**



注意：如果明细路由来自于不同的AS中，在其他AS的设备上进行聚合时，激活了As-Path关键字，则汇总路由将同时携带不同明细AS_path中的AS号，需要使用大括号来括起来，不过在选路上，这算一个（同样的，联邦里的小括号也算一个）





```
[r1]ip ip-prefix as permit 10.0.0.0 24
[r1]route-policy as permit node 10
[r1-route-policy]if-match ip-prefix as
[r1-route-policy]apply as-path 11 22 33 additive
[r1]route-policy as permit node 20
[r1-bgp]peer 12.0.0.2 route-policy as export


在原有AS_PATH属性的基础上添加AS号（additive导致的）
```



<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260929105547864.png" alt="image-20260929105547864" style="zoom:67%;" />

注意：因为AS_PATH属性本身是用来防环的，所有，如果里面出现真实的AS号，则将导致改路由无法传递到该AS中，造成通信障碍。

所以最好用这个`[r2-route-policy]apply as-path 1 1 1 additive`







**5，Origin属性：i > e > ？ (起源码)**





```
[r1]ip ip-prefix ogn permit 10.0.0.0 24
[r1]route-policy ogn permit node 10
[r1-route-policy]if-match ip-prefix ogn
[r1-route-policy]apply origin incomplete
[r1]route-policy ogn permit node 20
[r1-bgp]peer 12.0.0.2 route-policy ogn export

将10.0.0.0/24网段传出去的OGN改为 ？
```



<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260929112725323.png" alt="image-20260929112725323" style="zoom:67%;" />





**6，优选MED属性值最小的路由**

​	MED——多出口鉴别属性————他的默认值继承了IGP路由表中的开销值————不同AS之间控制选路的常用逻辑（两个 AS 之间有多个互联出口时，其中一个 AS 告诉另一个 AS：“去往我这里的某个网段，请优先走哪个入口。）**这个MED反映的是去往这个网段在那个AS内部的开销值**



注意：如果一台设备从自己IBGP对等体处学习到一条路由信息，则在传递出去时，将不携带MEP属性（不带属性就是为0，则可能出现选路不佳的情况。所以还是建议所有边界设备路由全部都发布）



注意：如果同一个网段的路由信息来自于同一个AS的设备，则可以比较第六条；如果来自于不同AS的设备，则将不比较第六条，直接比较第七条。（IGP都可能不一样，根本没有比较的意义）





































## BGP路由黑洞





<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260927143813284.png" alt="image-20260927143813284" style="zoom:67%;" />

由于BGP协议可以非直连建邻，故可能出现BGP协议在跨越未运行BGP协议的路由器时，导致BGP路由信息传递后，显示控制层面可达，但是数据层面，在流量经过这些没有运行BGP协议的路由器时，因为没有路由信息，而无法通过，被丢弃掉，造成路由黑洞。（其实就是R2想访问R1，结果R7传给R3的时候，因为AS内的路由器肯定没有AS外的路由信息，数据包会被丢弃）



怎么解决：

​	1，让所有设备运行BGP协议（不像人类啊😑）

​	2，在边界设备上进行重发布，将BGP的路由信息发布到IGP协议中，让设备通过IGP学习到路由。（那你做AS系统的意义何在啊😡）

​	3，MPLS VPN





同步原则--- 当一台路由器从自己的IBGP对等体处学到一条BGP路由时，将不会发送给自己的EBGP对等体。除非，该设备可以从自己的IGP处也学习到这条路由信息。（华为设备默认关闭了同步原则）













## BGP防环





水平分割：

​	**EBGP对等体的水平分割**————解决EBGP对等体之间出现的环路问题



AS-path————当一个路由信息离开一个AS时，将该AS的AS号记录在AS_PATH属性中。之后，如果设备收到一条路由信息，其中的AS_PATH属性有自己本地的AS号，则将不再学习该路由信息，防止路由回传，起到防环的效果。

（这个AS-path相当于一个链表，每次转发都会在上面写下抓发的设备，当你收到一个AS-path里面本来就有你的路由，你就能知道你之前肯定接手过这个路由）



AS_path也可以影响选路，可以优先选择AS_PATH属性值最短的路由。









​	**IBGP对等体的水平分割**————解决IBGP对等体之间出现的环路问题



<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260927160006326.png" alt="image-20260927160006326" style="zoom:67%;" />



IBGP的水平分割————当一台路由器从一个IBGP对等体处学来一条路由，将不再发送给自己其他的IBGP对等体。



R2把路由发给R4和R3，R3和R4之间不会互相转发这条路由



<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260927160150689.png" alt="image-20260927160150689" style="zoom:67%;" />

那这不炸了，IBGP对等体的水平分割相当于将IBGP对等体之间传递路由控制在1跳之内，可能造成路由的传递障碍。


​	所以用什么方法解决：

​		1，只能使用全连的IBGP对等体（额外造成资源浪费，导致网络拓展性降低）

### 路由反射器

​		2，路由反射器

————————RR

​		可以将某设备配置成为RR，则其在满足一定条件下，可以将收到的路由反射给其他设备

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260928111758022.png" alt="image-20260928111758022" style="zoom:50%;" />


在指定一台设备成为路由反射器的同时，需要再指定一个或多个IBGP对等体作为他的客户，RR和客户之间形成的整体称为反射簇。（用RR的RID作为反射簇的簇ID）



反射规则：（非客户和非客户之间不传）

​	1，所有客户发的路由信息，可以反射给所有的客户以及非客户

​	2，从非客户处学来的路由，则可以反射给自已所有的客户。

​	3，只有可用且优的路由会被反射





注意：路由反射器实质上是打破了IBGP水平分割机制，所以，可能出现环路，需要设置防环机制————

——————originator_id（起源者ID），cluster_list（簇列表）

oridinator_id（起源者1D）————标识一条反射的路由的始发的设备。（图中R3给R4反射的起源者ID会写2.2.2.2）

​	当一个设备收到的路由中没有起源者ID，则他在反射该路由时会将发送者的RID作为起源者ID携带在反射的路由条目中，之后的设备如果发现起源者ID中存在内容，则反射路由将不改变起源者ID，当一台设备收到一条反射的路由时，里面的起源者ID是自己本地的RID，则将不学习该路由，防止路由的回传，出现环路。（就是转发时没有起源者ID，就把发送者的RID贴上去，看到里面有起源者ID就不要动，收到路由时看到起源者ID是自己就别学）





——————cluster_list（簇列表）————当一条反射的路由离开一个反射簇时，需要将该反射簇的簇ID添加到cluster list中



<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260928115136744.png" alt="image-20260928115136744" style="zoom:67%;" />

如图，起源者ID为4.4.4.4，经过R1进行反射再回到R1，也会出环，

所以需要簇列表，每次离开一个簇列表，都把簇ID写在列表里面，这样就算R1收到反射回来的也能在簇列表里面看到自己的簇ID，从而不选用该路由信息



评价：多出来两个属性，就为了不改变原本的As-by-As的属性啊✋😭✋（当然，只在As内部防环使用，发送给自己的EBGP对等体时，不需要携带）

```
[r3-bgp]peer 2.2.2.2 reflect-client

让IP地址为2.2.2.2的成为自己的客户
```









### 联邦

​		3，联邦



<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260928185216499.png" alt="image-20260928185216499" style="zoom: 67%;" />

AS区域外的BGP就老老实实正常配就行了`[R1-bgp]peer 12.0.0.2 as-number 2`





```
[R2]bgp 64512————————————————————————————————————————————————————————————————————————————————————————
[R2-bgp]router-id 2.2.2.2
[R2-bgp]confederation id 2——————————————————————————————————————————————————————————————————————————
联邦内的需要用内部的as号，同时还需要声明自己的As大号

[R2-bgp]peer 12.0.0.1 as-number 1

[R2-bgp]peer 3.3.3.3 as-number 64512
[R2-bgp]peer 3.3.3.3 connect-interface LoopBack 0
[R2-bgp]peer 3.3.3.3 next-hop-local


```







```
[R3]bgp 64512————————————————————————————————————————————————————————————————————————————————————
[R3-bgp]router-id 3.3.3.3
[R3-bgp]confederation id 2————————————————————————————————————————————————————————————————————————
[R3-bgp]peer 2.2.2.2 as-number 64512
[R3-bgp]peer 2.2.2.2 connect-interface LoopBack 0
[R3-bgp]peer 2.2.2.2 next-hop-local




[R3-bgp]confederation peer-as 64513——————————————————————————————————————————————————————————————————————
只有在建立联邦的EBGP对等体的设备上，需要声明联邦对端的AS

[R3-bgp]peer 4.4.4.4 as-number 64513
[R3-bgp]peer 4.4.4.4 connect-interface LoopBack 0
[R3-bgp]peer 4.4.4.4 ebgp-max-hop 20————————————————————————————————————————————————————————————————————————
因为联邦的EBGP需要遵循EBGP对等体的传递原则，所以需要将TTL值改大
```



就是个思路问题，随机应变就好，需要留意的是需要写自己的小as和大As，要是本设备连接其他联邦，需要写对端的小as，并且需要调整TTL值



注意：联邦也打破了IBGP的水平分割，所以，可能会造成环路问题，它使用EBGP的水平分割，AS_PATH中携带联邦的AS号进行防环，只不过使用小括号括起来，在进行防环时，路由将不会回传（**联盟内 EBGP 用 AS_PATH 防环，子 AS 号用小括号括起来。路由器收到路由时，如果 AS_PATH 里有自己的子 AS 号，就丢弃不回传。对外只显示联盟 ID，隐藏内部子 AS。**）

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260928213758164.png" alt="image-20260928213758164" style="zoom:67%;" />











## BGP的基础配置





### 建邻

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260927191807378.png" alt="image-20260927191807378" style="zoom:80%;" />



```
EBGP对等体的直连建邻




[R1]bgp 1
这个1指的是自己所在的AS号

[R1-bgp]router-id 1.1.1.1
手工配置RID

[R1-bgp]peer 12.0.0.2 as-number 2
手工建立EBGP直连建邻（12.0.0.2是自己对端的IP，2是对端的AS）



[R2]bgp 2
[R2-bgp]router-id 2.2.2.2
[R2-bgp]peer 12.0.0.1 as-number 1
同样在R2上敲
```

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260927192549582.png" alt="image-20260927192549582" style="zoom:80%;" />













```
IBGP对等体的非直连建邻



[R2]bgp 2
[R2-bgp]router-id 2.2.2.2
[R2-bgp]peer 3.3.3.3 as-number 2

[R2-bgp]peer 3.3.3.3 connect-interface LoopBack 0
和3.3.3.3IP交互时选用loopback 0接口（主要用的IP地址）（详情可见open报文里的影响邻居关系建立的参数）



[R3]bgp 2
[R3-bgp]router-id 3.3.3.3
[R3-bgp]peer 2.2.2.2 as-number 2
[R3-bgp]peer 2.2.2.2 connect-interface LoopBack 0
```

注意：因为在一个AS内部会存在大量的备份链路，使用物理接口建立邻居，可能导致链路失效，所以尽量使用环回接口建立邻居关系，这样可以充分利用IGP内部的路由资源



用物理接口建 IBGP 的问题：

-   两台 IBGP 路由器之间常有多条备份链路
-   若用物理接口地址建邻居，主链路一断，TCP 连接断，IBGP 会话重置
-   引发 BGP 路由震荡、收敛慢

用环回接口建 IBGP 的优势：

-   环回接口永远 up（只要设备活着）
-   只要 IGP 能算出到达对方环回地址的路由，任意一条路径通，IBGP 会话就不断
-   主备链路切换对 BGP 透明，会话稳定









```
EBGP对等体的非直连建邻

[R4]bgp 2
[R4-bgp]router-id 4.4.4.4
[R4-bgp]peer 5.5.5.5 as-number 3
[R4-bgp]peer 5.5.5.5 connect-interface LoopBack 0

[R4-bgp]peer 5.5.5.5 ebgp-max-hop 255
将发给5.5.5.5IP的TTL值改为255



[R5]bgp 3
[R5-bgp]router-id 5.5.5.5
[R5-bgp]peer 4.4.4.4 as-number 2
[R5-bgp]peer 4.4.4.4 connect-interface LoopBack 0
[R5-bgp]peer 4.4.4.4 ebgp-max-hop 255

```

注意：在EBGP对等体非直连建邻之前，需要先保证IGP基础。（静态）

注意：因为EBGP对等体之间的TTL被设定为1，所以，EBGP对等体之间需要非直连建邻时，需要将TTL值改大









上面是建邻居，下面是发布路由



### 发布路由



#### network发布

```
通过network发布路由信息

[R1-bgp]network 1.1.1.1 24
```

注意：只要是路由表中存在的路由，都可以通过network进行发送。（也只能发送路由表中有的）

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260927204920412.png" alt="image-20260927204920412" style="zoom:67%;" />

下一跳是0.0.0.0，说明该路由是本地始发的



状态码：

​	*————可用————查看路由表看自己能否到达下一跳，可达就有这个符号，不可达就没这个符号，并且不参与BGP的优选

​	>————最优————所有到达相同目标网段的路由需要根据路径属性进行选举，属性最好的则为最优，就会贴上这个标签，只有同时拥有可用和最优才能传递和加路由表

​	I————代表该路由是从自己IBGP对等体处学来的。

​		我们知道，路由信息在AS内部会保持属性不变，但是这样里面的路由器就无法拥有到达边界IBGP的路由，这条命令`[R2-bgp]peer 3.3.3.3 next-hop-local`可以将发给3.3.3.3IP的数据包中的下一跳改为自己本地接口的地址，从而让AS里面的知道该怎么走

​	OGN————起源码——I——所有通过network发布的路由，起源码都是I

​					    e ---所有通过EGP协议导入到BGP中的路由，起源码为e

​					    ？---所有通过重发布导入的路由（其实是以上两种方法之外），起源码都是？

​	Path————As-Path————就是之前防环的链表

​	S --- suppressed ———— 抑制————自动聚合后将自动抑制明细路由，不在传递给自己的邻居。（因为有汇总被压下去了）





注意华为体系中，BGP协议的优先级设定为255





因为 **BGP 递归验证下一跳时，通常要求匹配到 32 位掩码的主机路由，而不是网段路由。**（BGP 递归下一跳时，要找一条能唯一确定这个下一跳地址的路由，通常就是 `/32` 主机路由。）









#### 重发布

```
通过import来批量发布路由


[r2-bgp]import-route ospf 1
```









#### 路由聚合

```
通过BGP的路由聚合发布
（只能针对重发布的路由进行聚合；只能按照主类进行聚合）

[r1]ip ip-prefix aa permit 172.16.0.0 16 greater-equal 24 less-equal 24
抓流量

[r1]route-policy aa permit node 10
[r1-route-policy]if-match ip-prefix aa
[r1-route-policy]qu
做路由策略

[r1-bgp]import-route direct route-policy aa
调用策略

[r1-bgp]summary automatic
开启自动汇总
```







<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260928092524892.png" alt="image-20260928092524892" style="zoom:67%;" />

注意：聚合后，设备上会自动生成一条指向汇总的空接口路由进行防环









```
[r1-bgp]aggregate 172.16.0.0 22
手工聚合
```



```
[r4-bgplaggregate 172.16.0.0 22 detail-suppressed
手工汇总时抑制所有子网段
```





```

[r4]ip ip-prefix sup permit 172.16.1.0 24
[r4]route-policy sup permit node 10
[r4-roate-policy]if-match ip-prefix sup
[r4-bgp]aggregate 172.16.0.0 22 suppress-policy sup
手工汇总时抑制0.0/22下的1.0/24网段（最后一句是抑制策略，逻辑是“被允许的抑制掉，没被允许的放通”，这也是路由策略里面不需要写放通所有的原因）
```

1，手工聚合后，不会自动抑制明细路由，导致路由条目的数量不减反增（所以需要上面代码框的第三种方法）

2，BGP的手工聚合可以在任意位置完成，导致聚合的路由可能会丟失一部分明细路由的属性（As-Path属性会丢失，可能导致环路出现）



可以看到，确实指定抑制了

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260928101423696.png" alt="image-20260928101423696" style="zoom:80%;" />









```
[r4-bgp]aggregate 172.16.0.0 22 as-set

```

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260928102828361.png" alt="image-20260928102828361" style="zoom:80%;" />



注意：如果明细路由来自于不同的AS中，在其他AS的设备上进行聚合时，激活了As-Path关键字，则汇总路由将同时携带不同明细AS_path中的AS号，需要使用大括号来括起来

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260928103924294.png" alt="image-20260928103924294" style="zoom:80%;" />









ATOMIC_AGGREGATE---纯粹的告警属性---当作手工聚合后，将所有的的明细路由抑制则会出现这个属性。
AGGREGATOR---聚合者---会标出聚合设备的RID以及所在AS

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260928110104299.png" alt="image-20260928110104299" style="zoom:67%;" />

```
另辟蹊径聚合

[r1]ip route-static 192.168.0.0 22 NULL 0
[r1-bgp]network 192.168.0.0 22
```









