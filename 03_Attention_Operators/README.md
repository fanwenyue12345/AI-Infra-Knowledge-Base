# Attention 算子研究知识库

本目录系统研究 Attention 算法及其高性能实现，建立从数学算法到 GPU Kernel、编译器和硬件执行的映射。

## 项目索引

| 项目 | 研究内容 | 状态 |
|---|---|---|
| [HiLS-Attention](HiLS-Attention/README.md) | 分层稀疏注意力、Chunk 检索、压缩表示、GPU Kernel | 学习与源码分析中 |
| FLA | 线性注意力、状态空间模型与高性能序列算子 | 待整理 |
| FlashAttention | 精确 Attention 的 IO-aware 实现 | 待整理 |
| HiLS × Cross-Hardware | 算子抽象与跨硬件适配 | 研究计划 |

## 统一分析框架

1. Background：研究背景、问题定义、相关工作。
2. Algorithm：数学公式、复杂度、近似与适用边界。
3. Model：张量形状、模块结构、前向与反向流程。
4. Operator：算子拆分、接口、数据布局、调用关系。
5. Kernel：Triton/CUDA/TileLang 实现、分块、访存、并行调度。
6. Hardware：计算单元、显存带宽、片上存储、编译器约束。
7. Benchmark：正确性、数值误差、延迟、吞吐、显存与端到端表现。
8. Research：瓶颈、假设、消融实验和潜在创新。

## 记录约定

- **Fact**：论文、官方文档或源码可以直接确认的事实。
- **Understanding**：个人对原理和实现的解释。
- **Hypothesis**：尚未通过实验验证的研究假设。
- **Result**：记录环境、硬件、输入规模、精度和测量边界的实验结果。

不要将算法复杂度下降、Kernel 加速和端到端模型加速混为一谈。
