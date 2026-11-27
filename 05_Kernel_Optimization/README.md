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

## 统一专题写作与可视化规范

所有二级专题固定采用三个一级章节：

1. **一句话定义与核心目的**：说明技术是什么、为什么需要、解决什么问题。
2. **数学原理与核心技术**：给出必要的数学思想、公式和符号解释；偏工程专题则解释关键机制。
3. **代表案例**：列出直接案例与关联案例，说明项目和专题的具体关系，并提供可靠链接。

配图必须服务于理解，优先采用与知识点匹配的真实可视化（如矩阵热图、频谱、奇异值谱、硬件结构或执行时间线）。Mermaid 仅用于流程/依赖关系，不替代概念图和数据图。GitHub 公式统一使用 `$...$` 与 `$$...$$`。

工程实现和性能比较暂时保留为后续扩展，不提前填充。完整规则见 [专题写作与可视化规范](../DOCUMENTATION_STANDARD.md)。

## 分类边界

本类聚焦单个 Kernel 内部的线程组织、资源分配、指令与流水执行。更高层的任务切分属于并行优化；编译器自动完成的 IR 变换和代码生成属于编译器优化。

## 相关入口
- [八大优化方法总览](../01_Optimization_Methods.md)
- [我们的研究记录](../02_Our_Research_Notes.md)
