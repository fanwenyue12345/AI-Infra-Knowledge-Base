# 数据复用

**所属分类：** [03 内存优化](README.md)  
**英文名称：** Data Reuse  
**专题编号：** 03

## 01. 一句话定义与核心目的

**一句话定义：** 数据复用是让一次从较慢存储层读取的数据，在被覆盖或释放之前服务于多个计算操作，避免重复搬运相同数据。

**核心目的：** 减少全局显存访问，提高每次数据加载所产生的有效计算量。数据复用是矩阵乘法、卷积、Attention 和科学计算 Kernel 的重要优化基础。

## 02. 数学原理与核心技术

### 2.1 可视化理解

![数据复用可视化](../images/03_Memory_Optimization/03_Data_Reuse.svg)

*图：矩阵乘法中，一个 A 元素会参与同一行多个输出的计算，一个 B 元素会参与同一列多个输出的计算。将数据块暂存于片上存储，可以让多个乘加操作复用它们。*

### 2.2 数学原理与核心技术

考虑矩阵乘法：

$$
C=AB,\qquad
C_{ij}=\sum_{k=1}^{K}A_{ik}B_{kj}.
$$

同一个 $A_{ik}$ 会被不同列 $j$ 的输出重复使用；同一个 $B_{kj}$ 会被不同的行 $i$ 重复使用。如果每次使用都重新从 HBM 读取，就会产生重复数据传输。

**Arithmetic Intensity（算术强度）** 可以用来描述计算与数据传输的比例：

$$
AI=\frac{\text{FLOPs}}{\text{Bytes transferred}}.
$$

提高数据复用通常能减少分母中的数据传输量，从而提高 AI；但具体收益取决于数据块能否放进片上存储，以及线程布局和同步开销。

**核心技术：**
- **Temporal Reuse（时间复用）：** 同一数据在不同时间的多个计算步骤中重复使用。
- **Spatial Reuse（空间复用）：** 相邻计算共享相同或相邻的数据，例如卷积窗口重叠。
- **Register Reuse：** 将数据保存在寄存器中，由同一线程执行多次计算。
- **Shared-Memory Reuse：** 线程块协作加载数据，再由多个线程复用。
- **Cache Reuse：** 利用硬件缓存自动保留近期访问的数据。
- **Loop Reordering / Blocking：** 调整循环顺序和分块大小，使工作集更适合缓存或片上存储。

数据复用并非越多越好。过大的复用 Tile 可能导致寄存器或共享内存占用过高，降低并行度；为复用而增加的数据搬运和同步也可能抵消收益。

## 03. 代表案例

| 项目 | 与本专题的关系 | 值得关注的内容 | 项目与资料 |
|---|---|---|---|
| CUTLASS | **直接案例：** GEMM 中复用 A、B Tile，并组织多级存储流水 | Threadblock / Warp / Instruction Tile | [GitHub](https://github.com/NVIDIA/cutlass) |
| FlashAttention | **直接案例：** 分块加载 Q/K/V，在线更新 Softmax 统计量，避免完整注意力矩阵落入 HBM | IO-aware 算法与片上复用 | [GitHub](https://github.com/Dao-AILab/flash-attention) · [论文](https://arxiv.org/abs/2205.14135) |
| NVIDIA CUDA Samples | **直接案例：** 通过矩阵乘法和卷积示例展示共享内存复用 | Tiled matrix multiplication、convolution | [GitHub](https://github.com/NVIDIA/cuda-samples) |
| Triton | **直接案例：** 使用块级编程表达 Tile 加载、复用和计算 | GEMM tutorial、block pointers、layout | [GitHub](https://github.com/triton-lang/triton) |

建议把数据复用与下一篇 Tiling 连起来学习：数据复用说明“为什么值得重复使用”，Tiling 说明“如何把可复用的数据组织成块”。
