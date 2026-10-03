# 2026-10-03 Ascend 生态增量更新

> 本文面向初学者，只记录已经核实的信息。华为官方发布、第三方生态项目和路线图内容分开说明。

## 1. UnifiedBus / Atlas SuperPoD

华为在 HUAWEI CONNECT 2026 发布 Atlas 960E SuperPoD，并介绍 UnifiedBus 架构。

初学者理解：

```text
单 NPU
  ↓
单服务器多 NPU
  ↓
SuperPoD（高速 Scale-up）
  ↓
SuperCluster（更大规模 Scale-out）
```

UnifiedBus 主要解决 CPU、NPU、内存和存储之间的大规模高速互联问题。

状态：

- Atlas 960E SuperPoD：正式发布
- UnifiedBus：正式发布架构
- 更大规模扩展能力：按厂商架构规划理解，不等同于所有规模均已商用

来源：Huawei HUAWEI CONNECT 2026 官方资料。

## 2. CANN / TorchNPU 版本学习原则

昇腾环境最容易出现的问题不是安装，而是版本混用。

必须记录：

```text
硬件型号：
OS：
Driver：
Firmware：
CANN：
Python：
PyTorch：
torch_npu：
上层框架：
```

不要采用：

```text
安装最新版 Driver
安装最新版 CANN
安装最新版 torch_npu
```

正确流程：

```text
确定硬件
 ↓
查官方兼容矩阵
 ↓
选择完整版本组合
 ↓
安装验证
```

## 3. vLLM Ascend 生态

vLLM Ascend 属于第三方开源生态，不是华为 CANN 本身。

学习时需要区分：

```text
大模型
 ↓
vLLM Ascend（推理框架适配）
 ↓
PyTorch + torch_npu
 ↓
CANN
 ↓
Ascend NPU
```

不同 release 会锁定不同版本组合，不建议单独升级其中一个组件。

## 4. DeepGEMM-Ascend

DeepGEMM-Ascend 属于 DeepSeek 开源生态项目。

定位：

- 高性能 GEMM 算子优化
- 面向 Ascend 平台适配
- 支持低精度计算场景

注意：

它不是 CANN 官方组件，也不能替代 CANN。

## 5. 学习路线调整

新增重点：

1. 先理解 Ascend / Atlas / NPU 关系
2. 掌握 CANN 软件栈
3. 学会 PyTorch + torch_npu 基础运行
4. 再学习 vLLM、MoE、分布式推理
5. 最后进入 SuperPoD、AI Fabric、存储和液冷

对于网络和系统集成人员，需要重点关注：

- AI Fabric
- RoCE / RDMA
- Clos 网络
- HCCL
- 存储网络
- 液冷和机柜设计
