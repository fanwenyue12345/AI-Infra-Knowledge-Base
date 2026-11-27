# 08 系统与分布式优化

**System & Distributed Optimization**

本目录研究训练与推理系统中的执行、通信、内存管理和请求调度，关注单个算子之外的端到端性能。

## 专题索引

| 编号 | 专题 | 状态 |
|---:|---|---|
| 1 | [CUDA Graph](01_CUDA_Graph.md) | 待共同设计 |
| 2 | [NCCL](02_NCCL.md) | 待共同设计 |
| 3 | [AllReduce](03_AllReduce.md) | 待共同设计 |
| 4 | [通信与计算重叠](04_Communication_Compute_Overlap.md) | 待共同设计 |
| 5 | [DeepSpeed ZeRO](05_DeepSpeed_ZeRO.md) | 待共同设计 |
| 6 | [Continuous Batching（连续批处理）](06_Continuous_Batching.md) | 待共同设计 |
| 7 | [KV Cache](07_KV_Cache.md) | 待共同设计 |
| 8 | [端到端性能](08_End_to_End_Performance.md) | 待共同设计 |

## 统一写作结构

1. 快速理解：定义、问题、核心结论和示意图。
2. 背景与问题：系统瓶颈和应用场景。
3. 核心原理与理论基础：执行模型、通信/存储机制和约束。
4. 工程实现：框架、API、运行时和关键参数。
5. 经典项目与案例：NCCL、Megatron-LM、DeepSpeed、vLLM 等。
6. 实验与性能验证：端到端吞吐、延迟、显存、通信和资源利用。
7. 我们的理解与研究问题：系统瓶颈、调度策略和可验证方向。

## 分类边界

本类关注完整训练/推理系统和跨设备协作。并行策略的基本模型放在并行优化中；通信与计算重叠在并行优化介绍调度思想，在本类深入 NCCL、运行时和真实系统实现。

## 相关入口
- [八大优化方法总览](../01_Optimization_Methods.md)
- [我们的研究记录](../02_Our_Research_Notes.md)
