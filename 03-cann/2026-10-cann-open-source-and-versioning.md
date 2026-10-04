# CANN：开源进展与版本配套（2026-10）

> 状态快照：2026-10-04。昇腾软件栈最容易踩坑的地方不是“不会安装”，而是**版本混装**。

## 1. CANN 在昇腾软件栈中的位置

先记住：

```text
PyTorch / MindSpore / 推理框架
        ↓
torch_npu / 框架适配层
        ↓
CANN
        ↓
Driver + Firmware
        ↓
Ascend NPU
```

CANN 不是一个单独的 Python 包，而是一套连接上层框架与昇腾硬件的 AI 软件栈。

## 2. 2026 年的新变化：持续社区化开源开发

华为在 HUAWEI CONNECT 2026 表示，CANN 已进入持续、社区驱动的开源开发阶段。

同时，面向新一代 Ascend 的开发能力也在继续扩展，包括：

- Ascend C 增加 SIMD + SIMT 编程模式。
- 新增 Regbase 编程能力。
- PTO ISA 已公开超过 120 条虚拟指令，覆盖基础计算、数据搬运、通信等类别。
- 华为开源了面向算子生成、模型适配、精度调优和行业部署的 Agent 工具，包括 CANNBot、Model Agent、MindStudio Agent、Solution Agent。

**状态：官方已宣布并持续推进。**

官方来源：
https://www.huawei.com/en/news/2026/9/hc-agentic-thinkpro-pto-cann

## 3. 不要只说“装最新版”

截至 2026-10-03，Ascend 官方 PyTorch 适配项目给出的推荐组合中可看到：

| 组件 | 推荐组合示例 |
|---|---|
| PyTorch | **2.12.0** |
| TorchNPU 安装包 | **2.12.0** |
| CANN | **9.1.0** |
| Python | **3.10 / 3.11 / 3.12 / 3.13 / 3.14** |
| Driver | **按具体 Ascend 硬件 + CANN 版本匹配** |
| Firmware | **按具体 Ascend 硬件 + CANN 版本匹配** |
| OS | **必须再查对应硬件/CANN 安装指南，不应凭这个表推断** |

来源：
https://github.com/Ascend/pytorch/blob/master/COMPATIBILITY.md

### 为什么 Driver / Firmware 没写一个统一版本

因为不同硬件（例如 A2、A3、310P、950 系列）对应的驱动、固件和 CANN 组合不完全相同。

因此正确做法不是：

```text
先装一个“最新 Driver”
再装一个“最新 CANN”
再装一个“最新 torch_npu”
```

而应该是：

```text
先确定硬件型号
    ↓
查官方兼容矩阵
    ↓
确定 Driver / Firmware
    ↓
确定 CANN
    ↓
确定 Python
    ↓
确定 PyTorch
    ↓
确定 torch_npu
```

## 4. TorchNPU 还有两套版本号，不要混淆

Ascend/PyTorch 官方文档里同时存在：

- **TorchNPU 产品版本**，例如 26.1.0
- **torch_npu Python 安装包版本**，例如 2.10.0.post4、2.12.0

例如 2026 年 7 月的 TorchNPU 26.1.0，可分别配套不同 PyTorch 分支：

| PyTorch | torch_npu 安装包 | CANN |
|---|---|---|
| 2.7.1 | 2.7.1.post8 | 9.1.0 |
| 2.9.0 | 2.9.0.post6 | 9.1.0 |
| 2.10.0 | 2.10.0.post4 | 9.1.0 |
| 2.11.0 | 2.11.0 | 9.1.0 |
| 2.12.0 | 2.12.0 | 9.1.0 |

### TorchNPU 26.1.1：当前活跃维护线

截至 2026-10-04，Ascend/PyTorch 官方兼容矩阵已经把 **TorchNPU 26.1.1** 列入当前活跃版本：

| TorchNPU 产品版本 | PyTorch | torch_npu 安装包 | CANN |
|---|---:|---:|---:|
| 26.1.1 | 2.12.0 | 2.12.0.post2 | 9.1.X |
| 26.1.1 | 2.11.0 | 2.11.0.post2 | 9.1.X |
| 26.1.1 | 2.10.0 | 2.10.0.post6 | 9.1.X |
| 26.1.1 | 2.9.0 | 2.9.0.post8 | 9.1.X |
| 26.1.1 | 2.7.1 | 2.7.1.post10 | 9.1.X |

这里要区分两个概念：

- 页面顶部的“推荐组合”仍是 PyTorch 2.12.0 + torch_npu 2.12.0 + CANN 9.1.0。
- 26.1.1 则属于当前活跃维护版本，并把 CANN 范围写成 **9.1.X**。

因此做生产部署时，应按目标框架和硬件选择完整版本组合，而不是看到某个 post 版本更高就单独升级。

兼容矩阵：
https://github.com/Ascend/pytorch/blob/master/COMPATIBILITY.md

版本说明：
https://github.com/Ascend/pytorch/blob/master/docs/zh/release_notes.md

## 5. 一个很容易被忽略的环境变化

当前 TorchNPU README 的示例环境初始化为：

```bash
source /usr/local/Ascend/cann/set_env.sh
```

老教程经常能看到：

```bash
source /usr/local/Ascend/ascend-toolkit/set_env.sh
```

所以复制旧教程命令之前，要先确认自己安装的 CANN 版本和实际目录结构。

## 6. 初学者的版本记录模板

以后每次做实验，都建议在实验文档最上方固定写：

```text
硬件：
OS：
架构：x86_64 / aarch64
Driver：
Firmware：
CANN：
Python：
PyTorch：
torch_npu：
推理框架（如 vLLM Ascend）：
模型：
```

只有这样，遇到算子不支持、编译失败、HCCL 错误或模型跑不起来时，才有条件排查。
