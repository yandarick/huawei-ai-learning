# Huawei AI Learning

[在线阅读](https://learn.rickcn.cn/huawei/)

> 面向 AI 初学者的华为 AI / 昇腾学习笔记。目标不是堆术语，而是逐层搞清楚：硬件是什么、软件栈怎么配合、大模型如何跑起来、最后怎样扩展到 AI 集群。

**最后更新：2026-10-08**

## 先看懂全景图

```text
AI 应用 / Agent / RAG
        ↓
大模型（盘古、DeepSeek、Qwen 等）
        ↓
PyTorch / MindSpore
        ↓
CANN（昇腾 AI 软件栈）
        ↓
Ascend NPU / Atlas
        ↓
超节点 SuperPoD / AI 集群
        ↓
RoCE / IB / 存储 / 服务器 / 液冷 / 数据中心
```

一句话理解：

- **Ascend / 昇腾**：华为 AI 计算平台的核心。
- **NPU**：执行 AI 计算的处理器。
- **Atlas**：基于昇腾的 AI 计算产品/服务器/集群体系。
- **CANN**：连接 AI 框架与昇腾硬件的软件栈，可类比理解为昇腾生态中非常关键的“CUDA 层”角色，但二者架构和接口并不等同。
- **MindSpore**：华为主导的 AI 框架。
- **PyTorch on Ascend**：让大量 PyTorch 工作负载迁移到昇腾的重要路线。
- **ModelArts**：华为云上的 AI 开发与训练平台。
- **SuperPoD / 超节点**：把大量 AI 加速卡通过高速互联组织成更大的计算系统。

## 2026 年值得重点关注

### 1. 从单卡转向超节点和大规模集群

华为 AI 基础设施的重点已经不只是“某一张昇腾卡有多快”，而是计算、互联、网络、存储和系统软件共同组成的大规模 AI 系统。

学习时因此不要只研究 NPU 参数，还需要逐步理解：

1. 单 NPU
2. 单服务器多 NPU
3. 多服务器训练
4. SuperPoD 超节点
5. 大规模 AI Cluster

### 2. CANN 是必须掌握的核心

以后看到昇腾相关报错时，经常会碰到：

- Driver
- Firmware
- CANN Toolkit
- AscendCL
- HCCL
- torch_npu
- 算子兼容
- 版本匹配

所以学习昇腾时，**CANN 应当排在很前面，而不是最后再学。**

### 3. PyTorch + Ascend 很重要

对于已经熟悉 CUDA / PyTorch 的开发者，PyTorch 迁移到昇腾是非常现实的路线。

初学者暂时记住：

```text
普通 PyTorch
   ↓
torch_npu
   ↓
CANN
   ↓
Ascend NPU
```

从[PyTorch on Ascend：版本选择、环境检查与第一个 NPU Tensor](04-pytorch-on-ascend/README.md)开始。教程已补充命令及验收说明，尚未进行昇腾硬件实测；本次变化见[2026-10 更新记录](updates/2026-10.md)。

## 学习路线

### Level 0：AI 基础

先理解：

- CPU / GPU / NPU
- Tensor
- FP32 / FP16 / BF16 / INT8 / INT4
- Training 与 Inference
- Transformer
- Token
- 参数量
- 显存/HBM
- FLOPS
- 带宽
- AllReduce

目标：以后看到“70B、BF16、8 卡训练、HCCL”时知道它们分别在说什么。

先阅读 [AI 基础：从参数量估算权重存储空间](01-ai-basics/README.md#从参数量估算权重存储空间)，用 70B 示例区分参数个数、数据类型、GB/GiB 与实际推理容量需求。

### Level 1：认识华为 AI 产品

学习：

- Ascend
- Atlas
- 昇腾服务器
- AI 加速卡
- SuperPoD
- 华为云 ModelArts

### Level 2：CANN

重点：

- CANN 架构
- Driver / Firmware / Toolkit
- AscendCL
- HCCL
- 算子
- profiling
- 环境变量
- 版本兼容矩阵

从 [HCCL 与 AllReduce 入门](03-cann/README.md#hccl-与-allreduce-入门)理解集合通信，用手算理解求和与平均的区别，并分清元素数量和通信成员数；该节未进行多卡通信实测。

### Level 3：AI 框架

两条路线：

**路线 A：PyTorch + torch_npu**

适合已有 PyTorch 生态和模型迁移。

**路线 B：MindSpore**

用于理解华为原生 AI 软件生态。

### Level 4：跑第一个大模型

目标：

```text
Linux
 ↓
Ascend Driver
 ↓
CANN
 ↓
PyTorch + torch_npu
 ↓
模型
 ↓
推理服务
 ↓
OpenAI-compatible API
```

建议从小模型开始，而不是直接挑战 70B。

准备推理环境前，先阅读[vLLM Ascend 版本配套与适用范围](06-llm-inference/2026-10-vllm-deepgemm-ecosystem.md)。该文说明指定 release 的依赖组合和硬件边界，尚未进行 NPU 推理实测。

### Level 5：大模型推理

继续学习：

- vLLM / 昇腾适配生态
- KV Cache
- Continuous Batching
- Prefill / Decode
- TTFT
- TPS
- 并发
- 量化

### Level 6：分布式训练

重点理解：

- Data Parallel
- Tensor Parallel
- Pipeline Parallel
- Expert Parallel
- MoE
- HCCL
- AllReduce
- Scale-up
- Scale-out

### Level 7：AI 集群与数据中心

这部分对网络/系统集成人员尤其重要：

- SuperPoD
- Spine-Leaf
- RoCE
- RDMA
- ECN / PFC
- AI Fabric
- 管理网络
- 存储网络
- 带外管理
- NVMe / 分布式存储
- 液冷
- 供电
- 机柜功率
- 集群监控
- 故障域

## 推荐目录

```text
huawei-ai-learning/
├── README.md
├── 01-ai-basics/
├── 02-ascend-hardware/
├── 03-cann/
├── 04-pytorch-on-ascend/
├── 05-mindspore/
├── 06-llm-inference/
├── 07-distributed-training/
├── 08-superpod-network/
├── 09-modelarts/
├── 10-labs/
└── updates/
```

## 第一个阶段的学习目标

暂时不要追求“会部署千卡集群”。

第一阶段只解决五个问题：

1. 昇腾是什么？
2. Atlas、Ascend、NPU 三者是什么关系？
3. CANN 为什么必须安装？
4. PyTorch 模型怎样跑到 Ascend 上？
5. 一台昇腾服务器怎样运行一个大模型？

搞懂这五个问题，再进入 SuperPoD 和大规模网络，会容易很多。

## 后续更新

本仓库将持续补充：

- 华为 AI 新产品与新版本
- Ascend / Atlas 硬件
- CANN 安装与排障
- PyTorch 模型迁移
- MindSpore
- 大模型推理
- DeepSeek / Qwen 等模型在昇腾上的实践
- SuperPoD 架构
- AI 集群网络
- 存储与液冷
- ModelArts
- 实验步骤与故障案例

## 官方资料入口

学习时优先使用华为官方资料：

- Huawei Ascend / 昇腾社区
- CANN Documentation
- MindSpore Documentation
- Huawei Cloud ModelArts Documentation
- Huawei 官方技术与产品发布资料

后续章节会为具体知识点附上对应官方来源和版本日期，避免把旧版本教程与当前版本混用。

## 核对网站阅读版本

网站从本仓库的 `main` 构建。打开[网站来源清单](https://learn.rickcn.cn/build-info.json)，找到 `repo` 为 `yandarick/huawei-ai-learning` 的记录，将其 `sha` 与 [main 的最近提交](https://github.com/yandarick/huawei-ai-learning/commits/main/)对照；文章顶部“查看原始笔记”链接也应指向该来源提交。

PR 已推送、合并到主线和网站成功发布是不同状态。若来源 SHA 尚未更新，网站可能仍在展示上一次成功构建；构建时间也不等于文章技术核验日期。这一核对方法依据[网站同步实现](https://github.com/yandarick/rick-ai-learning-site/blob/f52bcdf8462edddac93ac4f9fbd06505bcf501f9/scripts/sync-content.mjs)及线上来源清单，核查日期：2026-10-04。
