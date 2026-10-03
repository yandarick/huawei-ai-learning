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
