# AI Infra & High-Performance Computing Knowledge Base

个人 AI Infra、高性能计算与 GPU 算子研究知识库。

本仓库记录技术原理、经典工程案例、论文阅读、源码分析、实验复现，以及个人研究问题。内容区分为：

- **Knowledge**：公开技术与基础知识。
- **Understanding**：对数学原理、计算过程和工程实现的个人理解。
- **Research**：尚待验证的假设、实验设计和潜在创新方向。

## 知识地图

| 文档 | 内容 |
|---|---|
| [01 — 八类高性能计算优化方法](01_Optimization_Methods.md) | 从算法、计算图、内存、并行、Kernel、编译器、数值精度到系统优化 |
| [02 — 我们的研究记录](02_Our_Research_Notes.md) | ReWY 案例分析、个人理解、待验证问题与研究假设 |
| [03 — Attention 算子研究知识库](03_Attention_Operators/README.md) | FLA、HiLS、FlashAttention 等算法、模型、算子与跨硬件适配 |

## 八类优化方向

1. Algorithmic & Mathematical Optimization
2. Computation Graph Optimization
3. Memory Optimization
4. Parallelism Optimization
5. Kernel Optimization
6. Compiler Optimization
7. Numerical Precision Optimization
8. System & Distributed Optimization

## 记录原则

1. 尽可能从问题出发，解释技术为什么出现。
2. 数学公式要说明变量、推导和适用条件。
3. 区分算法思想、工程实现和实验结论。
4. 对外部项目和论文给出原始链接及引用。
5. 明确区分已有成果、个人理解与尚未验证的研究假设。
6. 性能比较必须说明硬件、输入规模、精度和计时范围。

## 研究主线

**Mathematics → Computation Graph → Operator / Kernel → Compiler → Hardware → System Benchmark**

本知识库持续更新，不以收集术语为目标，而以建立可复用的技术理解和可验证的研究能力为目标。
