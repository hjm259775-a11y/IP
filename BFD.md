# BFD



————双向转发检测————是一种用于快速检测，监控网络中链路或者IP路由转发联通情况的手段（其实就是个保活手段，可以救静态路由于水火，也能优化OSPF这种动态路由）







## BFD会话建立



Local Discriminator——本地标识符
Remote Discriminator——远端标识符





BFD会话的建立方式————1，静态会话————以上两个标识符需要手工配置



例子：A->B（LD——10，RD——20）（数字都是手工配置的）

​	  B->A（LD——20，RD——10）





​				   	2，动态会话————自动配置以上两个参数



例子：A->B（LD——X，RD——0）（同时发的）

​	  B->A（LD——Y，RD——0）（同时发的）

​	  A->B（LD——X，RD——Y）

​	  B->A（LD——Y，RD——X）













## BFD会话状态



Down

init

UP

admin Down（手工断开的）



<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260922215350781.png" alt="image-20260922215350781" style="zoom:67%;" />

1，R1和R2各自启动BFD，初始状态为down状态，发送状态为down的BFD报文

2，R2在DOWN状态下收到R1发来的DOWN包，R2就会从DOWN变成INIT，之后R2就会开始发送INIT的BFD报文，并且不再处理状态为DOWN的数据包

3，R1同理也变成INIT

4，R2在INIT状态下收到R1发来的INIT包，R2就会从INIT变成UP，之后R2就会开始发送UP的BFD报文

5，R1同理也变成UP







DOWN状态下收到报文：

​	1，收到DOWN状态的BFD报文，将状态切换为INIT

​	2，收到init状态的BFD报文，将状态直接切换为UP

​	3，收到UP状态的BFD报文，继续维持在DOWN状态（要么不是给我的，要么不是当前会话的）





INIT状态下收到报文：

​	1，收到DOWN状态的BFD报文，将维持在INIT状态

​	2，收到init状态的BFD报文，将状态直接切换为UP

​	3，收到UP状态的BFD报文，将状态直接切换为UP





UP状态下：

​	只有在链路出现故障或者手工关闭的情况下，切换为down状态。











## BFD数据包





### 组播

注意：BFD传输层使用的是UDP协议，目标端口号为3784，源端口号随机

单跳环境下可以使用组播发送BFD报文，默认的组播地址为224.0.0.184，会使用对应的组播MAC地址。



源端口：随机，目标端口：3784

源IP：自己接口IP，目标IP（组播）：224.0.0.184

源MAC：自己MAC，目标MAC（组播）：对应的组播MAC地址







<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260923104407504.png" alt="image-20260923104407504" style="zoom:80%;" />

*   最小 BFD 发送间隔

​	**图中数值**：1000 ms (1000000 us)

​	**含义**：这是本端设备向对端设备发送 BFD 控制报文时，期望使用的最小时间间隔。也就是本端“我最快能每隔 1000 毫秒发一个包”。

*   最小 BFD 接收间隔

​	**图中数值**：1000 ms (1000000 us)

​	**含义**：这是本端设备能够支持的、接收对端 BFD 控制报文的最小时间间隔。意思是“我要求你（对端）发给我的包，间隔不能小于 1000 毫秒，否则我可能处理不过来”







### 单播























## BFD配置





### 单跳检测场景

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260923110100543.png" alt="image-20260923110100543" style="zoom:67%;" />



1，单跳检测场景————单跳BFD是两边设备之间建立的会话，这两台设备是直连的



```
[R1]bfd 
[R1-bfd]qu
[R1]
激活BFD


[R1]bfd xgz bind peer-ip default-ip interface GigabitEthernet 0/0/0
创建BFD会话，本BFD名称为xgz（本地意义，就是个代号，对端甚至可以不一样），类型为peer-ip，因为是根据IP建立，default-ip代表组播发送，后面还需要跟上接口，也可以直接把default-ip interface GigabitEthernet 0/0/0换成12.0.0.2


[R1-bfd-session-xgz]discriminator local 10
[R1-bfd-session-xgz]discriminator remote 20
配置BFD会话


[R1-bfd-session-xgz]commit
激活配置
```



```
[R2]bfd 
[R2-bfd]qu
[R2]

[R2]bfd myn bind peer-ip default-ip interface GigabitEthernet 0/0/0
命名仅有本地意义

[R2-bfd-session-myn]discriminator local 20
[R2-bfd-session-myn]discriminator remote 10

[R2-bfd-session-myn]commit
```









查看其配置

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260923102945146.png" alt="image-20260923102945146" style="zoom:67%;" />









### 多跳检测场景

2，多跳检测场景————多跳BFD是两边设备之间建立的会话，但这两台设备不一定是直连的，中间可以隔着任意多台三层设备。

​	多跳只能采用IP地址

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260923110039173.png" alt="image-20260923110039173" style="zoom:50%;" />



```
[R6]bfd
[R6-bfd]qu
[R6]

[R6]bfd xgz bind peer-ip 45.0.0.4 
这次用IP地址建立单播

[R6-bfd-session-xgz]discriminator local 10
[R6-bfd-session-xgz]discriminator remote 20

[R6-bfd-session-xgz]commit

```



```
[R4]BFD
[R4-bfd]qu
[R4]
[R4]bfd myn bind peer-ip 56.0.0.6
[R4-bfd-session-myn]discriminator local 20
[R4-bfd-session-myn]discriminator remote 10
[R4-bfd-session-myn]commit
```











### 自动配置

**自动配置，也就是动态会话**

```
[R4]bfd
[R4-bfd]qu
[R4]

[R4]bfd vv bind peer-ip 56.0.0.6 source-ip 45.0.0.4 auto 
```



````
[R6]bfd
[R6-bfd]qu
[R6]

[R6]bfd ww bind peer-ip 45.0.0.4 source-ip 56.0.0.6 auto 
````

<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260923112511752.png" alt="image-20260923112511752" style="zoom:67%;" />









## BFD的检测模式



### 异步模式

——————两台设备均发送BFD报文分别进行保活（适用于二者路由器差不多配置的情况下使用）





### 回声模式



（单跳环境下使用）（适用于二者路由器配置性能相差较大的情况下使用，不会被配置低的设备拖累）

————————发送不是普通的BFD报文，而是BFD ECHO报文。



​	源IP：自己，目标IP：自己（这个报文比较特殊，能让这个包发出去）

​	源MAC：自己，目标MAC：对端



这样发给对端，对端解二层之后看目标IP，重新封装二层后又转发回去，自己收到了自己发的包，就能确定保活了，并且这个方法不受对方性能约束



​	被动回声模式————在两台设备启动异步模式的基础上，将发送的报文替换成为ECHO报文。

​	单臂回声模式————仅一台设备激活BFD，对端设备甚至都可以不激活BFD，

​	`[R4]bfd hhh bind peer-ip 56.0.0.6 interface GigabitEthernet 0/0/0 source-ip 45.0.0.4 one-arm-echo`这是单臂回声模式











## BFD和静态配置



 



<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260923152125108.png" alt="image-20260923152125108" style="zoom:67%;" />

```
[R1]bfd
[R1-bfd]qu
[R1]


[R1]ip route-static 0.0.0.0 0 12.0.0.2 track bfd-session xgz
再配完BFD后，配置静态路由时，需要追踪名字为xgz的bfd会话
```









## BFD和OSPF配置



```
[R1]bfd
[R1-bfd]qu
[R1]


[r1-ospf-1]bfd all-interfaces enable
让R1在该进程下所有运行 OSPF 的接口上，动态建立 BFD 会话（OSPF里面每个设备都敲一遍）
```









## BFD和VRRP配置



<img src="C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260923161923606.png" alt="image-20260923161923606" style="zoom:67%;" />

主设备是R1，备用设备是R2

配置完VRRP之后

```
[R1]bfd aaa bind peer-ip 192.168.1.2 source-ip 192.168.1.1 auto

[R2]bfd aaa bind peer-ip 192.168.1.1 source-ip 192.168.1.2 auto

让R1和R2下面两个接口建立BFD会话
```



```
[rl-bfd-session-aalmin-rx-interval 100
[rl-bfd-session-aa]min-tx-interval 100

[r2-bfd-session-aalmin-rx-interval 100
[r2-bfd-session-aa]min-tx-interval 100

修改接收发送间隔为100ms
```





```
[R1-GigabitEthernet0/0/0]vrrp vrid 10 track bfd-session session-name aaa increased 10

让接口视图下让VRRP 10去追踪名字是aaa的BFD会话，当 BFD 会话 down 时，则本接口优先级加10（火速抢占网关）
```







