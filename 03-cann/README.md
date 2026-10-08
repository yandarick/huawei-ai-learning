# 03 · CANN 入门

CANN（Compute Architecture for Neural Networks）是学习昇腾最关键的软件层之一。

## 小白理解

可以先把软件路径记成：

```text
AI 应用
 ↓
PyTorch / MindSpore
 ↓
torch_npu / 框架适配
 ↓
CANN
 ↓
Driver + Firmware
 ↓
Ascend NPU
```

这只是帮助入门的分层示意，不代表所有组件之间都是简单的一对一调用关系。

## 必学组件

- Driver
- Firmware
- CANN Toolkit
- AscendCL
- HCCL
- 算子
- Profiling
- 日志与故障定位

## 最重要的运维意识

昇腾环境出现问题时，第一件事通常不是重装系统，而是确认：

**硬件型号 → OS → Driver → Firmware → CANN → Python → PyTorch → torch_npu → 模型**

这些版本是否匹配。

## HCCL 与 AllReduce 入门

> 资料核查：2026-10-08（UTC）。本节依据 CANN 社区版 8.5.0 的 HCCL 文档讲解通信语义，只做纸面推演；未进行昇腾 NPU 或多卡通信实测，不提供安装与启动命令。

### 1. 先确定谁参加通信

HCCL 是华为集合通信库。**官方说明：**集合通信由多个 NPU 共同参与，常用于梯度同步等场景；梯度可以先理解为训练时用于调整模型参数的一组数。参见[集合通信说明](https://www.hiascend.com/doc_center/source/en/CANNCommunityEdition/850/commlib/hcclug/hcclug_000009.html)（CANN 8.5.0，核查：2026-10-08）。

阅读下面的例子时，把**通信域（communicator）**理解为本次通信的一组参与成员，把 **rank** 理解为其中的成员及其编号。这里的“所有成员”仅指这个通信域，不是整个机房的所有设备。

**记录建议：**实际实验应分别记录具体 Atlas/Ascend 型号、服务器台数、每台安装及参与的 NPU 数、总设备数、进程与设备映射、通信域成员数。只有明确采用“一进程对应一个 NPU 设备、各设备各加入一次”的假设时，参与进程数、参与设备数和 rank 数才在这个模型中相等；rank 数本身不能说明服务器台数、端口数或链路数。本节不假定任何型号的每机配置。

### 2. AllReduce：按位置合并，每个成员都拿到结果

**官方事实：**AllReduce 对通信域内各 rank 的输入做归约，再将结果交给每个 rank。归约就是按指定规则合并相同位置的数，例如求和；规则由 `op` 指定。它与只把归约结果交给指定根成员的 Reduce 不同。依据同一份[集合通信说明](https://www.hiascend.com/doc_center/source/en/CANNCommunityEdition/850/commlib/hcclug/hcclug_000009.html)（核查：2026-10-08）。

**教学假设：**通信域内只有 rank 0、1、2 三个成员，各有一个包含两个 FP32 数值的输入，选择求和 `sum`。下表是自设数据的数学预期，不是设备输出，也不代表三台服务器。

| 成员 | 通信前的输入 | AllReduce 求和完成后的输出 |
|---|---|---|
| rank 0 | `[1, 2]` | `[9, 12]` |
| rank 1 | `[3, 4]` | `[9, 12]` |
| rank 2 | `[5, 6]` | `[9, 12]` |

按以下顺序手算：

1. 第一个位置：`1 + 3 + 5 = 9`。
2. 第二个位置：`2 + 4 + 6 = 12`。
3. 每个成员都获得 `[9, 12]`，输出仍为两个数，不是把输入拼成六个数。

选择 `sum` 时不会自动求平均。若这个纸面例子要计算三个输入的等权平均，还需把结果除以 `3`，得到 `[3, 4]`。真实训练中是否已经缩放梯度，应按所用框架确认，不能在看到 AllReduce 后又盲目除一次。

### 3. 看接口时核对什么

[官方 HcclAllReduce 接口](https://www.hiascend.com/document/detail/en/CANNCommunityEdition/850/API/hcclapiref/hcclcpp_07_0021.html)规定各 rank 的 `count`、`dataType` 和 `op` 必须一致（CANN 8.5.0，核查：2026-10-08）。本例对应 `count=2`、FP32、`sum`；`count` 是每个输入的数据元素数，不是 rank 数或服务器数。“每个 rank 一个输入”也不等于“每个输入只能有一个数”。

该接口页对产品支持单列限制，例如 A2 系列只列出 Atlas 800T A2 训练服务器、Atlas 900 A2 PoD 集群基础单元和 Atlas 200T A2 Box16 异构子框；这不能外推为所有名称带 A2 的产品都支持。实际运行还须按选定版本核对型号、数据类型、归约操作及整套软件配套。本节引用 8.5.0 的通信定义，不将它与[PyTorch 入门教程](../04-pytorch-on-ascend/README.md)中的 CANN 9.1.0 基线拼成安装组合。

### 官方来源与验证边界

以下两页均于 **2026-10-08** 读取正文，适用范围为 **CANN 社区版 8.5.0 / HCCL**；本次读取的页面正文均未提供发布日期或更新日期，核查日不代表版本发布日。

- [Collective Communication](https://www.hiascend.com/doc_center/source/en/CANNCommunityEdition/850/commlib/hcclug/hcclug_000009.html)：集合通信、AllReduce 与 Reduce 的语义。
- [HcclAllReduce](https://www.hiascend.com/document/detail/en/CANNCommunityEdition/850/API/hcclapiref/hcclcpp_07_0021.html)：参数含义、一致性要求及产品限制。

本例仅核对输入与输出的算术关系，不描述底层传输算法，不估算链路流量、耗时或带宽；也未验证通信初始化、数值精度、驱动/固件配套或多卡训练能力。
