# 06 编译器优化

**Compiler Optimization**

本目录研究编译器如何分析、变换和生成适合目标硬件的执行代码。专题内容将在后续逐项讨论后完善。

## 专题索引

| 编号 | 专题 | 状态 |
|---:|---|---|
| 1 | [Compiler 基础](01_Compiler_Fundamentals.md) | 待共同设计 |
| 2 | [中间表示（IR）](02_Intermediate_Representation.md) | 待共同设计 |
| 3 | [Lowering（逐层降低）](03_Lowering.md) | 待共同设计 |
| 4 | [图编译](04_Graph_Compilation.md) | 待共同设计 |
| 5 | [编译器算子融合](05_Compiler_Operator_Fusion.md) | 待共同设计 |
| 6 | [Layout Transformation（布局变换）](06_Layout_Transformation.md) | 待共同设计 |
| 7 | [Auto-Tuning（自动调优）](07_Auto_Tuning.md) | 已建立初稿 |
| 8 | [代码生成](08_Code_Generation.md) | 待共同设计 |
| 9 | [跨硬件编译](09_Cross_Hardware_Compilation.md) | 待共同设计 |

## 统一专题写作与可视化规范

所有二级专题固定采用三个一级章节：

1. **一句话定义与核心目的**：说明技术是什么、为什么需要、解决什么问题。
2. **数学原理与核心技术**：给出必要的数学思想、公式和符号解释；偏工程专题则解释关键机制。
3. **代表案例**：列出直接案例与关联案例，说明项目和专题的具体关系，并提供可靠链接。

配图必须服务于理解，优先采用与知识点匹配的真实可视化（如矩阵热图、频谱、奇异值谱、硬件结构或执行时间线）。Mermaid 仅用于流程/依赖关系，不替代概念图和数据图。GitHub 公式统一使用 `$...$` 与 `$$...$$`。

工程实现和性能比较暂时保留为后续扩展，不提前填充。完整规则见 [专题写作与可视化规范](../DOCUMENTATION_STANDARD.md)。

## 分类边界

本类聚焦编译期分析、IR 变换、Lowering 和代码生成。运行时任务分配属于系统优化；手工设计单个 Kernel 的执行细节属于 Kernel 优化。

## 相关入口
- [八大优化方法总览](../01_Optimization_Methods.md)
- [我们的研究记录](../02_Our_Research_Notes.md)
