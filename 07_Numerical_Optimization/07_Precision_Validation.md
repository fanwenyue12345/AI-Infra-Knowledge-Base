# 精度验证

**所属分类：** [07 数值精度优化](README.md)  
**英文名称：** Precision Validation  
**专题编号：** 07

## 01. 一句话定义与核心目的

**一句话定义：** 精度验证是将优化实现的数值结果与可信参考结果或明确的误差标准比较，判断低精度、近似和变换后的计算是否满足正确性要求。

**核心目的：** 防止性能优化以不可接受的数值偏差换取速度，建立可重复、可量化的正确性证据。精度验证应覆盖不同输入规模、数据类型、边界情况和实际任务指标，而不是只检查单个随机样本。

## 02. 数学原理与核心技术

### 2.1 可视化理解

![精度验证对比](../images/07_Numerical_Optimization/07_Precision_Validation.svg)

*图：优化实现与高精度参考值逐元素比较，统计绝对/相对误差，并检查最大误差、均方误差和任务级指标是否满足预设阈值。*

### 2.2 数学原理与核心技术

设参考结果为 $y$，待验证实现输出为 $\hat{y}$。常用指标包括最大绝对误差：

$$
E_{max}=\max_i|\hat{y}_i-y_i|,
$$

均方根误差：

$$
RMSE=
\sqrt{\frac{1}{N}\sum_{i=1}^{N}(\hat{y}_i-y_i)^2},
$$

以及相对误差。对接近零的参考值，单纯相对误差会不稳定，因此常使用带绝对容差的判据：

$$
|\hat{y}_i-y_i|
\leq atol+rtol\,|y_i|.
$$

其中 $atol$ 是绝对容差，$rtol$ 是相对容差。

**核心技术：**
- **Reference Implementation：** 使用高精度、成熟库或可信算法构建参考结果。
- **Elementwise Error：** 统计逐元素误差、最大误差和误差分布。
- **Tolerance Test：** 使用 atol/rtol 等阈值判断是否通过。
- **Randomized Testing：** 多种随机输入、不同分布和不同形状测试。
- **Edge-case Testing：** 覆盖零值、极大/极小值、非连续布局、边界尺寸和特殊值。
- **Gradient Validation：** 对可微算子比较梯度，必要时使用有限差分或高精度参考。
- **Task-level Validation：** 对模型推理/训练验证准确率、损失或下游任务指标。
- **Reproducibility：** 固定版本、硬件、随机种子和测量设置，记录验证条件。

性能报告应同时记录精度条件和速度结果。若某项优化只在特定 Shape、输入范围或精度阈值下成立，应明确写出适用范围，不能把局部通过描述成普遍正确。

## 03. 代表案例

| 项目 | 与本专题的关系 | 值得关注的内容 | 项目与资料 |
|---|---|---|---|
| PyTorch Testing | **直接工具：** 提供张量比较、误差容差和数值测试设施 | assert_close、gradcheck、dtype tests | [GitHub](https://github.com/pytorch/pytorch) · [Testing 文档](https://docs.pytorch.org/docs/stable/testing.html) |
| NVIDIA Transformer Engine | **直接案例：** FP8/混合精度工作流包含精度策略和数值检查 | FP8 recipe、scaling、accuracy validation | [GitHub](https://github.com/NVIDIA/TransformerEngine) |
| oneDNN | **直接案例：** 高性能算子库需在多数据类型和实现路径下维持数值正确性 | numerical behavior、primitive tests、data types | [GitHub](https://github.com/oneapi-src/oneDNN) |
| ReWY | **关联案例：** 公开结果中特别标注 precision-qualified 结果，说明性能数据需要与精度验证绑定 | full saving、精度门槛、性能结果筛选 | [GitHub](https://github.com/flashserve/ReWY) |

建议把“数值正确性测试”和“任务级精度评估”分开记录：前者检查算子输出与梯度，后者判断误差是否影响真实模型或科学计算任务。
