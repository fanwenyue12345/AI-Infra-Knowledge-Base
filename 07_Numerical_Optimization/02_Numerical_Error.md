# 数值误差

**所属分类：** [07 数值精度优化](README.md)  
**英文名称：** Numerical Error  
**专题编号：** 02

## 01. 一句话定义与核心目的

**一句话定义：** 数值误差是计算机数值结果与所定义的精确数学结果之间的偏差，来源包括输入数据误差、浮点舍入、截断近似和算法本身的误差传播。

**核心目的：** 定量描述计算结果偏离真实值的程度，区分误差从哪里产生、如何传播，以及哪些误差会被算法放大，从而判断结果是否可信并选择合适的数值方法和精度。

## 02. 数学原理与核心技术

### 2.1 可视化理解

![数值误差可视化](../images/07_Numerical_Optimization/02_Numerical_Error.svg)

*图：左侧表示实数在浮点数集合中的离散表示与舍入；右侧示意两个接近数相减时，有效数字可能发生消去。*

### 2.2 数学原理与核心技术

给定精确值 $x$ 和计算近似值 $\hat{x}$，常用误差定义为：

**绝对误差：**

$$
E_{\mathrm{abs}}=|\hat{x}-x|
$$

**相对误差：**

$$
E_{\mathrm{rel}}=\frac{|\hat{x}-x|}{|x|},\qquad x\ne 0
$$

浮点基本运算常用模型表示为：

$$
\operatorname{fl}(a\circ b)=(a\circ b)(1+\delta),
\qquad |\delta|\le u
$$

其中 $\circ$ 表示满足模型条件的基本运算，$u$ 是 unit roundoff。该模型并非对溢出、下溢、NaN 或所有边界情形都成立。

**核心技术与概念：**

- **Rounding Error（舍入误差）：** 精确运算结果无法由当前浮点格式表示时产生。
- **Truncation Error（截断误差）：** 用有限项级数、有限步迭代或离散网格近似连续对象时产生。
- **Cancellation（消去误差）：** 两个接近数相减，导致高位有效数字抵消，剩余结果对输入误差更敏感。
- **Condition Number（条件数）：** 描述问题本身对输入扰动的敏感程度；问题病态不等同于算法不稳定。
- **Forward / Backward Error：** 前向误差衡量计算结果与真实解的距离；后向误差衡量结果能否看作某个轻微扰动问题的精确解。
- **Error Propagation：** 研究局部误差如何沿运算链、迭代过程或计算图传播。

需要区分三个概念：问题的条件数描述问题本身的敏感性；算法稳定性描述算法如何传播误差；最终误差则同时受两者和输入误差影响。

## 03. 代表案例

| 项目 / 资料 | 与本专题的关系 | 值得关注的内容 | 链接 |
|---|---|---|---|
| Higham, *Accuracy and Stability of Numerical Algorithms* | **经典参考：** 系统介绍舍入误差、条件数和稳定性分析 | 前向误差、后向误差、误差传播 | [SIAM 图书](https://epubs.siam.org/doi/book/10.1137/1.9780898718027) |
| NumPy | **实践工具：** 提供浮点精度、最大值、最小正规数和间距信息 | `numpy.finfo`、`spacing`、dtype | [官方文档](https://numpy.org/doc/stable/reference/generated/numpy.finfo.html) |
| Python math.fsum | **直接案例：** 使用部分和策略改善浮点求和精度 | 多项浮点求和与舍入误差 | [官方文档](https://docs.python.org/3/library/math.html#math.fsum) |
| LAPACK | **工程案例：** 数值线性代数算法中系统考虑误差与稳定性 | 线性方程求解、分解、误差估计 | [GitHub](https://github.com/Reference-LAPACK/lapack) |
