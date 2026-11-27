# 05 Kernel 优化

**Kernel Optimization**

本目录研究单个 GPU Kernel 如何映射到硬件并高效执行。专题内容将在后续逐项讨论后完善。

## 专题索引

| 编号 | 专题 | 状态 |
|---:|---|---|
| 1 | [Kernel 执行模型](01_Kernel_Execution_Model.md) | 待共同设计 |
| 2 | [Thread / Warp / Block / SM](02_Thread_Warp_Block_SM.md) | 待共同设计 |
| 3 | [Warp 调度](03_Warp_Scheduling.md) | 待共同设计 |
| 4 | [Occupancy（占用率）](04_Occupancy.md) | 待共同设计 |
| 5 | [寄存器](05_Registers.md) | 待共同设计 |
| 6 | [Shared Memory（共享内存）](06_Shared_Memory.md) | 待共同设计 |
| 7 | [Tensor Core](07_Tensor_Core.md) | 待共同设计 |
| 8 | [流水线](08_Pipelining.md) | 待共同设计 |
| 9 | [Profiler 与性能分析](09_Profiling.md) | 待共同设计 |

## 统一写作结构

1. 快速理解：定义、问题、核心结论和示意图。
2. 背景与问题：技术出现的原因和性能瓶颈。
3. 核心原理与理论基础：执行模型、公式、资源约束和适用条件。
4. 工程实现：CUDA/Triton 代码、硬件映射、关键参数。
5. 经典项目与案例：关联项目档案、论文和源码。
6. 实验与性能验证：Profiler 指标、基线和性能分析。
7. 我们的理解与研究问题：瓶颈、假设和可验证方向。

## 分类边界

本类聚焦单个 Kernel 内部的线程组织、资源分配、指令与流水执行。更高层的任务切分属于并行优化；编译器自动完成的 IR 变换和代码生成属于编译器优化。

## 相关入口
- [八大优化方法总览](../01_Optimization_Methods.md)
- [我们的研究记录](../02_Our_Research_Notes.md)
