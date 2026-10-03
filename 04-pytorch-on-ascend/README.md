# 04 · PyTorch on Ascend

## 为什么优先学习这条路线

大量现有 AI 项目以 PyTorch 为基础，因此“如何把 PyTorch 工作负载运行在 Ascend 上”是非常实用的问题。

## 基础路径

```text
PyTorch Model
     ↓
torch_npu
     ↓
CANN
     ↓
Ascend NPU
```

## 后续实验

1. 检查 NPU
2. 检查 CANN
3. 创建 Python 环境
4. 安装匹配版本的 PyTorch / torch_npu
5. 创建 Tensor
6. 把 Tensor 放到 NPU
7. 运行简单神经网络
8. 下载小型 LLM
9. 运行推理
10. 封装 HTTP / OpenAI-compatible API

所有实验将明确记录版本，避免照抄过时命令。
