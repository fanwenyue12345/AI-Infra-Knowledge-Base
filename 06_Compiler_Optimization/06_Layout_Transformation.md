# Layout Transformation（布局变换）

**所属分类：** [06 编译器优化](README.md)  
**英文名称：** Layout Transformation  
**专题编号：** 06

## 01. 一句话定义与核心目的

**一句话定义：** Layout Transformation 是改变张量元素在内存地址、线程、Warp、寄存器或共享内存之间的映射方式，以适应目标计算和硬件访问模式的变换。

**核心目的：** 让数据布局与线程访问、向量化、矩阵指令和内存层级匹配，减少不合并访问、Bank Conflict 和额外数据重排。布局不仅是张量在内存中的形状，还包括元素由哪些线程持有、如何分布到寄存器和片上存储。

## 02. 数学原理与核心技术

### 2.1 可视化理解

![Layout Transformation](../images/06_Compiler_Optimization/06_Layout_Transformation.svg)

*图：同一逻辑矩阵可采用不同物理地址顺序，也可分配给不同线程。逻辑索引不变，但线程持有关系和访存连续性会改变。*

### 2.2 数学原理与核心技术

逻辑张量索引到线性内存地址的映射可写为：

$$
\operatorname{addr}(i_0,\ldots,i_{n-1})
=
base+\sum_{k=0}^{n-1}i_k s_k,
$$

其中 $s_k$ 是对应维度的 stride。对于连续行优先矩阵 $A\in\mathbb R^{M\times N}$，地址偏移为：

$$
offset(i,j)=iN+j.
$$

转置视图的逻辑索引不变，但 stride 会交换；若直接按不合适的线程方向读取，就可能导致非合并访存。

**核心技术：**
- **Stride / Contiguous Layout：** 描述逻辑维度与物理地址之间的步长关系。
- **Thread Layout：** 决定每个线程负责哪些逻辑元素。
- **Warp Layout：** 决定 Warp 内各 Lane 对应的数据索引。
- **Blocked Layout：** 将连续数据块分配给线程组，改善合并访问与局部性。
- **Swizzle：** 对地址或 Tile 顺序做置换，缓解 Bank Conflict 或改善硬件访问。
- **Layout Conversion：** 在寄存器、Shared Memory 和全局内存布局之间转换，可能需要数据搬运或同步。

布局变换可能提高访存效率，也可能引入额外的数据重排成本。必须结合消费者算子、数据复用、硬件指令要求和整体数据流选择布局。

## 03. 代表案例

| 项目 | 与本专题的关系 | 值得关注的内容 | 项目与资料 |
|---|---|---|---|
| Triton | **直接案例：** 显式描述张量元素在线程和 Warp 中的分布 | blocked layout、slice layout、convert_layout | [GitHub](https://github.com/triton-lang/triton) · [文档](https://triton-lang.org/main/) |
| CUTLASS | **直接案例：** 通过 TensorRef、Tile 与线程布局组织矩阵数据 | layout、threadblock swizzle、tensor memory | [GitHub](https://github.com/NVIDIA/cutlass) |
| MLIR GPU / Linalg | **直接案例：** 在 IR 中表示并变换张量与设备数据布局 | affine maps、memref layout、GPU mapping | [MLIR 文档](https://mlir.llvm.org/) |
| TVM | **直接案例：** 通过 TensorIR 与调度变换改变循环和数据访问布局 | layout transform、cache_read/write、reorder | [GitHub](https://github.com/apache/tvm) |

学习 Triton 时，建议把同一个二维 Tile 的逻辑索引、Lane 分配和最终访存地址画在一起，理解 layout 不只是 shape。
