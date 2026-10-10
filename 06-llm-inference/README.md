# 06 · 大模型推理

## Prefill、Decode 与 KV Cache

> 资料核查：2026-10-09（UTC）。本节只学习普通自回归文本生成的计算顺序；示例是纸面推演，未进行 NPU 实测，不涉及安装或部署。

### 1. 一次回答为什么分成两个阶段

Token 是模型处理文本的单位；本节用抽象编号表示，不假设一个 token 等于一个汉字。自回归生成可以先理解为：根据已有上下文预测下一个 token，再把它用于后续预测。

**官方事实：**[CANN 商用版 8.2.RC1“大模型推理流程简介”](https://www.hiascend.com/document/detail/zh/canncommercial/82RC1/acce/llmdatadistdev/llmpd_42_19.html)将使用 KV Cache 的普通生成过程分成以下两阶段（核查：2026-10-09；这里只引用原理，不推荐安装该版本）：

- **Prefill（预填充）**：处理输入提示词，保存相应的 KV Cache，并得到第一个输出 token。
- **Decode（解码）**：把上一步输出的 token 作为新输入，结合已有缓存继续计算，逐步产生后续 token。

因此，Prefill 已经参与首个 token 的生成；Decode 仍需执行模型计算，不能理解为从缓存直接读取一段现成答案。

### 2. KV Cache 保存的是什么

**官方事实：**[Hugging Face Transformers 4.57.1 的 Caching 文档](https://huggingface.co/docs/transformers/v4.57.1/cache_explanation)说明，KV Cache 保存注意力层为已处理 token 计算出的 Key（键）和 Value（值），后续步骤复用它们，并加入新输入 token 的 K、V（核查：2026-10-09）。它们是计算产生的张量，与模型权重、回答文本不同；复用历史 K、V 后，新 token 仍需与历史信息进行注意力计算。

**教学假设：**输入恰好是 `p1、p2、p3` 三个 token，接下来输出 `y1、y2、y3`；只考虑一个请求、普通逐 token 生成和保留完整历史的缓存，不使用前缀复用、滑动窗口或推测解码。这些符号不是实际分词或模型预测结果。

| 步骤 | 本轮新送入模型的 token | 本轮计算后缓存覆盖的 token | 本轮预测的输出 |
|---|---|---|---|
| Prefill | p1、p2、p3 | p1、p2、p3（3 个） | y1 |
| 第一次 Decode | y1 | p1、p2、p3、y1（4 个） | y2 |
| 第二次 Decode | y2 | p1、p2、p3、y1、y2（5 个） | y3 |

按上述假设，得到 `y3` 时，`y3` 还未作为新输入参加下一轮计算。表中数的是已经处理的 token 位置，不是输出数量、缓存块数或 GB/GiB 容量。

### 3. 用这条流程理解容量与复用

缓存用额外存储换取历史 K、V 的计算复用。在本例保留完整历史的假设下，输入越长、继续处理的 token 越多，逻辑缓存就越大。但不能把表中的增长直接当作 NPU 显存每步实测增量：上述 Transformers 文档也区分了随序列增长的动态缓存与预先分配的固定缓存。

所以，[纯权重容量估算](../01-ai-basics/README.md#从参数量估算权重存储空间)之外仍要给 KV Cache 留空间。此处没有模型结构、缓存精度与分配策略等输入，不能据此计算具体容量或卡数。

跨请求复用还涉及额外机制。[Ascend/MindIE-LLM 的 Prefix Cache 文档](https://github.com/Ascend/MindIE-LLM/blob/master/docs/zh/user_guide/feature/prefix_cache.md)说明，可在满足条件时复用共同前缀的 KV Cache，减少重复的 Prefill 计算（核查：2026-10-09）。不能仅凭“用了 KV Cache”就推定新请求会自动命中缓存；这里不据 `master` 文档推定某个发布版或 Atlas 型号支持该功能。

**学习自查：**得到第一个输出主要对应哪个阶段？Decode 为什么还需要历史 K、V？表中预测出 `y2` 时缓存为何只覆盖到 `y1`？能回答这三个问题，再阅读[推理框架版本配套与生态](2026-10-vllm-deepgemm-ecosystem.md)。

### 官方来源与适用范围

以下三页均于 **2026-10-09** 读取正文；页面正文均未提供发布日期或更新日期，核查日期不等于产品发布日期。

| 官方页面 | 本节引用范围 |
|---|---|
| [CANN 大模型推理流程简介](https://www.hiascend.com/document/detail/zh/canncommercial/82RC1/acce/llmdatadistdev/llmpd_42_19.html) | CANN 商用版 8.2.RC1 / LLM DataDist 开发指南的两阶段原理；不引用其软件配套或性能结论 |
| [Transformers Caching](https://huggingface.co/docs/transformers/v4.57.1/cache_explanation) | 4.57.1 文档中的缓存语义；不证明该版本与本仓库的 PyTorch、torch_npu、CANN 组合兼容 |
| [MindIE-LLM Prefix Cache](https://github.com/Ascend/MindIE-LLM/blob/master/docs/zh/user_guide/feature/prefix_cache.md) | `master` 文档中的跨请求前缀复用概念；实际功能限制须按选定 release 重新核对 |

本节未提供可执行命令，未运行分词、模型生成或缓存测量，也未验证任何昇腾硬件环境。

## 读懂推理延迟：TTFT、TPOT 与 E2EL

> 资料核查：2026-10-10（UTC）。本节学习如何阅读流式文本生成的延迟指标；流式是指响应分批返回。以下数字均为教学假设，未进行服务压测或昇腾 NPU 实测。

### 1. 先确定从哪里开始计时

**官方事实：**[vLLM 的 Benchmark CLI 文档](https://github.com/vllm-project/vllm/blob/main/docs/benchmarking/cli.md)明确在压测客户端测量延迟，TTFT 从发送请求计到收到第一份流式输出；不同工具的指标名称并未统一，比较时应核对计时起止点和公式（`main`，核查：2026-10-10）。

[MindIE 2.3.0 的性能测试指标表](https://www.hiascend.com/doc_center/source/en/mindie/230/servicedeploy/servicedev/mindie_service0111.html)给出 TTFT、TPOT、E2EL 的定义与 TPOT 公式；[AISBench 性能测评结果说明](https://gitee.com/aisbench/benchmark/blob/master/doc/users_guide/performance_metric.md)进一步说明 E2EL 从请求发送计到接收全部响应（均核查：2026-10-10）。按这些口径，先区分三个问题：

| 指标 | 全称 | 回答的问题与单位 |
|---|---|---|
| TTFT | Time To First Token | 等多久才开始收到输出？用 ms 或 s 表示 |
| TPOT | Time Per Output Token | 首个 token 之后，平均每个输出 token 花多久？用 ms/token 或 s/token 表示 |
| E2EL | End-to-End Latency | 一条请求从发出到接收完响应共多久？用 ms 或 s 表示 |

**由计时边界可知：**客户端的 TTFT 还可能包含排队、传输等等待，不能直接当作 NPU 内部的 Prefill 算子耗时。同样，TTFT 小只说明开始返回得快，不代表整条回答很快完成。

### 2. 用一条请求手算，理解为什么减一

**教学假设：**只考察一个成功请求，恰好输出 `y1、y2、y3、y4` 四个 token，每份流式输出恰好含一个 token；收到 `y4` 时响应即完成，没有额外结束等待。不采用推测解码或多 token 合并返回，也不对应任何 Atlas 型号的性能。

| 客户端观察到的事件 | 相对发送时刻的时间 |
|---|---|
| 发送请求 | 0 ms |
| 收到 y1 | 120 ms |
| 收到 y2 | 150 ms |
| 收到 y3 | 200 ms |
| 收到 y4，响应完成 | 240 ms |

按顺序计算：

1. `TTFT = 120 − 0 = 120 ms`。
2. `E2EL = 240 − 0 = 240 ms`。
3. 官方公式为 `TPOT = (E2EL − TTFT) ÷ (输出 token 数 − 1)`，所以本例是 `(240 − 120) ÷ (4 − 1) = 40 ms/token`。

减去首个 token 的等待后，只剩三个输出间隔：`30、50、40 ms`，平均为 `40 ms`。这并不表示每个间隔都等于平均值。若只输出一个 token，公式分母为零，不能照算；阅读报告时需查看工具对此类请求的处理规则，不能自行把它记作“零延迟”。

### 3. 读报告时再核对两个边界

- **Token 与返回块是否一一对应。**上述 vLLM 文档说明，ITL（Inter-token Latency）记录相邻流式输出之间的间隔；例如推测解码可让一次输出包含多个 token。因此不要把返回块数量当作输出 token 数，也不要默认报告中的 ITL 与 TPOT 必然相等（`main`，核查：2026-10-10）。
- **统计的是单条请求还是一批请求。**上述 AISBench 文档将 P99 TPOT 解释为请求 TPOT 值的第 99 百分位。它与所有请求的平均 TPOT 不是同一个统计量，也不是本例中最慢的单次输出间隔（`master`，核查：2026-10-10）。

**阅读建议：**先记工具版本与测量位置，再核对单位、输入/输出长度、并发设置及统计口径。秒与毫秒换算为 `1 s = 1000 ms`；本例 `40 ms/token = 0.040 s/token`，不是 `40 token/s`。只有条件和口径一致，延迟结果才适合比较。

### 官方来源与验证边界

以下三页均于 **2026-10-10** 读取正文；本节只引用指标定义，不据此拼接软件安装组合或推定硬件支持。

| 官方页面 | 适用范围及资料日期 |
|---|---|
| [MindIE Performance/Accuracy Test Tool](https://www.hiascend.com/doc_center/source/en/mindie/230/servicedeploy/servicedev/mindie_service0111.html) | MindIE 2.3.0 文档中的 AISBench 指标表；正文未提供发布日期或更新日期 |
| [vLLM Benchmark CLI](https://github.com/vllm-project/vllm/blob/main/docs/benchmarking/cli.md) | `main` 文档的客户端延迟口径；本次可读页面未提供发布日期或更新日期，不外推到本仓库介绍的 vLLM Ascend 0.23.0 |
| [AISBench 性能测评结果说明](https://gitee.com/aisbench/benchmark/blob/master/doc/users_guide/performance_metric.md) | MindIE 官方链接所指项目的 Gitee `master` 文档；页面显示文件提交作者时间为 2025-07-02 14:43（UTC+8），正文未单列发布日期或更新日期，该时间不作为工具版本发布日期 |

本节没有可执行命令；仅复核纸面算术，未运行压测工具、模型推理或 NPU 实验，未测量网络、排队或算子耗时。
