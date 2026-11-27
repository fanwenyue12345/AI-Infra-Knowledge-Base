# GPU 存储层级

**所属分类：** [03 内存优化](README.md)  
**英文名称：** GPU Memory Hierarchy  
**专题编号：** 01

## 01. 一句话定义与核心目的

**一句话定义：** GPU 存储层级是由寄存器、片上共享存储与缓存、设备显存等不同容量、带宽和访问特性的存储资源构成的分层体系。

**核心目的：** 让计算单元尽可能快速地获得所需数据。高性能 Kernel 需要理解数据位于哪一层、如何在层级之间移动，以及如何通过局部性和复用减少对较慢存储层的访问。存储层级并非所有 GPU 完全相同，具体结构和容量取决于架构。

## 02. 数学原理与核心技术

### 2.1 可视化理解

![01_GPU_Memory_Hierarchy 可视化](../images/03_Memory_Optimization/01_GPU_Memory_Hierarchy.svg)

*图：从寄存器、共享内存与缓存到设备显存，容量、作用范围和访问特性不同。层级关系为概念化展示，具体架构因 GPU 而异。*

### 2.2 数学原理与核心技术

### 2.1 可视化理解

![GPU 存储层级](../images/03_Memory_Optimization/01_GPU_Memory_Hierarchy.svg)

*图：从线程私有寄存器、线程块可协作访问的 Shared Memory，到缓存和设备显存，容量通常逐级增大；访问代价、作用范围和管理方式各不相同。图为概念层级，不代表所有 GPU 的严格物理连接顺序。*

### 2.2 数学原理与核心技术

理解 GPU 存储层级，核心是区分**容量、访问延迟、带宽、作用范围和可编程控制方式**。

- **Registers（寄存器）：** 通常由线程私有，用于保存当前计算的操作数和中间结果。寄存器资源有限，单线程使用过多会限制一个 SM 同时驻留的线程块数量。
- **Shared Memory（共享内存）：** 由同一线程块中的线程协作访问，适合显式暂存数据、线程间交换数据和分块复用。
- **L1 Cache：** SM 附近的缓存资源，具体与 Shared Memory 的容量配置关系依架构而异。
- **L2 Cache：** 通常由设备上多个 SM 共享，用于缓存跨 SM 访问的数据。
- **Global Memory / HBM 或 GDDR：** 容量较大，是 GPU 设备内存的主要来源；访问延迟相对较高，但可提供很高的总带宽。

可用一个简化的性能模型理解数据访问：

$$
T_{\mathrm{memory}}\approx
\frac{D}{B_{\mathrm{effective}}}
$$

其中 $D$ 是需要传输的数据量，$B_{\mathrm{effective}}$ 是实际有效带宽。若数据在片上被重复使用，便可能减少从较低层级重新读取的数据量。

**核心技术：**
- **Locality（局部性）：** 时间局部性指近期访问的数据可能再次使用；空间局部性指相邻地址的数据可能连续使用。
- **Register Tiling：** 将当前 Tile 的部分数据保存在寄存器中，减少重复加载。
- **Shared-Memory Tiling：** 将数据块搬到共享内存，让线程块内多个线程复用。
- **Cache Reuse：** 利用缓存自动保留近期访问的数据。
- **Memory Capacity Management：** 控制寄存器、共享内存和全局显存占用，避免资源压力限制并行度。

需要注意：寄存器并非“总是比共享内存快就应该尽可能多用”。过高的寄存器占用可能降低 Occupancy，过大的共享内存 Tile 也可能限制并发线程块数量。优化目标是整体执行时间，而不是单独追求某一层存储使用量最大化。

## 03. 代表案例

| 项目 | 与本专题的关系 | 值得关注的内容 | 项目与资料 |
|---|---|---|---|
| CUDA Programming Guide | **直接资料：** 解释 CUDA 线程层级、设备内存、共享内存与缓存机制 | Memory hierarchy、Shared Memory、Register 使用 | [官方文档](https://docs.nvidia.com/cuda/cuda-c-programming-guide/) |
| Triton | **直接案例：** 以块级编程表达数据加载、布局和片上复用 | Block pointer、layout、num_warps、shared-memory lowering | [GitHub](https://github.com/triton-lang/triton) · [文档](https://triton-lang.org/main/) |
| CUTLASS | **直接案例：** 将 GEMM 数据分块并映射到寄存器、共享内存和 Tensor Core | 多级 Tile、pipeline、线程布局 | [GitHub](https://github.com/NVIDIA/cutlass) |
| FlashAttention | **关联案例：** 通过分块和在线 Softmax 累积，减少对 HBM 中间矩阵的读写 | IO-aware 算法、片上数据复用 | [GitHub](https://github.com/Dao-AILab/flash-attention) · [论文](https://arxiv.org/abs/2205.14135) |

建议先通过 CUDA Programming Guide 建立存储层级概念，再阅读 CUTLASS 和 FlashAttention，观察算法如何围绕片上存储容量组织数据。
