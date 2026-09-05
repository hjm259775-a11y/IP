**PPPOE技术**

（PPPover Ethernet，以太承载PPP协议）



首先，我们回顾下以太网帧协议

​	1、以太网接入技术无法提供用户身份

​	2、验证以太网接入需要实现给用户自动分配公网IP地址

​	3、以太网接入受到双绞线距离限制



但是PPP协议可以给用户自动分配公网IP地址，所以需要在以太网的基础之上承载PPP（可能有人会问为什么不用DHCP，因为性价比不高☝️🤓）。







PPPoE协议采用C/S方式，将PPP报文封装在以太网帧之内，使PPP帧可以在以太网上进行传输，同时让以太网可以具备PPP功能，在以太网上提供点到点的连接

![image-20260807052254287](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260807052254287.png)







**PPPOE工作的三个阶段：**

*   1、Discovery阶段：协商PPPoE的seession-ID，用来区分不同的逻辑点

    ![image-20260807054248110](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260807054248110.png)

    (1）由客户端向服务器端**广播**发送PAD报文，询问PPPoE服务器

    ​	PADI (PPPOE Active Discovery Initiation)

    (2）PPPoE服务器收到消息后，**单播**回复一个PADO，告诉客户端由自己给其提供服务

    ​	PADO(PPPOE Active DiscoVery Offer)

    (3）客户端**单播**发送PADR报文请求PPPoE服务器端提供服务

    ​	PADR (PPPOE Active Discovery Request)

    (4）服务器**单播**发送PADS报文同意建立会话连接---sessionid实现

    ​	PADS （PPPOE Active Discovery Session-Confirmation包含session ID信息)

*   2、ppp session协商阶段：在PPPoE会话中进行ppp协商
    	LCP协商
    	身份验证
    	NCP协商

+   3、PPPOE会话终结，PPPoE断开
    当PPPoE客户端希望关闭连接时，会向PPPoE服务器端发送一个PADT（PPPoEActive DiscoVery Terminate）报文，用于关闭连接。同样，如果PPPoE服务器端希望关闭连接时，也会向PPPoE客户端发送一个PADT报文。





**PPPOE配置：**

![image-20260807060455080](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260807060455080.png)



服务器：

```
[PPPOS server]aaa
[PPPOS server-aaa]local-user xuaner password cipher xgz123
[PPPOS server-aaa]local-user xuaner service-type ppp 
创建用于拨号验证的用户，服务类型为PPP

[PPPOS server]interface Virtual-Template 1
[PPPOS server-Virtual-Template1]ip address 100.1.1.2 24
创建并配置VT(Vinual-Template，虚拟模版，虚拟接口)，一个虚拟接口就得面对一个私网的接口

[PPPOS server-Virtual-Template1]remote address pool chi
让这个虚拟模板调用名叫chi的地址池

[PPPOS server]ip pool chi
[PPPOS server-ip-pool-chi]network 100.1.1.0 mask 24
配置地址池，哪怕上面先声明用这个池子，我后面再配也不迟

[PPPOS server]interface GigabitEthernet 0/0/0
[PPPOS server-GigabitEthernet0/0/0]pppoe-server bind virtual-template 1
将virtual-template 1与GigabitEthernet 0/0/0做绑定，在以太网接口启动PPPoE·Server功能，毕竟数据都是物理接口传过来的
```





客户端：

```
[PPPOE client]interface Dialer 1
创建虚拟拨号接口

[PPPOE client-Dialer1]dialer user xuaner
指定对端用户名

[PPPOE client-Dialer1]dialer bundle 1
此处的数字必须和上面一致

[PPPOE client-Dialer1]ppp chap user xuaner
[PPPOE client-Dialer1]ppp chap password cipher xgz123
配置本地作为被认证方，以及账号密码信息

[PPPOE client-Dialer1]ip address ppp-negotiate 
配置由对端分配IP地址

[PPPOE client-Dialer1]dialer timer idle 0
设置拨号空闲时间为0，就是一点不许停


[PPPOE client]interface GigabitEthernet 0/0/0
[PPPOE client-GigabitEthernet0/0/0]pppoe-client dial-bundle-number 1
将上面设置的虚拟拨号接口应用到物理接口上
```



将军莫虑，且看此图：

![image-20260807071429190](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260807071429190.png)

Dialer1获得了IP地址





配置客户端访问公网的NAT

```
[PPPOE client]acl 2000
[PPPOE client-acl-basic-2000]rule permit source 192.168.1.0 0.0.0.255
[PPPOE client-acl-basic-2000]qu

[PPPOE client]interface Dialer 1
[PPPOE client-Dialer1]nat outbound 2000
要把数据发给拨号接口，你也看到了，物理接口没有IP，逻辑连接也仅仅只是虚拟拨号接口和服务器虚拟模板的连接
```



当然，别忘了缺省

````
[PPPOE client]ip route-static 0.0.0.0 0 Dialer 1
````









查看：

```
[PPPOE client]display pppoe-client session summary
查看客户端

[PPPOS server]display pppoe-server session all
查看服务器
```

![image-20260807073127031](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260807073127031.png)

![image-20260807073234963](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260807073234963.png)



