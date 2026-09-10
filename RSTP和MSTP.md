

# RSTP和MSTP





802.1D生成树的缺点：

​	1，收敛速度慢

​	2，链路利用率低



PVST————基于VLAN的生成树协议（思科）————每一个VLAN一棵树，这榉可以解决链路利用率的问题，但是可能出现新的问题，就是如果VLAN过多，则可能导致维护树形结构的流量过大，导致资源浪费











## RSTP

802.1W--- RSTP---快速生成树协议（可以向下兼容802.1D生成树。）



**改进点一**：改变了原先的接口角色

------------------------

​	802.1D————根端口，指定端口，非指定端口

​	802.1W————根端口，指定端口，替代端口(Alternate），备份端口（Backup)



​	1，替代端口————相当于是根端口的备份，如果一个端口因为接收了其他设备的BPDU后导致被阻塞，则该端口将成为替代端口（就是在竞选指定端口和非指定端口的时候输了，成非指定端口了，这个就是替代端口）

​		在根端口发生故障的时候，该端口将直接变成根端口，并且进入转发状态。（为了加快收敛）

​		如果存在多个端口，则将选择所有替代端口中参数最好的，直接成为根端口，进入到转发状态。

​	2，备份端口————相当于指定端口的备份，如果一个端口因为接收了自己设备的BPDU后导致被阻塞，则该端口将成为备份端口

​		如果指定端口出现故障，则备份端口将直接成为指定端口，并进入到转发状态。



<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260909194808709.png" alt="image-20260909194808709" style="zoom:67%;" />

可以看到，要是左边的指定端口坏了，右边原本被堵塞的就会成为指定端口开始转发





**改进点二**：改变了接口的状态

------------------------

​	802.1D：禁用，阻塞，侦听，学习，转发

​	802.1W：Discarding（丢弃状态）————不能转发业务数据帧，也不能学习MAC地址

​			学习————不能转发业务数据帧，但能学习MAC地址

​			转发————既能转发业务数据帧，也能学习MAC地址





**改进点三**：针对配置BPDU进行了改进

------------------

![image-20260909204712619](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260909204712619.png)

RST BPDU---0x02

P/A机制————RSTP加速收敛的核心机制（Agreement和Proposal）————收敛过程不再依赖计时器，角色一旦确定，则可以立马进入到转发状态。（根网桥确定后，从根网桥开始，觉得选举变为两两之间，只要两两之间一旦确定角色之后，其余剩余的所有接口会被置于Discarding状态，这个状态不会转发，只会在那悄悄的继续协商，这叫同步模式）

​	指定端口会向根端口发送一个RST BPDU（Proposal置为1），代表请求，根端口收到后，会将其余剩余端口置于Discarding状态（避免后面的网络出现临时环路），然后将根端口的状态置为转发状态，然后给指定端口发送一个RST BPDU（Proposal置为1），代表同意，指定接口收到之后，也会进入转发状态，后面的以此类推







**改进点四**：对配置BPDU的处理进行了优化

------------------

​	802.1D中---收敛完成后，仅根网桥会周期性的发送配置BPDU，其他设备仅转发

​	802.1W中--- 收敛完成后，所有指定接口均主动发送根网桥的配置BPDU，周期2S一次



​	802.1D中---失效判断时间---默认20S

​	802.1W中---如果设备在超时时间内没有收到配置BPDU，则认为邻居失效，超时实际按为2倍的周期时间，默认6S。





**改进点五**：快速收敛机制

------------------

​	1，替代端口和备份端口的快速切换

​	2，P/A机制

​	3，边缘接口---交换机连接终端的接口，也需要参与到生成树的选举当中，但是，这些接口角色固定，并且，因为连接终端设备，所以，不会出现临时环路，可以直接进入转发状态。需要通过手工将这些接口配置成为边缘接口则可以实现立即进入转发状态的效果。

```
[Huawei-GigabitEthernet0/0/3]stp edged-port enable
将此接口设置为边缘接口
```



```
[Huawei-GigabitEthernet0/0/3]stp bpdu-filter enable
开启BPDU的过滤，让此接口不再周期性的发送
```





注意：因为边缘接口是手工配置，为了防止边缘接口计入到其他交换网络中，导致出现临时环路问题，所以，可以开启BPDU的保护功能(如果边缘接口接收到了配置BPQU，则将关闭边缘接口)

```
[Huawei]stp bpdu-protection

开启保护功能，在收到BPDU时会将边缘接口打开
```











**改进点六**：拓扑变更机制的改进

------------------



<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260909223627037.png" alt="image-20260909223627037" style="zoom:67%;" />













## MSTP



802.1S——MSTP——多生成树协议（向下兼容RSTP和STP）





**Instance**——实例——一个VLAN或者多个VLAN的集合，一个instance一棵树

​	Insatnceid---12位二进制（0-4094）--- instance 0具有特殊含义---华为设备默认存在instance 0，所有的VLAN一开始都属于instance o。

​		在BID中，优先级占据16位，但实际只使用了前四位，后面的12位称为拓展系统ID，在802.1S中，用来携带instanceid，区分不同树的配置BPDU。





**region**————域————MST域————如果一个交换网络规模过大，可以划分成为多个MST域分别维护树形结构，当然，如果一个交换网络规模适中，则也可以只有一个MAST 域。

设备划分到同一个MST域中时，需要保证一下三个参数完全相同

​	1，region name域名（域名可以自定义）

​	2，revsion level修订等级（同一区域所有设备的这个数字需要一样）

​	3，instance和vlan的影射关系（每个设备所记的vlan和instance对应列表需要一致）







## 配置MSTP



<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260910133030792.png" alt="image-20260910133030792" style="zoom:67%;" />





交换网络存在vLan 1-10

1，vlan 1-5映射到instance1

2，Vlan 6-10 影射到instance2

3，sw1成为instance1的主根，instance2的备份根。

4，SW2作为instonce2的主根，instance1的备份根



```
[SW1]vlan batch 1 to 10
[SW1]port-group group-member GigabitEthernet 0/0/1 GigabitEthernet 0/0/2
[SW1-port-group]port link-type trunk
[SW1-port-group]port trunk allow-pass vlan 1 to 10

[SW1]stp enable
[SW1]stp mode mstp

[SW1]stp region-configuration
进入region
[SW1-mst-region]region-name xgz
修改域名
[SW1-mst-region]revision-level 1                                   INTEGER<0-65535>  Revision level
修改修订等级
[SW1-mst-region]instance 1 vlan 1 to 5
[SW1-mst-region]instance 2 vlan 6 to 10
修改映射关系

[SW1-mst-region]active region-configuration
此命令可以激活以上命令，重要
```

其他设备都一样





```
[SW1-mst-region]display stp region-configuration
查看当前设备的MST信息
```

![image-20260910135113198](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260910135113198.png)

注意：在没有任何操作时，设备默认使用MAC地址作为MST域名，修订等级默认为0。

敲完命令之后：

![image-20260910153144701](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260910153144701.png)

```
[SW2-mst-region]display stp brief 
 MSTID  Port                        Role  STP State     Protection
   0    GigabitEthernet0/0/1        DESI  FORWARDING      NONE
   0    GigabitEthernet0/0/2        DESI  FORWARDING      NONE
   1    GigabitEthernet0/0/1        DESI  FORWARDING      NONE
   1    GigabitEthernet0/0/2        DESI  FORWARDING      NONE
   2    GigabitEthernet0/0/1        DESI  FORWARDING      NONE
   2    GigabitEthernet0/0/2        DESI  FORWARDING      NONE
   
   
 [SW3]display stp brief 
 MSTID  Port                        Role  STP State     Protection
   0    GigabitEthernet0/0/1        ALTE  DISCARDING      NONE
   0    GigabitEthernet0/0/2        ROOT  FORWARDING      NONE
   1    GigabitEthernet0/0/1        ALTE  DISCARDING      NONE
   1    GigabitEthernet0/0/2        ROOT  FORWARDING      NONE
   2    GigabitEthernet0/0/1        ALTE  DISCARDING      NONE
   2    GigabitEthernet0/0/2        ROOT  FORWARDING      NONE
   
<SW1>display stp brief 
 MSTID  Port                        Role  STP State     Protection
   0    GigabitEthernet0/0/1        DESI  FORWARDING      NONE
   0    GigabitEthernet0/0/2        ROOT  FORWARDING      NONE
   1    GigabitEthernet0/0/1        DESI  FORWARDING      NONE
   1    GigabitEthernet0/0/2        ROOT  FORWARDING      NONE
   2    GigabitEthernet0/0/1        DESI  FORWARDING      NONE
   2    GigabitEthernet0/0/2        ROOT  FORWARDING      NONE   
   
   
   可以看到，instance 1和instance 2选的角色都是一样的，所以需要干涉选举
```



干涉选举：

```
[SW1]stp instance 1 root primary
让SW1成为instance 1区域的主根

[SW3]stp instance 2 root secondary
让SW3成为instance 2区域的副根
```



















port-group group-member GigabitEthernet 0/0/1 GigabitEthernet 0/0/2
port link-type trunk
port trunk allow-pass vlan 1 to 10

qu

stp enable
stp mode mstp

stp region-configuration

region-name xgz

revision-level 1

instance 1 vlan 1 to 5
instance 2 vlan 6 to 10

active region-configuration
