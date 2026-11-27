# HiLS-Attention：算法、模型与 GPU 算子研究

> 研究对象：Tencent-Hunyuan/HiLS-Attention  
> 官方仓库：https://github.com/Tencent-Hunyuan/HiLS-Attention  
> 论文入口：https://arxiv.org/abs/2607.02980  
> 当前状态：论文学习、模型源码阅读；尚未完成独立复现和性能实验。

## 1. 研究定位

HiLS-Attention 面向长上下文建模，以 Chunk 为单位组织稀疏检索，通过分层注意力组合局部窗口与检索到的历史 Chunk。

本知识库关注四层：
- 算法：Chunk 表示、检索分数、Top-K、分层归一化。
- 模型：Q/K/V、Landmark Query、GQA、RoPE、KV Cache。
- 算子：Chunk Pooling、Top-K、局部 Attention、稀疏 Attention。
- 硬件：TileLang/Triton Kernel、数据搬运、并行度和跨硬件适配。

## 2. 阅读顺序

| 顺序 | 文档 | 目标 |
|---|---|---|
| 1 | [算法与数学原理](01_Algorithm_and_Math.md) | 理解问题定义与核心公式 |
| 2 | [模型源码导读](02_Model_Code_Walkthrough.md) | 理解 HiLSAttention 的函数和张量流 |
| 3 | [算子与 Kernel 研究](03_Operators_and_Kernels.md) | 从 PyTorch 参考实现走到 GPU Kernel |
| 4 | [实验与跨硬件适配](04_Benchmark_and_Portability.md) | 建立正确性、性能和适配流程 |

## 3. 核心源码入口

- models/FlashHiLS/hils_attention.py
- ops/chunk_attn_pool_tilelang.py
- ops/topk_head_softmax.py
- ops/hils_fwd_bwd_head.py
- ops/flex_attn_tilelang.py
- ops/rope_tilelang_fp32.py

核心模型类：HiLSAttention(nn.Module)。

概念前向链路：

hidden_states → Q/K/V 与 Landmark Query → RoPE/Cache → Sliding-Window Attention → Chunk 表示 → Top-K 检索 → 稀疏 Attention → 分支融合 → output projection

具体路径受配置、训练/推理模式、序列长度及 Cache 状态影响。

## 4. 关键张量约定

令 Batch 为 B，序列长度为 L，Query Head 数为 Hq，KV Head 数为 Hkv，Head Dimension 为 d，Chunk Size 为 S，Chunk 数为 Nc。

- 输入 X: [B,L,D]
- Query Q: [B,L,Hq,d]
- Key/Value K,V: [B,L,Hkv,d]
- 分块 Key Kc: [B,Nc,S,Hkv,d]
- 压缩表示 K': [B,Nc,Hq,d]（典型 GQA 展开后）
- Chunk bias b': [B,Nc,Hq]

GQA group size 为 Hq/Hkv，要求整除。

## 5. 研究问题

1. Chunk Pooling 的计算和内存访问瓶颈是什么？
2. Top-K 检索是否会在长序列下成为新的瓶颈？
3. Chunk Size / Top-K / Head 数量如何影响延迟和检索质量？
4. 稀疏索引的不规则访存如何影响 GPU 利用率？
5. 如何保持数学接口不变，为不同硬件实现独立 Backend？
6. 算子级加速能否传递到 Prefill、Decode 和端到端服务？

## 6. 记录边界

- 本目录是个人学习与研究笔记，不代表腾讯混元团队的官方解释。
- 论文公式、源码实现和个人推导分开标注。
- 数学近似成立不等于检索始终准确；需通过检索指标和任务评估。
- Kernel 更快不等于端到端模型更快。
- 未运行的代码、未测试的硬件和未验证的假设不得写成实验结果。

## 7. 参考资料

- [官方代码仓库](https://github.com/Tencent-Hunyuan/HiLS-Attention)
- [论文](https://arxiv.org/abs/2607.02980)
- [TileLang](https://github.com/tile-ai/tilelang)
- [Triton](https://github.com/triton-lang/triton)
- [PyTorch](https://pytorch.org/)
