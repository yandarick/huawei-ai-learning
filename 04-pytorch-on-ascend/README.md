# 04 · PyTorch on Ascend

> 资料核查：2026-10-04（UTC）。本章只学习“选配套版本、检查已有环境、创建第一个 NPU Tensor”。全部示例命令均未在本次维护中执行，尚无昇腾硬件实测结果。

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
Driver（驱动）+ Firmware（固件）
     ↓
Ascend NPU
```

PyTorch 提供张量和模型接口，`torch_npu` 是把 PyTorch 接到昇腾上的适配插件。CANN 提供计算所需的软件能力；驱动负责操作系统与设备交互，固件运行在设备侧。它们属于不同层，不能用某一层的版本号代替整套环境记录。

## 1. 先选一整套版本

建议按这个顺序准备，已有服务器时先记录现状，不急着升级：

1. 确认具体 Atlas/Ascend 型号、操作系统、CPU 架构，以及使用物理机还是容器。
2. 如果目标项目指定了 PyTorch/CANN 版本，先看该项目要求，再到官方矩阵找能同时满足的组合。
3. 在同一行读取 PyTorch、`torch_npu` 安装包和 CANN 配套；再查 Python 支持范围。
4. 按目标硬件和选定的 CANN 核对驱动、固件、操作系统要求。矩阵中的 Python 包组合不能代替这一步。

**官方说明，核查于 2026-10-04：**[Ascend/PyTorch 兼容矩阵](https://github.com/Ascend/pytorch/blob/master/COMPATIBILITY.md)的推荐组合为 PyTorch `2.12.0`、`torch_npu` 安装包 `2.12.0`、CANN `9.1.0`；下表据此选一个教学基线。

| 层次 | 本章示例选择 | 如何理解 |
|---|---|---|
| TorchNPU 产品版本 | `26.1.0` | 产品发布编号，不能直接当作 Python 包版本 |
| PyTorch | `2.12.0` | Python 包名为 `torch` |
| torch_npu 安装包 | `2.12.0` | Python 中使用 `import torch_npu` |
| CANN | `9.1.0` | 单独记录软件栈版本及实际安装目录 |
| Python | `3.11.x` | 本章的教学选择，实验记录还应写下具体补丁号 |
| Driver / Firmware / OS | 待按具体硬件核对 | 本章不提供跨型号通用版本号 |

产品编号与上述安装包的对应关系见[官方版本说明](https://github.com/Ascend/pytorch/blob/master/docs/zh/release_notes.md)（核查：2026-10-04）；该页将 `26.1.0` 标为正式版本，发布时间只写到 **2026 年 7 月**，未提供具体日。

同日核查发现：[兼容矩阵](https://github.com/Ascend/pytorch/blob/master/COMPATIBILITY.md)为 PyTorch `2.12.0` 列出 Python `3.10–3.14`，而[版本说明](https://github.com/Ascend/pytorch/blob/master/docs/zh/release_notes.md)的对应行列出 `3.10.x–3.13.x`。两页范围不完全一致，因此本章选择两者均列出的 `3.11.x`；这不是对 Python `3.14` 可用或不可用的实测结论。

矩阵还列有其他产品版本及 `.postN` 包，不能只比较数字大小就替换本章某一个组件。更多背景见[两套版本号与配套关系](../03-cann/2026-10-cann-open-source-and-versioning.md)。上述在线 `master` 文档会变，部署时需要重新核对。

## 2. 检查已有环境

### 先把硬件与软件记录分开

以下是实验记录模板，未知项写“未核实”，不要填上预期值冒充实测值：

```text
实验日期与结果：未执行 / 失败（首个错误）/ 通过
硬件型号、OS、CPU 架构：
物理机或容器（容器请记录镜像标签/摘要）：
Driver 版本：
Firmware 版本：
CANN 版本、组件及实际安装目录：
Python 完整版本、解释器路径：
PyTorch 完整版本：
torch_npu 完整安装包版本：
所用官方配套页面与核查日期：
```

驱动、固件和 CANN 的实际版本应从设备提供方的安装记录及相应产品指南的查询结果取得，分别与配套要求比较。这里尚未核实某台服务器的这些信息，也未从昇腾下载入口取得可读的硬件配套表；拿到这些信息前，只能准备方案，不能认定环境兼容。安装目录存在，也不等于设备能够运行计算。

### 再核对当前 Python 环境

适用范围：Linux Bash、已配置的上述教学基线。先进入你实际用于实验的 Python 环境，再执行以下**未实测示例**；`python` 应指向该环境的解释器。

```bash
python --version
python -c "import sys; print(sys.executable)"
python -m pip list
```

检查列表中的 `torch` 与 `torch_npu`，记录完整版本（包括可能出现的 `+cpu`、`.postN` 后缀），并与选定行核对。使用 `python -m pip` 是为了让包查询跟随同一个解释器。查询思路来自[官方“查询版本”](https://github.com/Ascend/pytorch/blob/master/docs/zh/installation_guide/references/check_installed_versions.md)（核查：2026-10-04）；本章调整了命令写法，没有采用该页开发版本的示例输出。

包不存在或版本不符时先停在这里，按选定版本的官方安装指南准备环境。[官方 README 的安装示例](https://github.com/Ascend/pytorch/blob/master/README.zh.md)使用 CPU 版 PyTorch 加 TorchNPU 插件（核查：2026-10-04）；`+cpu` 本身不能用来判断 NPU 是否可用，后面还要检查插件和设备。

### 加载对应 CANN 环境

[官方 README 快速开始](https://github.com/Ascend/pytorch/blob/master/README.zh.md)在上述 `2.12.0 / 9.1.0` 示例下给出的默认路径如下（核查：2026-10-04；**本次未执行**）：

```bash
# 先确认实际安装位置；路径不同应使用该安装目录提供的脚本
source /usr/local/Ascend/cann/set_env.sh
```

这一步为当前 shell 加载 CANN 所需环境变量，并不安装 CANN。文件不存在时应核对版本、目录和安装记录，不能仅凭旧教程猜路径。后续 Python 命令在同一个 shell 中执行。

## 3. 第一个 NPU Tensor：搬过去、计算、取回来

Tensor（张量）可以先理解为多维数字数组。本例使用二维矩阵，关注三个属性：`shape` 是行列大小，`dtype` 是数字类型，`device` 是数据所在设备。

适用范围：第 1 节配套已核对、环境已配置且当前进程能够访问 NPU 的 Linux 环境。本例按[官方 README 的 NPU 矩阵乘法示例](https://github.com/Ascend/pytorch/blob/master/README.zh.md)改写为固定输入；可用性检查依据[官方 NPU 模块中的 `is_available` 与 `device_count`](https://github.com/Ascend/pytorch/blob/master/torch_npu/npu/__init__.py)（均核查于 2026-10-04，`master` 源码仅作为接口参考）。**以下命令未实测，不是运行日志。**

```bash
python - <<'PY'
import torch
import torch_npu

available = torch_npu.npu.is_available()
print("NPU available:", available)
if not available:
    raise SystemExit("当前进程未发现可用 NPU；停止实验，先检查环境。")
print("NPU count:", torch_npu.npu.device_count())

x_cpu = torch.tensor([[1, 2], [3, 4]], dtype=torch.float32, device="cpu")
y_cpu = torch.tensor([[5, 6], [7, 8]], dtype=torch.float32, device="cpu")
x_npu = x_cpu.to("npu:0")
y_npu = y_cpu.to("npu:0")
z_npu = x_npu.mm(y_npu)

print("CPU input:", x_cpu.device, x_cpu.shape, x_cpu.dtype)
print("NPU result:", z_npu.device, z_npu.shape, z_npu.dtype)
result_cpu = z_npu.to("cpu")
print(result_cpu)
assert z_npu.device.type == "npu"
assert result_cpu.tolist() == [[19.0, 22.0], [43.0, 50.0]]
print("本次小矩阵实验通过")
PY
```

逐步理解：

1. 显式导入插件后检查设备是否可见。`True` 和设备数量只说明这一层检查通过，后面的计算仍可能失败。
2. 两个 CPU 张量均为 `2 × 2`、`float32`（FP32）。`.to("npu:0")` 返回位于该 NPU 设备的张量，原来的 `x_cpu` 仍在 CPU 上；必须接住返回值。本例选择当前进程中的逻辑设备 `npu:0`，不能据此推定机箱上的物理卡编号。
3. `mm` 做矩阵乘法，不是逐元素相乘。结果左上角应为 `1 × 5 + 2 × 7 = 19`。
4. 先检查结果的 NPU 设备属性，再把结果复制到 CPU 核对数值。最后打印出的 CPU 张量没有 `npu:0` 标记，不能因此误判此前计算发生在 CPU。

设备转换和矩阵乘法语义分别见 [PyTorch 2.12 `Tensor.to`](https://docs.pytorch.org/docs/2.12/generated/torch.Tensor.to.html) 与 [PyTorch 2.12 `torch.mm`](https://docs.pytorch.org/docs/2.12/generated/torch.mm.html)（核查：2026-10-04）；上游 API 文档本身不代表昇腾支持全部参数和硬件组合。

**预期验收标准（不是本机实测结果）：**程序正常结束，结果设备为 `npu:0`，形状为 `2 × 2`，类型为 `torch.float32`，取回的矩阵为 `[[19, 22], [43, 50]]`，两项断言通过。本例的小整数结果可以精确比较；不要将这种比较方法直接用于一般浮点模型的精度验收。

即使通过，也只验证了该环境下的数据迁移和一次小矩阵运算，不能据此声称大模型推理、训练、多卡通信或性能已经验证。没有可访问的昇腾 NPU 时，可以手算理解本例，实验记录仍应写“未进行 NPU 实测”。

## 4. 卡在哪一步，就先检查哪一层

以下为排查建议，不是已确认的错误原因：

| 现象 | 优先检查 |
|---|---|
| 找不到 `torch` / `torch_npu` | 当前解释器路径、同一环境的包列表、完整包版本 |
| 导入插件时报动态库或符号错误 | CANN 环境是否加载、实际加载目录、PyTorch/插件/CANN 是否配套；保留首个错误 |
| NPU 不可见或查询直接报错 | 硬件是否已分配给当前环境、设备权限、驱动与固件状态；容器还需核对设备与驱动映射 |
| 张量迁移或 `mm` 失败 | 输入设备和类型、目标硬件的算子支持、完整软件版本记录 |
| 数值断言失败 | 保存输入、输出及报错；不要删掉断言后把实验记为通过 |

下一步再学习简单神经网络与模型迁移；本次先把单设备小实验的前提和验收边界说清楚。
