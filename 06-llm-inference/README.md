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
