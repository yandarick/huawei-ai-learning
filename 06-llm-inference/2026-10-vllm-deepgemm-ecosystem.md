# 昇腾大模型推理生态：vLLM Ascend、Triton-Ascend、DeepGEMM-Ascend

> 状态快照：2026-10-03。本文中的 vLLM、Triton、DeepSeek 项目属于昇腾生态的重要开源项目，但不是华为产品发布本身；这里作为生态补充记录。

## 1. vLLM Ascend 当前稳定线

vLLM Ascend 项目在 2026-09-21 发布 **v0.23.0.post1**，属于 v0.23.0 正式版本之后的修复型 post release。

v0.23.0 的依赖组合明确列出：

| 组件 | vLLM Ascend 0.23.0 |
|---|---|
| vLLM | 0.23.0 |
| Python | >= 3.10，< 3.13 |
| CANN | 9.0.1（A2 / A3 / Ascend 950） |
| PyTorch | 2.10.0 |
| torch_npu | 2.10.0.post2 |
| Triton Ascend | 3.2.1 |
| Mooncake | 0.3.11.post1（release image） |

项目来源：
https://github.com/vllm-project/vllm-ascend/releases

### 为什么这个表很重要

它直接说明了一个原则：

> **框架生态的“推荐组合”不一定等于 TorchNPU 主仓库当前的最新推荐组合。**

例如 TorchNPU 主仓库当前已经可以看到 CANN 9.1.0 / PyTorch 2.12.0 的推荐组合，但 vLLM Ascend 0.23.0 明确锁定在 CANN 9.0.1 + PyTorch 2.10.0 + torch_npu 2.10.0.post2。

所以部署 vLLM 时，不要擅自把其中某一个组件单独升级到“最新版”。

## 2. vLLM Ascend 0.23.0 主要看点

该正式版本重点包括：

- Ascend 950 支持增强。
- DeepSeek V4 端到端支持。
- 稀疏注意力。
- Context Parallel。
- KV Cache offload。
- Prefix Cache。
- P/D 解耦相关能力。
- 图模式与推测解码能力增强。
- 更多模型与硬件组合。

同时 release note 也明确记录了 Known Issues，因此生产部署前仍然必须先核对具体模型 + 硬件 + 并行策略。

**状态：第三方开源生态正式版本。**

## 3. Triton-Ascend

Triton-Ascend 让开发者可以用 Triton 风格开发昇腾算子。

当前项目公开的正式版本动态中：

- 3.2.0：2026-01-20
- 3.2.1：2026-04-30
- 3.2.2：2026-07-31

项目：
https://github.com/triton-lang/triton-ascend

注意：即使 Triton-Ascend 已有 3.2.2，vLLM Ascend 0.23.0 的依赖仍明确写 **3.2.1**。这再次说明“所有组件都装最新”并不是正确的部署方法。

## 4. DeepGEMM-Ascend：非常新的生态进展

DeepSeek 在 **2026-09-30** 发布了 **DeepGEMM-Ascend** 初始版本。

项目定位：

- 把 DeepGEMM 的 API 和开发流程带到 Huawei Ascend 平台。
- 初始版本面向 **Ascend 950**。
- 支持 BF16、FP8、FP4 GEMM。
- 支持 MQA logits。
- 支持 MegaMoE 相关算子。
- 使用 Ascend MAD、稀疏加载、协程流水等昇腾特定优化方法。

项目：
https://github.com/deepseek-ai/DeepGEMM-Ascend

**状态：第三方（DeepSeek）开源项目，2026-09-30 初始发布。不能把它写成华为官方 CANN 功能。**

## 5. 小白如何理解这几个项目的层次

可以先按下面理解：

```text
大模型
  ↓
vLLM Ascend          ← 推理框架 / 推理服务
  ↓
PyTorch + torch_npu
  ↓
CANN
  ↓
Ascend NPU

Triton-Ascend         ← 写高性能算子的工具链之一
DeepGEMM-Ascend       ← 专注高性能矩阵乘等 kernel 的生态项目
```

它们不是互相替代的关系，而是处于不同层次。

## 6. 实验时必须记录的版本

以后做 vLLM Ascend 实验时至少记录：

```text
硬件：
OS：
Driver：
Firmware：
CANN：
Python：
PyTorch：
torch_npu：
vLLM：
vLLM Ascend：
Triton-Ascend：
模型：
量化方式：
并行方式：TP / PP / DP / EP / PCP ...
```

如果少记其中几项，后面很容易出现“同一条命令别人能跑、自己跑不了”的情况。
