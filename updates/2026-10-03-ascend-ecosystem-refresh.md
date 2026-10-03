# 2026-10-03 Ascend 生态增量更新

> 本文面向初学者。资料核对日期：2026-10-03（UTC）。华为官方发布、第三方生态项目和学习建议分开说明；动态文档中的版本信息应在实际部署前再次核对。

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
- UnifiedBus：Atlas 960E SuperPoD 采用的互联架构；此处不将 2026 年的产品发布表述为 UnifiedBus 首次发布
- 更大规模扩展能力：按厂商架构规划理解，不等同于所有规模均已商用

来源：[华为官方：Atlas 960E SuperPoD 发布，2026-09-17](https://www.huawei.com/en/news/2026/9/hc-ascend960-supernode)。该文明确介绍 UnifiedBus、统一内存寻址和单 SuperPoD 最高 4,096 NPU 的厂商设计能力；产品发布不等于所有规模已经商用。

## 2. CANN / TorchNPU 版本学习原则

昇腾环境中的一个重要风险是版本混用。应按目标硬件和上层框架选择完整兼容组合。

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

截至 2026-10-03，本次核对的 [vLLM Ascend v0.23.0 发布说明](https://github.com/vllm-project/vllm-ascend/releases/tag/v0.23.0)（发布于 2026-08-16）列出：上游 vLLM 0.23.0、Python >=3.10 且 <3.13、PyTorch 2.10.0、torch_npu 2.10.0.post4；A2、A3 和 Ascend 950 对应 CANN 9.1.0、Triton Ascend 3.2.2。Atlas 300I DUO 必须查看专用安装指南，不能直接照搬这组硬件配套，且该发布说明不支持其使用 Triton Ascend。

[项目官方安装文档](https://docs.vllm.ai/projects/ascend/en/main/getting_started/installation.html)也强调整套兼容验证；其中 Python 3.12 是发布镜像的验证版本，不等同于源码包唯一允许的 Python 版本。这里是指定版本的核对结果，不表示 v0.23.0 是最新发布，也不能外推到其他 release。

仓库旧文章中的版本组合尚需按各自 release 重新复核；不能把旧表与本节组件混装。

## 4. DeepGEMM-Ascend

DeepGEMM-Ascend 属于 DeepSeek 开源生态项目。

定位：

- 高性能 GEMM 算子优化
- 面向 Ascend 平台适配
- 支持低精度计算场景

注意：

它不是 CANN 官方组件，也不能替代 CANN。

来源：[DeepSeek 官方 DeepGEMM-Ascend README](https://github.com/deepseek-ai/DeepGEMM-Ascend/blob/main/README.md)（2026-10-03 核对）。README 将其定位为 Ascend NPU 算子库，说明在 Ascend 950 系列开发和验证，支持 BF16、FP8、FP4 等计算场景，并明确依赖 CANN 和 torch_npu；其他硬件的兼容性不能仅凭“面向 Ascend”推定。

## 5. 学习路线调整

以下为学习顺序建议，不是厂商发布事实：

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
