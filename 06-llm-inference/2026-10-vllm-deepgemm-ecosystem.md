# 昇腾大模型推理生态：vLLM Ascend、Triton-Ascend、DeepGEMM-Ascend

> 状态快照：2026-10-03；vLLM Ascend 版本配套及相关说明复核于 2026-10-06（UTC），其他内容保留原核查日期。本文中的 vLLM、Triton、DeepSeek 项目属于昇腾生态的重要开源项目，但不是华为产品发布本身；这里作为生态补充记录。

## 1. vLLM Ascend 0.23.0：先看完整配套

本节的学习目标是读懂指定版本的依赖组合，避免把单独可用的组件混装成一个未经验证的推理环境。

**官方说明（核查：2026-10-06）：**[v0.23.0 发布说明](https://github.com/vllm-project/vllm-ascend/releases/tag/v0.23.0)标注发布于 **2026-08-16**，当前正文的 Dependencies 列出以下组合。这是指定版本的资料核对，不表示它是最新版本，也不表示本机已验证。

| 组件 | vLLM Ascend 0.23.0 |
|---|---|
| vLLM | 0.23.0 |
| Python | >= 3.10，< 3.13 |
| CANN | 9.1.0（A2 / A3 / Ascend 950） |
| PyTorch | 2.10.0 |
| torch_npu | 2.10.0.post4 |
| Triton Ascend | 3.2.2（A2 / A3 / Ascend 950） |
| Mooncake | 0.3.11.post1（发布镜像中的版本） |

同一发布说明要求 **Atlas 300I DUO** 使用对应平台的 CANN 安装指南，并明确该产品不支持 Triton Ascend，不能照搬上表的 A2 / A3 / Ascend 950 配套。

[项目官方安装指南](https://docs.vllm.ai/projects/ascend/en/main/getting_started/installation.html)在本次核查时也列出上述 PyTorch、torch_npu、CANN 与 Triton Ascend 组合，并区分 **Python 允许范围 `>=3.10,<3.13`** 和 **发布镜像采用的验证版本 `3.12`**。前者是版本范围，后者是一个具体环境，不能相互替代。该页未标注发布日期或更新日期；它属于会变化的 `main` 文档，使用其他 release 时应重新核对。

**不要把 `.post1` 简化成“只修错误、依赖不变”。**[v0.23.0.post1 发布说明](https://github.com/vllm-project/vllm-ascend/releases/tag/v0.23.0.post1)的正文标题日期为 **2026-09-21**，并说明包含错误修复、依赖及文档更新（核查：2026-10-06）。上表依据 `v0.23.0` 正文整理，不据此宣称 `.post1` 的全部依赖完全相同。

### 为什么这个表很重要

它直接说明了一个原则：

> **框架生态的“推荐组合”不一定等于 TorchNPU 主仓库当前的最新推荐组合。**

例如，本仓库[PyTorch 小实验](../04-pytorch-on-ascend/README.md)选择的是张量入门教学组合；准备 vLLM 时，应先确定目标 release 和硬件，再读取其整套配套，不能直接沿用小实验环境。

所以部署 vLLM 时，不要擅自把其中某一个组件单独升级到“最新版”。

**验证边界：**驱动（Driver）、固件（Firmware）和 OS 仍须按具体产品及 CANN 要求分别核对；Python 包配套表不能替代硬件侧检查。本次只读取官方文档，未安装软件、未执行安装指南中的命令，未进行 NPU 推理实测。

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

配套应跟随目标推理框架：[vLLM Ascend v0.23.0 发布说明](https://github.com/vllm-project/vllm-ascend/releases/tag/v0.23.0)列出的 Triton Ascend 是 **3.2.2**，适用范围见第 1 节（核查：2026-10-06）。不能仅凭 Triton-Ascend 自身版本更新就替换推理框架的配套。

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
