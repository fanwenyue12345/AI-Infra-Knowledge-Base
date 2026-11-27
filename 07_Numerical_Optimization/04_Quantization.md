# 量化

**所属分类：** [07 数值精度优化](README.md)  
**英文名称：** Quantization  
**专题编号：** 04

## 01. 一句话定义与核心目的

**一句话定义：** 量化是把高精度数值映射到较低位宽的离散数值集合，以减少模型权重和激活的存储成本，并在支持的硬件上降低计算成本。

**核心目的：** 以可控精度损失换取更小模型、更低内存带宽需求和更高推理吞吐。量化不仅是把 FP32 转成 INT8，还需要确定缩放因子、零点、量化粒度和校准策略。

## 02. 数学原理与核心技术

### 2.1 可视化理解

![量化映射示意](../images/07_Numerical_Optimization/04_Quantization.svg)

*图：连续浮点数值经过缩放、舍入和裁剪映射到有限整数格点；反量化后只能近似恢复原数值。*

### 2.2 数学原理与核心技术

一种常见的仿射量化表达为：

$$
q=\operatorname{clip}
\left(
\operatorname{round}\left(\frac{x}{s}\right)+z,
q_{min},q_{max}
\right),
$$

反量化为：

$$
\hat{x}=s(q-z),
$$

其中 $s$ 是 scale，$z$ 是 zero-point，$q$ 是整数表示。对称量化常取 $z=0$；具体取值与量化范围和数据分布有关。

**核心技术：**
- **PTQ（Post-Training Quantization）：** 模型训练完成后直接量化，通常需要校准数据估计范围。
- **QAT（Quantization-Aware Training）：** 训练时模拟量化误差，使模型适应低精度表示。
- **Symmetric / Asymmetric Quantization：** 分别采用零点为零或允许非零零点的映射。
- **Per-tensor / Per-channel Quantization：** 一个张量共用 scale，或不同通道使用独立 scale。
- **Activation / Weight Quantization：** 分别量化激活和权重；两者的数据分布和动态范围可能不同。
- **Calibration：** 利用代表性数据确定量化范围和 scale。
- **Clipping：** 将超出量化范围的值截断，避免少量极端值占据全部量化动态范围。

量化误差取决于数值分布、位宽、量化粒度和模型敏感度。权重和激活的误差也可能通过网络层级传播，因此必须验证最终任务表现。

## 03. 代表案例

| 项目 | 与本专题的关系 | 值得关注的内容 | 项目与资料 |
|---|---|---|---|
| NVIDIA TensorRT | **直接案例：** 推理引擎支持多种低精度与量化工作流 | INT8 calibration、FP8、precision constraints | [GitHub](https://github.com/NVIDIA/TensorRT) · [文档](https://docs.nvidia.com/deeplearning/tensorrt/) |
| llama.cpp | **直接案例：** 支持多种 GGUF 权重量化格式，面向本地 LLM 推理 | Q4/Q5/Q8、量化权重布局 | [GitHub](https://github.com/ggml-org/llama.cpp) |
| bitsandbytes | **直接案例：** 提供 8-bit 与 4-bit 优化器和模型量化组件 | LLM.int8、NF4、低精度优化器 | [GitHub](https://github.com/bitsandbytes-foundation/bitsandbytes) |
| PyTorch AO | **直接案例：** 提供模型量化和低精度优化工具 | PTQ、QAT、quantized operators | [GitHub](https://github.com/pytorch/ao) |
