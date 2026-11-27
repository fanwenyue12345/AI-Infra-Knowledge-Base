# 稀疏计算

**所属分类：** [01 算法与数学优化](README.md)  
**英文名称：** Sparse Computation  
**专题编号：** 04

## 01. 一句话定义与核心目的

**一句话定义：** 稀疏计算利用矩阵、张量或计算图中大量元素为零或无需参与运算的结构，只对必要的数据和操作进行处理。

**核心目的：** 避免对零值或不重要元素执行无效计算，降低存储、算术和数据搬运成本。实际收益取决于稀疏度、稀疏模式、索引开销以及硬件对不规则访问的支持。

## 02. 数学原理与核心技术

### 2.1 可视化理解

![NVIDIA NVPL Sparse CSR storage format](https://docs.nvidia.com/nvpl/latest/sparse/_images/csr.png)

*图源：NVIDIA NVPL Sparse 官方文档。展示稠密矩阵中的非零元素如何映射到 CSR 的 row offsets、column indices 与 values。参见[NVIDIA 原始文档](https://docs.nvidia.com/nvpl/latest/sparse/storage_format/sparse_matrix.html)。*

*图：左侧用蓝色单元标出 4×4 矩阵的非零元素；中间列出其坐标和值；右侧给出 COO 的 row、col、data 三个数组。该例有 3 个非零值，稀疏格式还需要额外索引存储。*

### 2.2 数学原理与核心技术


若矩阵 $A\in\mathbb R^{m\times n}$ 只有 $nnz$ 个非零元素，稠密存储需要保存 $mn$ 个数；稀疏表示则只保存非零值及其位置。

稠密矩阵向量乘法约需 $2mn$ FLOPs；稀疏矩阵向量乘法约需 $2nnz$ FLOPs，另有索引读取和不规则访存成本。

核心技术：
- **CSR（Compressed Sparse Row）：** 按行存储非零值、列索引和行指针。
- **CSC（Compressed Sparse Column）：** 按列压缩存储，适合某些列操作。
- **COO（Coordinate）：** 直接保存非零元素的坐标和值，格式简单，常用于构建和转换。
- **Block Sparsity：** 按块组织非零结构，改善规则性和硬件利用率。
- **Structured Sparsity：** 通过固定模式（例如 N:M）让硬件更容易加速。
- **Sparse Attention：** 只计算局部、选定或重要的注意力连接。

稀疏计算不一定比稠密计算快：当矩阵较小、稀疏度不够高或访问过于不规则时，索引与访存开销可能抵消节省的 FLOPs。

## 03. 代表案例

| 项目 | 采用的技术与价值 | 项目链接 |
|---|---|---|
| SuiteSparse | 稀疏矩阵算法与数据结构的重要软件集合 | [GitHub](https://github.com/DrTimothyAldenDavis/SuiteSparse) |
| SciPy Sparse | Python 科学计算中的 CSR、CSC、COO 等稀疏矩阵接口 | [GitHub](https://github.com/scipy/scipy) · [文档](https://docs.scipy.org/doc/scipy/reference/sparse.html) |
| NVIDIA cuSPARSE | GPU 稀疏线性代数库，提供稀疏矩阵运算与格式支持 | [官方文档](https://docs.nvidia.com/cuda/cusparse/) |
| xFormers | 提供多种高效注意力实现，包括部分稀疏注意力机制 | [GitHub](https://github.com/facebookresearch/xformers) |

建议先理解 CSR/CSC 的存储方式，再比较稀疏 GEMM、稀疏 Attention 在不同稀疏模式下的实际收益。
