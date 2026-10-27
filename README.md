# 华为 HCIP 网络进阶与 eNSP 实战笔记

这是我个人在学习华为认证（HCIP-Datacom）进阶网络技术过程中的理论与实验记录。

> **核心亮点**：本仓库中涉及的所有网络高级特性、协议及复杂拓扑，**均已在华为 eNSP 模拟器中完成环境搭建、配置下发与全网连通性测试。**

## 🗂️ 目录导航

### 一、 交换技术进阶 (Switching)
- [VLAN.md](./VLAN.md) - 虚拟局域网进阶应用
- [STP.md](./STP.md) - 生成树协议基础与防环
- [RSTP和MSTP.md](./RSTP和MSTP.md) - 快速生成树与多实例生成树
- [链路聚合.md](./链路聚合.md) - Eth-Trunk 链路带宽扩容与冗余
- [VRRP.md](./VRRP.md) - 虚拟路由器冗余协议（网关高可用）

### 二、 路由与广域网技术 (Routing & WAN)
- [OSPF.md](./OSPF.md) - 开放最短路径优先协议深度解析
- [OSPF IA整理.md](./OSPF%20IA整理.md) - OSPF 区域间（Inter-Area）路由实验
- [路由策略.md](./路由策略.md) - 策略路由与流量控制
- [PPPOE.md](./PPPOE.md) - 宽带拨号技术配置

### 三、 隧道与虚拟专用网 (VPN & Tunnels)
- [VPN简介和GRE.md](./VPN简介和GRE.md) - GRE 隧道基础
- [MGRE.md](./MGRE.md) - 多点 GRE 隧道与 NHRP 协议

### 四、 底层与高级架构
- [网络类型及数据链路层协议.md](./网络类型及数据链路层协议.md) - 底层链路特性
- [MPLS.md](./MPLS.md) - 多协议标签交换技术基础

## 💻 实验环境与工具

- **模拟器**：华为 eNSP (Enterprise Network Simulation Platform)
- **抓包分析**：Wireshark
- **涉及技术栈**：OSPF, MPLS, MGRE, PPPoE, STP/RSTP/MSTP, VRRP, 路由策略等。

## 🎯 实战目标

- 掌握中大型企业网络的拓扑规划与底层链路打通。
- 熟练配置多区域 OSPF 及路由策略，实现复杂的流量调度。
- 利用 VRRP 和 STP 构建高可靠的园区网架构。
- 掌握 MPLS 及隧道技术，理解企业广域网互联的底层逻辑。

---
