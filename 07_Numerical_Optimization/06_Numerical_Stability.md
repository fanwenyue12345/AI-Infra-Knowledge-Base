# 数值稳定性

**所属分类：** [07 数值精度优化](README.md)  
**英文名称：** Numerical Stability  
**专题编号：** 06

## 01. 一句话定义与核心目的

**一句话定义：** 数值稳定性描述数值算法在有限精度计算和输入扰动下，是否能产生与原问题精确解保持合理接近的结果。

**核心目的：** 避免算法把微小舍入误差放大成显著结果偏差，保证计算在不同数据规模、输入范围和精度设置下可靠。稳定性关注算法如何处理误差；条件数则描述问题本身对输入扰动的敏感程度。

## 02. 数学原理与核心技术

### 2.1 可视化理解

![数值稳定性对比](../images/07_Numerical_Optimization/06_Numerical_Stability.svg)

*图：稳定算法中误差增长受到控制；不稳定计算可能放大舍入扰动。示意图强调误差趋势，实际稳定性需针对具体算法和问题分析。*

### 2.2 数学原理与核心技术

对问题 $y=f(x)$，输入扰动 $\Delta x$ 引起输出扰动 $\Delta y$。局部相对敏感程度可由条件数刻画：

$$
\kappa_f(x)
\approx
\left|\frac{x f'(x)}{f(x)}\right|.
$$

条件数大表示问题本身对输入误差敏感；即使使用稳定算法，也可能无法消除问题固有的敏感性。

**Backward Stability（后向稳定性）** 是分析算法稳定性的核心概念之一：计算结果 $\hat{y}$ 可以看作某个轻微扰动后问题的精确解。对于线性方程组 $Ax=b$，若计算结果满足：

$$
(A+\Delta A)\hat{x}=b+\Delta b,
$$

且 $\Delta A$、$\Delta b$ 相对较小，则算法在后向意义上稳定。前向误差还会受到问题条件数影响。

**核心技术与概念：**
- **Condition Number：** 衡量问题对输入扰动的敏感性。
- **Forward Error：** 计算结果与精确解之间的差异。
- **Backward Error：** 使计算结果成为精确解所需的最小问题扰动。
- **Stable Reformulation：** 用数值更稳健的等价表达替换容易发生消去或溢出的表达。
- **Scaling / Normalization：** 调整变量和方程尺度，改善数值范围与条件。
- **Stable Reduction：** 采用 Pairwise Summation、补偿求和等方法降低累积误差。
- **Pivoting：** 在线性求解中通过选主元改善数值行为。

一个重要区别是：病态问题不等于算法不稳定；问题条件数高时，输入微小变化就可能导致解显著变化。应同时分析问题条件和算法稳定性。

## 03. 代表案例

| 项目 | 与本专题的关系 | 值得关注的内容 | 项目与资料 |
|---|---|---|---|
| LAPACK | **经典直接案例：** 提供经过数值分析设计的线性代数算法 | pivoting、factorization、condition estimation | [GitHub](https://github.com/Reference-LAPACK/lapack) |
| Higham《Accuracy and Stability》 | **经典理论资料：** 系统讨论条件数、前向/后向误差与算法稳定性 | backward stability、conditioning | [SIAM 图书](https://epubs.siam.org/doi/book/10.1137/1.9780898718027) |
| NumPy / SciPy | **直接案例：** 提供稳定的线性代数与数值计算接口 | linalg.solve、cond、SVD | [NumPy](https://github.com/numpy/numpy) · [SciPy](https://github.com/scipy/scipy) |
| cuSOLVER | **直接案例：** GPU 线性代数求解器，包含分解、求解和条件估计工具 | factorization、solver、iterative refinement | [官方文档](https://docs.nvidia.com/cuda/cusolver/) |
