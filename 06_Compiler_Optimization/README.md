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
| 7 | [Auto-Tuning（自动调优）](07_Auto_Tuning.md) | 待共同设计 |
| 8 | [代码生成](08_Code_Generation.md) | 待共同设计 |
| 9 | [跨硬件编译](09_Cross_Hardware_Compilation.md) | 待共同设计 |

## 统一写作结构

1. 快速理解：定义、问题、核心结论和示意图。
2. 背景与问题：编译器要解决什么抽象差距和性能问题。
3. 核心原理与理论基础：IR、变换规则、正确性和约束。
4. 工程实现：编译流水线、工具链、接口和代码生成。
5. 经典项目与案例：MLIR、TVM、XLA、TorchInductor 等。
6. 实验与性能验证：编译时间、运行性能、代码质量和正确性。
7. 我们的理解与研究问题：自动化、可移植性和优化搜索。

## 分类边界

本类聚焦编译期分析、IR 变换、Lowering 和代码生成。运行时任务分配属于系统优化；手工设计单个 Kernel 的执行细节属于 Kernel 优化。

## 相关入口
- [八大优化方法总览](../01_Optimization_Methods.md)
- [我们的研究记录](../02_Our_Research_Notes.md)
