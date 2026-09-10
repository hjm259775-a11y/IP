# 链路聚合和VRRP





我们知道，交换机出现环路运行stp协议的话就会堵塞其中一个端口，但是我想让两条链路同时分担怎么办

## 链路聚合



逻辑上将多个物理接口（成员接口）抽象成为一个聚合接口（ETH-TRUNK接口），相当于将多条物理链路（成员链路）抽象成为一条逻辑链路（Eth-trunk链路）————链路聚合技术。



​	链路聚合的条件：

​		1，聚合接口必须具有相同的传输速率，双工类型，接口类型（ACCESS,TRUNK），PVID以及允许列表。

​		2，聚合链路的两端必须分别在同一台设备（不允许跨设备聚合，就是不许尿分叉）





### 配置



![image-20260910165458970](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260910165458970.png)

```
[SW2]interface Eth-Trunk 0
创建聚合接口

[SW2-Eth-Trunk0]trunkport GigabitEthernet 0/0/1 0/0/2
划入物理接口
```



```
[SW1]interface GigabitEthernet 0/0/1
[SW1-GigabitEthernet0/0/1]eth-trunk 0

[SW1]interface GigabitEthernet 0/0/2
[SW1-GigabitEthernet0/0/2]eth-trunk 0
这样也可以
```







```
[SW2-Eth-Trunk0]display eth-trunk 
查看
```

![image-20260910170058828](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260910170058828.png)



可以看到，交换机已经将这两个看成一条eth-trunk了

![image-20260910170234094](C:\Users\xgz24\AppData\Roaming\Typora\typora-user-images\image-20260910170234094.png)





```
[swl-GigabitEthernet0/0/1]undo eth-trunk
把此接口从eth-trunk拉出来
```





注意：华为设备为了保证聚合的成员接口配置完全相同，需要在做配置之前先进行聚合，再完成配置。













