# 中间表示（IR）

**所属分类：** [06 编译器优化](README.md)  
**英文名称：** Intermediate Representation  
**专题编号：** 02

## 01. 一句话定义与核心目的

**一句话定义：** 中间表示（IR）是编译器在源语言与目标机器指令之间使用的结构化程序表示，用于承载运算、数据类型、控制流和依赖关系。

**核心目的：** 将源语言的多样性与目标硬件的复杂性解耦，使优化可以在统一表示上实施，也让同一前端能够连接多个后端。不同层级的 IR 会保留不同程度的抽象信息。

## 02. 数学原理与核心技术

### 2.1 可视化理解

![02_Intermediate_Representation 可视化](../images/06_Compiler_Optimization/02_Intermediate_Representation.svg)

*图：高层 IR 保留张量和算子语义；中层 IR 显式表达循环和内存；低层 IR 更接近目标指令。不同编译器的层级名称并不完全相同。*

### 2.2 数学原理与核心技术

### 2.1 可视化理解

![编译器多层 IR](../images/06_Compiler_Optimization/02_Intermediate_Representation.svg)

*图：高层 IR 保留张量、算子等领域语义；中层 IR 显式表达循环和内存；低层 IR 更接近目标指令。不同编译器的 IR 层级与名称并不完全相同。*

### 2.2 数学原理与核心技术

例如矩阵乘法：

$$
C_{ij}=\sum_{k=0}^{K-1}A_{ik}B_{kj}.
$$

在高层 IR 中，它可能只是一个矩阵乘法算子；在循环层 IR 中，它可能被表示为三层循环；在 GPU 低层 IR 中，则可能进一步体现线程索引、Tile、内存搬运和矩阵指令。

**核心技术与概念：**
- **SSA（Static Single Assignment）：** 每个 SSA 值只定义一次，便于数据流分析和优化。
- **CFG（Control Flow Graph）：** 用基本块和控制流边表示分支与循环。
- **Dialect：** MLIR 中针对不同领域或抽象层定义的一组操作、类型和属性。
- **Operation / Value / Block：** IR 中常见的运算、数据值和基本控制流结构。
- **High-level / Mid-level / Low-level IR：** 分别保留领域语义、程序结构或硬件执行细节。
- **IR Verification：** 检查类型、操作约束和结构不变量，防止非法变换继续传播。

IR 不只是“编译器内部的一种代码格式”。它决定了哪些信息能被分析、哪些优化容易表达，以及不同硬件后端如何共享前端成果。

## 03. 代表案例

| 项目 | 与本专题的关系 | 值得关注的内容 | 项目与资料 |
|---|---|---|---|
| LLVM IR | **直接案例：** 基于 SSA 的低层通用 IR | LLVM Language Reference、CFG、SSA | [文档](https://llvm.org/docs/LangRef.html) · [GitHub](https://github.com/llvm/llvm-project) |
| MLIR | **直接案例：** 多层、可扩展的 IR 基础设施 | Builtin、Affine、Linalg、GPU、LLVM Dialect | [官网](https://mlir.llvm.org/) |
| OpenXLA StableHLO | **直接案例：** 用于机器学习计算图交换和编译的高层操作集合 | Tensor operations、shape、semantics | [GitHub](https://github.com/openxla/stablehlo) |
| Triton | **直接案例：** 提供面向 GPU 张量程序的多层 IR 与编译流程 | Triton IR、TTGIR、LLVM IR、PTX | [GitHub](https://github.com/triton-lang/triton) |

学习 IR 时建议追踪同一个矩阵乘法在 StableHLO、MLIR、Triton IR 和 LLVM IR 中如何逐步改变表达形式。
