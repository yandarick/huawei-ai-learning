# MindSpore 入门与当前版本状态

> 状态快照：2026-10-03。

## 1. MindSpore 是什么

MindSpore（昇思）是一个开源 AI 框架，可以运行在 Ascend 等硬件上。

对于初学者，可以把它与 PyTorch 放在同一“框架层”理解：

```text
模型代码
  ↓
PyTorch / MindSpore
  ↓
昇腾软件栈
  ↓
Ascend NPU
```

并不是所有昇腾项目都必须使用 MindSpore。当前生态中，PyTorch + TorchNPU 也是非常重要的路线。

## 2. 当前正式发布版本：MindSpore 2.10

MindSpore **2.10** 于 **2026-07-30 正式发布**。

主要方向包括：

- HyperParallel：面向超节点/大规模训练的并行体系。
- DTensor：分布式张量抽象。
- FSDP：参数、梯度和优化器状态分片。
- Context Parallel：面向 128K+ 长序列训练。
- Pipeline Parallel：支持 GPipe、1F1B、VPP 等调度。
- Activation Checkpoint / 重计算。
- Swap：把部分激活卸载到 CPU。
- Distributed Checkpoint。
- Muon / AdamW 等分布式优化。
- AKG / MFusion 图算融合。
- MindSpore Lite 云侧与端侧推理增强。

官方发布说明：
https://www.mindspore.cn/version-updates/zh/2_10

Release Notes：
https://www.mindspore.cn/docs/zh-CN/r2.10.0/RELEASE.html

**状态：正式发布。**

## 3. 哪些功能是稳定，哪些还不是

MindSpore 2.10 Release Notes 会明确标记功能成熟度。

例如：

- DTensor：STABLE
- FSDP：STABLE
- Context Parallel：STABLE
- Pipeline Parallel：STABLE
- Swap：STABLE
- Distributed Checkpoint：STABLE
- Tensor Parallel（TP）：在 2.10 文档中标记为 **DEMO**

因此不能只看到“支持 TP”就默认它和所有 STABLE 功能处于完全相同的成熟度。

## 4. 为什么网上已经能看到 2.11 文档

MindSpore 的 master 文档中已经出现 **2.11.0 Release Notes** 内容，例如 TP(MC2)、优化器状态 Swap、Context Parallel 扩展等。

但截至本次检查：

- 官方“版本动态”页面最新正式发布仍是 **2.10**
- master 页面属于开发分支文档

因此本仓库暂时把：

- **MindSpore 2.10：正式发布**
- **MindSpore 2.11 master 内容：开发中 / 未作为当前正式稳定版引用**

master 文档：
https://www.mindspore.cn/docs/zh-CN/master/RELEASE.html

## 5. 小白该不该先学 MindSpore

建议先知道它是什么，但不用一开始同时深入两套框架。

如果你的目标是理解昇腾软件栈，可以先走：

```text
Python
 ↓
PyTorch
 ↓
torch_npu
 ↓
CANN
 ↓
Ascend
```

等把 NPU、CANN、模型推理跑通后，再学习：

```text
MindSpore
 ↓
HyperParallel
 ↓
大模型训练 / 超节点
```

这样学习曲线更平缓。
