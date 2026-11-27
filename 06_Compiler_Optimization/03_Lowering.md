# Lowering（逐层降低）

**所属分类：** [06 编译器优化](README.md)  
**英文名称：** Compiler Lowering  
**专题编号：** 03

## 01. 一句话定义与核心目的

**一句话定义：** Lowering 是把高层、抽象的程序表示逐步转换为更低层、更接近目标硬件执行机制的表示过程。

**核心目的：** 将高级张量算子、循环和并行抽象逐步落实为具体的数据类型、内存访问、线程布局、硬件指令和调用约定，同时在每一层保留必要语义并暴露新的优化机会。

## 02. 数学原理与核心技术

### 2.1 可视化理解

![03_Lowering 可视化](../images/06_Compiler_Optimization/03_Lowering.svg)

*图：矩阵乘法从高层算子逐步降低为循环、Tile、线程映射、内存操作和目标指令。具体阶段可能合并、交错或重复执行。*

### 2.2 数学原理与核心技术

### 2.1 可视化理解



### 2.2 数学原理与核心技术

高层表达：

$$
C=AB
$$

可以先展开为循环：

$$
C_{ij}=\sum_{k=0}^{K-1}A_{ik}B_{kj}.
$$

随后再通过 Tiling 将循环迭代空间划分成小块，并映射到线程块、Warp 和矩阵硬件。Lowering 的核心不是单纯“展开公式”，而是逐步明确原先隐藏的执行细节。

**核心技术：**
- **Dialect Conversion：** 将一种 IR Dialect 转换为另一种 Dialect。
- **Loop Lowering：** 将张量或向量操作变成显式循环。
- **Bufferization：** 将张量值语义转换为显式内存缓冲区和读写操作。
- **GPU Lowering：** 将并行循环或 Tile 映射为 Grid、Block、线程与设备内存操作。
- **Instruction Lowering：** 将抽象矩阵操作、归约或向量操作转换为目标 ISA 支持的指令。
- **Legality / Conversion Target：** 规定哪些高层操作必须消除、哪些操作允许保留，保证转换结果满足目标层要求。

Lowering 不是每次都单向、不可逆地变得更低级。编译器可能在不同层之间反复执行规范化和优化；过早丢失高层信息，也可能让某些优化更难进行。

## 03. 代表案例

| 项目 | 与本专题的关系 | 值得关注的内容 | 项目与资料 |
|---|---|---|---|
| MLIR | **直接案例：** 提供 Dialect Conversion 与渐进式 Lowering 机制 | ConversionTarget、RewritePattern、Linalg-to-LLVM | [官网](https://mlir.llvm.org/) · [GitHub](https://github.com/llvm/llvm-project) |
| Triton | **直接案例：** 将块级张量程序逐步降低到 GPU 线程和目标代码 | Triton IR、GPU IR、LLVM/PTX lowering | [GitHub](https://github.com/triton-lang/triton) |
| IREE | **直接案例：** 面向异构设备的端到端编译与运行时系统 | Flow、Stream、HAL、Codegen | [GitHub](https://github.com/iree-org/iree) |
| TVM | **直接案例：** 从张量程序与调度表示生成目标设备代码 | TensorIR、lower、target codegen | [GitHub](https://github.com/apache/tvm) |

建议沿着 MLIR 的 Linalg → SCF / Affine → GPU / LLVM Dialect 示例，观察一个张量算子怎样逐步显式化。
