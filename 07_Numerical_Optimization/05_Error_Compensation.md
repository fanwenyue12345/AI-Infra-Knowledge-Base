# 误差补偿

**所属分类：** [07 数值精度优化](README.md)  
**英文名称：** Error Compensation  
**专题编号：** 05

## 01. 一句话定义与核心目的

**一句话定义：** 误差补偿是在数值计算过程中估计、保存或修正已产生的误差，使后续计算能够部分恢复被舍入、截断或近似丢失的信息。

**核心目的：** 在不必全面提高数据精度或计算成本的情况下，改善数值结果的准确性。误差补偿可用于求和、迭代算法、低精度矩阵运算和量化系统；不同场景的误差来源与补偿机制并不相同。

## 02. 数学原理与核心技术

### 2.1 可视化理解

![补偿求和可视化](../images/07_Numerical_Optimization/05_Error_Compensation.svg)

*图：普通求和会丢弃当前舍入误差；Kahan 求和维护补偿变量，将部分丢失的低位信息带入后续运算。*

### 2.2 数学原理与核心技术

以 Kahan compensated summation 为例，目标是计算：

$$
S=\sum_{i=1}^{n}x_i
$$

普通浮点累加可能在每一步舍入。Kahan 算法维护累加值 $s$ 和补偿量 $c$：

$$
y_i=x_i-c
$$

$$
t=s+y_i
$$

$$
c=(t-s)-y_i
$$

$$
s=t
$$

其中 $c$ 估计本次加法中未能保留的低位信息，并在下一步加入。不同文献对补偿变量的符号定义可能不同，但更新逻辑等价。

**核心技术：**

- **Kahan Summation：** 用补偿变量改善浮点求和精度。
- **Neumaier Summation：** 对不同大小量级的加数提供更稳健的补偿求和变体。
- **Pairwise Summation：** 将求和组织为树形分组，常能降低误差增长并利于并行化。
- **Compensated Dot Product：** 对乘积和累加中的误差进行补偿，具体实现依赖 FMA 等硬件能力。
- **Residual / Error Feedback：** 在量化或低精度更新中保存未表达的残差，并在后续步骤反馈。
- **Iterative Refinement：** 先用较低精度求近似解，再用残差修正解，必要时迭代多次。

误差补偿不能保证所有情况下都得到精确结果。其效果取决于误差模型、运算顺序、溢出下溢和实现精度；补偿变量自身也会受到浮点舍入影响。

## 03. 代表案例

| 项目 / 资料 | 与本专题的关系 | 值得关注的内容 | 项目与资料 |
|---|---|---|---|
| Python `math.fsum` | **直接案例：** 使用部分和策略改善浮点求和精度 | 多部分累加、舍入误差处理 | [官方文档](https://docs.python.org/3/library/math.html#math.fsum) |
| NVIDIA CUB | **关联案例：** 提供 GPU reduction / scan 原语，可用于比较不同归约组织方式与精度行为 | DeviceReduce、归约树、累加类型 | [GitHub](https://github.com/NVIDIA/cccl) |
| ExBLAS | **直接案例：** 提供高精度、可复现的浮点归约与点积算法 | long accumulator、精确或可复现累加 | [GitHub](https://github.com/riakymch/exblas) |
| Iterative Refinement / LAPACK | **直接案例：** 使用残差修正低精度线性系统求解结果 | 混合精度迭代改进、残差计算 | [LAPACK GitHub](https://github.com/Reference-LAPACK/lapack) |
| Deep Learning Gradient Compression with Error Feedback | **关联案例：** 在梯度压缩中保留量化残差并反馈到后续迭代 | Error Feedback、压缩误差累积 | [论文检索入口](https://arxiv.org/search/?query=error+feedback+gradient+compression&searchtype=all) |
