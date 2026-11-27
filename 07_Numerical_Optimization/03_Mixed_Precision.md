# 混合精度

**所属分类：** [07 数值精度优化](README.md)  
**英文名称：** Mixed Precision  
**专题编号：** 03

## 01. 一句话定义与核心目的

**一句话定义：** 混合精度是在同一个算法或训练流程中，根据不同计算阶段的数值敏感性，组合使用两种或多种数值精度格式。

**核心目的：** 在尽可能保持结果质量和收敛性的前提下，利用低精度格式减少数据存储与搬运成本，并使用高吞吐硬件单元加速计算；对敏感操作则保留较高精度，避免误差累积、梯度下溢或训练不稳定。

## 02. 数学原理与核心技术

### 2.1 可视化理解

![混合精度训练流程](../images/07_Numerical_Optimization/03_Mixed_Precision.svg)

*图：示意一种常见训练策略：低精度副本用于前向和反向计算，梯度或关键归约按需要使用 FP32，主权重和优化器更新保留较高精度。具体策略随硬件、框架和训练算法而变化。*

### 2.2 数学原理与核心技术

混合精度不是单一数据类型，而是对不同操作选择不同数值格式。以矩阵乘法为例，可以使用低精度输入：

$$
A_{\mathrm{low}},B_{\mathrm{low}}
$$

并在较高精度中累积乘积：

$$
C_{\mathrm{high}}=
\operatorname{Accumulate}_{\mathrm{high}}
(A_{\mathrm{low}}B_{\mathrm{low}}).
$$

这样可以利用低精度矩阵乘法单元，同时减轻累积过程的舍入误差。具体支持的输入格式和累积格式取决于硬件指令与库实现。

**核心技术：**

- **FP16 / BF16 / FP8：** 作为低精度计算或存储格式；不同格式的指数范围和尾数精度不同。
- **FP32 Accumulation：** 低精度乘法结果在 FP32 中累积，常用于矩阵乘法和归约。
- **Master Weights：** 在某些训练方案中保留 FP32 主权重，低精度副本用于计算，更新后再转换。
- **Loss Scaling：** 对 FP16 训练中的损失或梯度进行缩放，降低小梯度下溢风险；更新前需要正确反缩放并处理 Inf/NaN。
- **Autocast：** 框架根据算子类型和策略自动选择输入精度。
- **FP8 Scaling：** 通过缩放因子将张量映射到 FP8 可表示范围，常见策略包括按张量或按块缩放。

混合精度需要验证的不只是单次算子误差，还包括训练损失、梯度、收敛速度和最终任务指标。BF16 的指数范围较大，但尾数精度低于 FP16；不能简单认为某一种格式在所有场景都更好。

## 03. 代表案例

| 项目 | 与本专题的关系 | 值得关注的内容 | 项目与资料 |
|---|---|---|---|
| NVIDIA Transformer Engine | **直接案例：** 为 Transformer 训练提供 FP8 与混合精度支持 | FP8 recipes、scaling、延迟缩放、精度保护 | [GitHub](https://github.com/NVIDIA/TransformerEngine) · [文档](https://docs.nvidia.com/deeplearning/transformer-engine/) |
| PyTorch AMP | **直接案例：** 通过 autocast 和 GradScaler 实现自动混合精度训练 | 算子精度策略、FP16 loss scaling、BF16 autocast | [GitHub](https://github.com/pytorch/pytorch) · [AMP 文档](https://docs.pytorch.org/docs/stable/amp.html) |
| NVIDIA Apex | **关联案例：** 提供历史上广泛使用的混合精度训练工具与示例 | O1/O2 等混合精度策略及训练实践 | [GitHub](https://github.com/NVIDIA/apex) |
| DeepSpeed | **直接案例：** 在大规模训练中支持 FP16、BF16 等训练精度策略 | mixed precision、loss scaling、分布式训练 | [GitHub](https://github.com/deepspeedai/DeepSpeed) · [文档](https://www.deepspeed.ai/) |
