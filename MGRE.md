三、MGRE (Multi Genric Routing Encapsulation)



## 简介
定义：多点通用路由封装协议，适合多个分公司需要和总部连接的情况

特点：通过构建公共隧道实现总部和分部、分部与分部之间的通信
	  所有私网中，有一方的公网地址必须固定，其他私网公网地址可以不固定（可以不是总部）





举例子：

![image-20260812194441713](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260812194441713.png)



MGRE VPN的封装方式是gre p2mp

```
[R1]interface Tunnel 0/0/0
创建GRE随道接口

[R1-Tunnel0/0/0]ip address 192.168.5.1 24
配置隧道IP地址

[R1-Tunnel0/0/0]tunnel-protocol gre p2mp 
定义封装方式

[R1-Tunnel0/0/0]source 15.0.0.1
自己隧道的IP


总部的
```

```
[R2]interface Tunnel 0/0/0
创建GRE随道接口

[R2-Tunnel0/0/0]ip address 192.168.5.2 24
配置隧道IP地址

[R2-Tunnel0/0/0]tunnel-protocol gre p2mp 
定义封装方式

[R2-Tunnel0/0/0]source GigabitEthernet 0/0/0
自己隧道的IP，因为分部的IP地址可能会换，所以用接口接入


分部的
```











想实现多对一，还需要用到NHRP协议

**2、NHRP协议——下一跳解析协议**

(1）工作原理

*   在私网中选择一个NHRP中心站点，其出口的公网IP必须是固定的：
*   NHRP中心站点要求所有分支都需要将自己物理公网接口IP和隧道IP发给中心站点。(发生变化就需要重新发送。)。
*   NHRP中心站点会将所有的分支的地址映射关系动态的记录在本地。发送信息时查询即可
*   分支之间需要发送信息也需要获取这个映射关系，就需要先问NHRP中心站点要。



```
[R1-Tunnel0/0/0]nhrp network-id 100
设置NHRP的编号，不同的隧道用的编号不同，这一个总部和三个分部用的是同一个编号


总部
```

```
[R1-Tunnel0/0/0]nhrp network-id 100
同上

[R2-Tunnel0/0/0]nhrp entry 192.168.5.1 15.0.0.1 register
写上总部的隧道IP和总部的公网IP，让分部去找总部来注册自己的信息


分部
```







**路由表**

不过需要注意的是，这和GRE不同的地方在于写路由表时，下一条地址不能写自己的出口路由，需要写目标区域所对应的隧道IP地址

```
[R1]ip route-static 192.168.2.0 24 192.168.5.2
```

















**查看**

```
[R1]display nhrp peer all
查看总部或者分部的NHRP邻居表
```

![image-20260813015932355](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260813015932355.png)

看一下总部的视图，会发现，上面写着NBMA，这不是之前学的无广播多点接入式网络吗

















以及：

![image-20260816032811203](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260816032811203.png)





