# 数值稳定性

**所属分类：** [07 数值精度优化](README.md)  
**英文名称：** Numerical Stability  
**专题编号：** 06

## 01. 一句话定义与核心目的

**一句话定义：** 数值稳定性描述一个算法在有限精度计算中对舍入误差和扰动的控制能力，即计算误差是否会被算法过程过度放大。

**核心目的：** 让算法在真实浮点硬件上仍能得到可信结果，避免理论上正确的公式因溢出、下溢、消去误差或误差累积而产生严重偏差。稳定算法尤其重要于深度学习、线性代数、科学仿真和迭代求解。

## 02. 数学原理与核心技术

### 2.1 可视化理解

![稳定 Softmax 可视化](../images/07_Numerical_Optimization/06_Numerical_Stability.svg)

*图：Softmax 的数学结果在所有输入同时减去同一个常数后保持不变。先减去最大值可使指数输入不大于零，避免大正数指数导致溢出。*

### 2.2 数学原理与核心技术

以 Softmax 为例：

$$
\operatorname{softmax}(x_i)=
\frac{e^{x_i}}{\sum_j e^{x_j}}
$$

当输入包含很大的正数时，直接计算 $e^{x_i}$ 可能溢出。令：

$$
m=\max_j x_j
$$

利用指数的平移性质：

$$
\operatorname{softmax}(x_i)=
\frac{e^{x_i-m}}{\sum_j e^{x_j-m}}
$$

因为 $x_i-m\le 0$，指数项不会因大正输入而溢出。这是数学等价但数值更稳定的计算形式。

**核心技术：**

- **Stable Reformulation：** 用数学等价但数值更稳定的表达式计算，例如 log-sum-exp、稳定 Softmax。
- **Scaling / Normalization：** 在合适位置缩放数据，避免中间量超出可表示范围。
- **Pivoting：** 在线性方程求解和矩阵分解中选择合适主元，降低误差放大风险。
- **Conditioning Awareness：** 识别问题本身是否病态，避免把问题敏感性误判为算法实现错误。
- **Stable Reduction：** 选择合理的归约顺序、累加精度或补偿策略。
- **Regularization：** 在病态问题或反问题中引入约束，改善问题条件，但会改变原问题或引入偏差。

需要区分稳定性与精度：稳定算法对舍入误差控制较好，但若问题本身高度病态，结果仍可能对输入扰动敏感；反之，某次计算误差较小也不能单独证明算法稳定。

## 03. 代表案例

| 项目 | 与本专题的关系 | 值得关注的内容 | 项目与资料 |
|---|---|---|---|
| LAPACK | **直接案例：** 数值线性代数库大量算法围绕稳定分解、主元选择和误差控制设计 | LU with pivoting、QR、SVD、误差估计 | [GitHub](https://github.com/Reference-LAPACK/lapack) |
| PyTorch | **直接案例：** 提供数值稳定的算子实现与数值精度说明 | Softmax、LogSumExp、归约精度与数值测试 | [GitHub](https://github.com/pytorch/pytorch) · [Numerical Accuracy](https://docs.pytorch.org/docs/stable/notes/numerical_accuracy.html) |
| SciPy | **直接案例：** 科学计算算法库，提供稳定的线性代数、特殊函数和优化实现 | 线性代数、数值积分、特殊函数 | [GitHub](https://github.com/scipy/scipy) · [文档](https://docs.scipy.org/doc/scipy/) |
| Higham, *Accuracy and Stability of Numerical Algorithms* | **经典理论参考：** 系统讨论稳定算法、条件数和误差分析 | Backward Stability、Forward Error、Condition Number | [SIAM 图书](https://epubs.siam.org/doi/book/10.1137/1.9780898718027) |
