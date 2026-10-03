# Atlas 960E SuperPoD 与 UnifiedBus：2026-10 小白版

> 状态快照：2026-10-03。本文把“已经正式发布的产品/架构”和“未来路线图”分开写，避免把计划中的芯片当成今天已经可以买到的产品。

## 先记住一句话

传统服务器更像“很多台机器通过网络一起干活”；SuperPoD 的目标，是通过高带宽、低时延互联，让大量 CPU、NPU、内存和存储更像一个大的计算系统协同工作。

## 1. Atlas 960E SuperPoD 是什么

华为在 2026-09-17 的 HUAWEI CONNECT 2026 正式发布 Atlas 960E SuperPoD。

华为公开的产品数据包括：

- 单个 SuperPoD 最多扩展到 **4,096 个 NPU**。
- 标称计算能力：**8 EFLOPS FP8**、**16 EFLOPS FP4**。
- 使用 **UnifiedBus** 做统一互联。
- 使用 **Hi-ONE NPO（Near-Packaged Optics）光互联**。
- 采用全液冷和正交架构。
- Hi-ONE 单引擎传输能力为 **7.2 Tbit/s**。
- 华为称，相比传统 800G 光模块方案，可减少超过 550 kW 互联功耗，并给出 99.8% 系统可用性数据。

这些性能、功耗和可用性数字属于**厂商发布数据**，学习时可以用于理解产品定位，但不要把它们当成第三方独立 benchmark。

**状态：正式发布（2026-09-17）**。

官方来源：
https://www.huawei.com/en/news/2026/9/hc-ascend960-supernode

## 2. UnifiedBus 到底是什么

小白可以先把 UnifiedBus 理解为：

```text
CPU
 │
NPU ── UnifiedBus ── Memory
 │
SSD / Storage
```

它不是单纯的“交换机协议”，而是华为面向 SuperPoD / SuperCluster 的统一互联体系，目标是把原来分散的多类互联协议和内存语义进一步统一。

华为在 2026-09-17 公布的几个关键点：

- 统一协议与内存语义，支持 SuperPoD 内全局统一内存寻址。
- 直接互联 CPU、NPU、Memory、SSD。
- 支持 CPU 与 NPU 的异构协同。
- 支持分层存储与资源池化。
- 支持机柜内、机柜间、集群间的不同距离互联。

### 机柜内：LinkBlade

LinkBlade 主要解决机柜内部的高速互联，强调减少铜缆和连接损耗。

### 机柜间：LinkDevice

公开数据：

- **176 个端口**
- **每端口 1.6 Tbit/s**
- 总互联能力约 **280 Tbit/s**
- RTT 最低约 **2 微秒**

### 集群间：UBG

UnifiedBus UBG 交换设备面向更大规模 SuperCluster：

- radix fan-out 最高 **1,024**
- 华为给出的扩展目标可达百万 NPU 级

官方来源：
https://www.huawei.com/en/news/2026/9/hc-lingqu-agent-ai

## 3. UnifiedBus 和 RoCE 是什么关系

两者不要混为一个概念。

可以先这样理解：

```text
Scale-up（把一组计算资源做得更像“一台大机器”）
    → UnifiedBus / SuperPoD

Scale-out（把多个大计算单元继续组成更大集群）
    → UnifiedBus 或 RoCE / Clos / Multi-Rail
```

华为公开说明，多个 Atlas 960E SuperPoD 可以通过 UnifiedBus 或 RoCE 进一步组成 SuperCluster。

Atlas 960E 相关公开设计中：

- 两层、四平面的 Clos 架构可互联最高约 **512,000 个 NPU**
- 配合 multi-rail 拓扑，公开目标可到 **100 万 NPU**

这属于超大规模 AI 基础设施设计，不是初学阶段就要部署的内容。现阶段先理解 Scale-up 与 Scale-out 的区别即可。

## 4. 小规模也会用 UnifiedBus

UnifiedBus 不只用于几千卡的大集群。

华为还公布了一个更容易理解的例子：

- 两台 Atlas 650E 风冷服务器
- 通过 UnifiedBus 直接互联
- 不经过外部交换机即可组成 **16-NPU** 计算单元

这个例子很适合帮助理解 SuperPoD 的思想：先把少量服务器深度互联，再向更大的系统扩展。

## 5. Ascend 960 / 970 / 980：哪些还只是路线图

截至 2026-10-03，华为公开路线图中：

| 产品 | 当前应如何理解 |
|---|---|
| Atlas 960E SuperPoD | **正式发布**，2026-09-17 |
| Ascend 960DT | **路线图**，计划 2027 Q1 |
| Ascend 960PR | **路线图**，计划 2027 Q3 |
| Ascend 970 | **路线图**，计划 2028 |
| Ascend 980 | **路线图**，计划 2029 |

所以看到“Ascend 960/970/980”时，一定要先问：这是已发布产品、某个 SuperPoD 系统名称，还是芯片路线图？

路线图来源：
https://www.huawei.com/en/news/2026/9/hc-wang-keynote

## 6. 初学者现在该学什么

先按下面顺序：

1. NPU 是什么。
2. 一台服务器为什么会有多个 NPU。
3. 多 NPU 为什么需要高速互联。
4. Scale-up 与 Scale-out 的区别。
5. UnifiedBus 与 RoCE 分别更偏向解决什么问题。
6. 最后再学习 Clos、Multi-Rail、PFC/ECN、RDMA 和大规模故障域设计。

不要一开始就背 4,096 卡、512,000 卡、100 万卡这些数字；先理解架构层次更重要。
