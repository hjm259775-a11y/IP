# VLAN



IA学的那一套是思科的配置

现在我们来学习下华为的，当然，二者做出的效果是一样的



之前有access和trunk接口，现在我们添加：

​	**hybrid————混杂接口**（华为设备所有接口的默认类型为hybrid）



先来浅浅查看下：

![image-20260906153007559](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260906153007559.png)

==PVID==---接口所属的Vlan  ID————华为设备所有接口默认的PVID都是1

我们之前学的是当数据转发的别的广播域的时候才需要带上标签，现在是**华为交换机内部所有的数据帧都必须带标签，如果没有标签的数据帧进入到交换机中，要打上该接口PVID对应VID的标签**



==VLAN List==————允许列表————接口允许放通的VLAN ID，所有接口默认允许放通VLAN 1



==U/T标记==————如果数据发出到链路时，需要根据UT标记判断是否带标签

​	U——不带标签发出

​	T——带标签发出

注意：TRUNK接口，在放通了和PVID相同的VLAN时，发出到链路时不带标签。（即PVID和VLAN List相等时，发出不带标签）





## 具体流程

ACCESS

​	1，如果从链路上接收到一个untagged帧，**其实就是收到终端给交换机发送数据时**，先给数据帧打上接口PVID对应VID的标签，之后，查看VLAN LIST，如果VLAN LIST中存在对应的VID，则允许转发该数据帧。因为ACCESS接口默认放通PVID对应的VID，所以，必然可以进行转发

​	2，如果从交换机的其他接口接收到一个tagged帧，**其实就是第一种情况所出现的数据帧要进行转发的时候**，因为已经存在标签，则不需要再打标签，之后要看允许列表是否允许该VID的数据帧通过，如果允许通过，还需要看此接口的U/T标记，因为access是U，所以还需要剥离标签后发出。要是允许列表没有该VID，就直接丢弃，不转发

​	3，如果从链路上接收到一个tagged帧，这种情况很少，因为已经存在标签，所以不需要打标签，直接观察允许列表，是否放通VID，如果允许列表中存在，则直接转发；如果不存在，则直接丢弃该数据帧，不转发。（就是第一种情况操作把打标签去掉）



TRUNK

​	1，如果从链路上接收到一个tagged帧，**其实就是收到别的交换机发来的数据帧**，因为已经存在标签，所以不需要打标签，直接观察允许列表，是否放通VID，如果允许列表中存在，则直接转发；如果不存在，则直接丢弃该数据帧，不转发。

​	2，如果从交换机的其他接口接收到一个数据帧，**就是要转发的时候**，因为已经存在标签，则不需要再打标签，之后要看允许列表是否允许该VID的数据帧通过，如果允许通过，还需要看此接口的U/T标记，因为是trunk接口，大多数要带标签发出，==除非PVID和VLAN List相同时，发出时不带标签==

​	3，如果从链路上接收到一个untagged帧，先给数据帧打上接口PVID对应VID的标签，之后，查看VLAN LIST，如果VLAN LIST中存在对应的VID，则允许转发该数据帧。如果不存在，就丢弃





==**VLAN工作逻辑**==

说了这么多，其实很简单，总结一下：

​	一个数据帧传进交换机时，查看其是否有标签，有就不管，没有就把当前进入的接口的PVID的标签，最后查看允许列表，没有的话直接丢弃。

​	交换机要对一个数据帧进行转发的时，看出去的这个接口的允许列表是否有这个数据帧的VID，有就可以从这个接口转发，没有就丢弃，最后还要看U/T值，U就把标签剥下来，T就带上原标签





所以，不同的接口类型其实就是在定制VLAN通行表时的权限不同：

​	ACCESS ———— 只能修改PVID，允许列表自动匹配PVID且仅允许通过一个VLAN，不能修改UT标记

​	Trunk ———— 可以需改PVID，可以修改允许列表，且可以放通多个VLAN，不能修改U/T标记

​	Hybrid ———— 可以需改PVID，可以修改允许列表，且可以放通多个VLAN，也可以修改U/T标记



你可能会问，有这个Hybrid有什么用

![image-20260906204525586](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260906204525586.png)

以如图所示配置，就可以实现PC8和PC9无法通信，发送数据帧时会因为允许列表没有对应数字而被拦下



你可能会问，那我用trunk不行吗🤓☝️，还真不行，首先要知道，发送给终端的数据帧是不能带标签的，不然会丢弃，而trunk的U只有一个名额可以不带标签，但是你要发给终端的也许是各种VID的数据帧，所以必须用hy来进行自定义U/T标记



添：最小VLAN透传原则：**交换机接口的允许列表中，只放通必须通过的VLAN，不放通任何多余的VLAN。**

## 配置



![image-20260906231015939](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260906231015939.png)

实现PC2无法和PC3通信，从左到右分别用的VLAN 2 3 4





```
[LW1]vlan batch 2 3 4
[LW1]interface GigabitEthernet 0/0/1

[LW1-GigabitEthernet0/0/1]port link-type hybrid 
接口类型

[LW1-GigabitEthernet0/0/1]port hybrid pvid vlan 2
修改PVID

[LW1-GigabitEthernet0/0/1]port hybrid untagged vlan 2 3 4
设置U的VLAN

[LW1-GigabitEthernet0/0/1]undo port hybrid untagged vlan 1
删除hy的U配置里的vlan（遵循最小VLAN透传原则）



[LW1]interface GigabitEthernet 0/0/2
[LW1-GigabitEthernet0/0/2]port link-type hybrid 
[LW1-GigabitEthernet0/0/2]port hybrid pvid vlan 3
[LW1-GigabitEthernet0/0/2]port hybrid untagged vlan 2 3
因为不想和PC3交流，所以只允许1和2
[LW1-GigabitEthernet0/0/2]undo port hybrid untagged vlan 1



[LW1]interface GigabitEthernet 0/0/3
[LW1-GigabitEthernet0/0/3]port link-type trunk 
[LW1-GigabitEthernet0/0/3]port trunk allow-pass vlan 2 3 4
[LW1-GigabitEthernet0/0/3]undo port trunk allow-pass vlan 1


```







```
到目前为止
[LW1-GigabitEthernet0/0/3]display port vlan active 
T=TAG U=UNTAG
-------------------------------------------------------------------------------
Port                Link Type    PVID    VLAN List
-------------------------------------------------------------------------------
GE0/0/1             hybrid       2       U: 2 to 4
GE0/0/2             hybrid       3       U: 2 to 3
GE0/0/3             trunk        1       T: 2 to 4
```



LW2就不写了，这也不是实验报告，和之前一样的









顺便一提，要是做VLAN间路由的话，多臂路由时交换机和路由器间的链路的VLAN类型是access，单臂路由是trunk（因为单臂路由不需要路由器识别802.1Q的标签）





------------------------

拓展配置：

```
[SW2-GigabitEthernet0/0/3]undo port trunk allow-pass vlan 1
不允许VLAN 1通过
```

![image-20260906162334100](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260906162334100.png)







```
[LW1-GigabitEthernet0/0/1]undo port hybrid untagged vlan 4
删除hy的U配置里的vlan
```









```
[SW2-GigabitEthernet0/0/3]port trunk pvid vlan 2
修改Trunk接口的PVID
```







```
[SW2]display port vlan active
查看VLAN
```

```
[SW1-GigabitEthernet0/0/6]undo port default vlan
让本接口的VLAN恢复默认
```





------------------------



下面我们来讲交换机的三层接口————SVI接口————也叫VLANIF接口（是交换机的虚拟接口）



**二层交换机**，为了远程登陆方便管理，所以需要一个IP地址，自然，只需要一个VLANIF接口管理VLAN————配置VLANIF接口对应的VLAN



**三层交换机**每一个VLAN都可以创建一个VLANIF接口，并且，三层交换机具有路由表，所以，很适合成为网关设备







![image-20260907133958974](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260907133958974.png)

```
[LW1]interface Vlanif 2
[LW1-Vlanif2]ip address 192.168.1.1 24

[LW1]interface Vlanif 3
[LW1-Vlanif3]ip address 192.168.2.1 24

配置VLANIF接口，每个VLAN对应一个VLANIF接口（没有对应VLAN是没法创建VLANIF的）
配置IP地址作为网关
```



```
[LW1-Vlanif4]display ip interface brief

Interface                         IP Address/Mask      Physical   Protocol  
MEth0/0/1                         unassigned           down       down      
NULL0                             unassigned           up         up(s)     
Vlanif1                           unassigned           up         down      
Vlanif2                           192.168.1.1/24       up         up        
Vlanif3                           192.168.2.1/24       up         up        
Vlanif4                           192.168.100.1/24     down       down
可以看到，就算有了vlan和IP地址，没有实际的VLAN 4也是全down的




[LW1]display ip routing-table

Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface
      127.0.0.0/8   Direct  0    0           D   127.0.0.1       InLoopBack0
      127.0.0.1/32  Direct  0    0           D   127.0.0.1       InLoopBack0
    192.168.1.0/24  Direct  0    0           D   192.168.1.1     Vlanif2
    192.168.1.1/32  Direct  0    0           D   127.0.0.1       Vlanif2
    192.168.2.0/24  Direct  0    0           D   192.168.2.1     Vlanif3                                         &
    192.168.2.1/32  Direct  0    0           D   127.0.0.1       Vlanif3
可以看到，交换机也有路由表了
```



每一个VLANIF都有一个自己的MAC地址



**来模拟一下跨VLAN的通信过程：**

​	PC1发送数据想给PC2

​		SIP：192.168.1.10，DIP：192.168.2.10

​		SMAC：PC1mac，DMAC：网关（VLANIF 2的mac）

​	数据发送到LW1：

​		会先打上VID为2的标签，之后查看DMAC，发现是给自己的，就解封装，同时也会把VID 2解掉（没错，刚打上就解掉了🤪），之后查看自己本地路由表&，发现2.0对应的是VLANIF 3接口，对应的物理接口分别是0/0/3和0/0/4，会在这些接口都发出以下，当然，trunk发出去时还要加上标签VID 3

​		SIP：192.168.1.10，DIP：192.168.2.10

​		SMAC：VLANIF 3网关，DMAC：PC3mac

​	然后就把数据发到PC3了







好，到目前为止，这些是三层交换机干的活，就是代替了之前IA里面最上面的路由器，三层交换机不需要建立子接口，像上面一样创建VLANIF和IP地址即可

如图：

![image-20260907200933557](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260907200933557.png)

LW5和LW4之间是trunk接口

详情可见VLAN2文件夹





