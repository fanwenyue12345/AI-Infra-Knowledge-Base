# 浮点数基础

**所属分类：** [07 数值精度优化](README.md)  
**英文名称：** Floating-Point Fundamentals  
**专题编号：** 01

## 01. 一句话定义与核心目的

**一句话定义：** 浮点数是用有限位数的符号、指数和有效数表示实数近似值的计算机数值格式。

**核心目的：** 在有限存储空间内表示跨度很大的数值，并让计算机能够高效执行科学计算和机器学习运算；理解其表示范围、有效精度和舍入规则，是分析数值误差与选择 FP32、FP16、BF16、FP8 等格式的基础。

## 02. 数学原理与核心技术

### 2.1 可视化理解

![浮点数格式结构](../images/07_Numerical_Optimization/01_Floating_Point_Fundamentals.svg)

*图：浮点格式将位宽分配给符号位、指数位和尾数/有效数位。指数位主要影响动态范围，有效数位主要影响精细分辨能力；具体编码还包含偏置指数、隐含位和特殊值规则。*

### 2.2 数学原理与核心技术

一个正规二进制浮点数可概念化表示为：

$$
x=(-1)^s\times(1.f)_2\times 2^{e-b},
$$

其中 $s$ 为符号位，$f$ 为有效数小数部分，$e$ 为存储指数，$b$ 为指数偏置。实际 IEEE 754 格式还定义了非正规数、正负零、无穷大和 NaN。

**关键概念：**
- **Significand / Mantissa：** 有效数决定可区分数值的精细程度。
- **Exponent：** 指数决定数值可表示的数量级范围。
- **Precision：** 有效数位数所决定的相对分辨能力。
- **Dynamic Range：** 格式可表示的最小非零数与最大有限数之间的范围。
- **Rounding：** 精确结果不能表示时，按规定舍入到可表示数。
- **ULP：** 某个数值附近相邻可表示浮点数之间的间距。
- **Subnormal：** 非正规数用于在接近零的区域提供渐进下溢。

常见格式的侧重点不同：

| 格式 | 位宽 | 典型特点 |
|---|---:|---|
| FP32 | 32 bit | 精度和动态范围较均衡，常作参考精度 |
| FP16 | 16 bit | 比 FP32 节省存储，指数范围较窄 |
| BF16 | 16 bit | 与 FP32 相同的指数位数，尾数精度较低 |
| FP8 | 8 bit | 存储和计算成本更低，通常需要缩放和更严格的误差管理 |

浮点加法通常不满足结合律：

$$
(a+b)+c\neq a+(b+c),
$$

因为每一步舍入可能不同。因此并行归约、算子融合和混合精度可能改变数值结果，即使数学表达式看起来相同。

## 03. 代表案例

| 项目 | 与本专题的关系 | 值得关注的内容 | 项目与资料 |
|---|---|---|---|
| IEEE 754 | **基础标准：** 定义二进制浮点格式、舍入和特殊值行为 | binary32、binary16、rounding、subnormal | [IEEE 754 标准](https://standards.ieee.org/ieee/754/6210/) |
| NumPy | **直接案例：** 提供多种浮点数据类型及类型转换 | dtype、astype、finfo、精度范围 | [GitHub](https://github.com/numpy/numpy) · [文档](https://numpy.org/doc/stable/user/basics.types.html) |
| NVIDIA CUDA | **直接资料：** 说明 GPU 浮点格式和运算行为 | FP16、BF16、TF32、FP8 支持 | [CUDA Programming Guide](https://docs.nvidia.com/cuda/cuda-c-programming-guide/) |
| PyTorch | **直接案例：** 张量 dtype 决定运算与存储精度 | dtype promotion、autocast、precision controls | [GitHub](https://github.com/pytorch/pytorch) · [文档](https://docs.pytorch.org/docs/stable/tensors.html) |
