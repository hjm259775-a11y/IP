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









## BFD配置





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









2，多跳检测场景————多跳BFD是两边设备之间建立的会话，但这两台设备不一定是直连的，中间可以隔着任意多台三层设备。

















