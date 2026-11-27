# 编译器算子融合

**所属分类：** [06 编译器优化](README.md)  
**英文名称：** Compiler Operator Fusion  
**专题编号：** 05

## 01. 一句话定义与核心目的

**一句话定义：** 编译器算子融合是在计算图或 IR 中识别满足条件的连续操作，并将其组合成更少的执行单元或 Kernel 的编译期变换。

**核心目的：** 减少 Kernel Launch、中间张量物化和重复内存访问，同时利用跨算子上下文进行优化。它与“手写一个融合 Kernel”不同，重点是编译器如何自动识别融合区域并生成实现。

## 02. 数学原理与核心技术

### 2.1 可视化理解

![05_Compiler_Operator_Fusion 可视化](../images/06_Compiler_Optimization/05_Compiler_Operator_Fusion.svg)

*图：编译器根据依赖、合法性和成本模型识别融合区域，将多个操作合并为更少的执行单元；过度融合可能增加资源压力。*

### 2.2 数学原理与核心技术

### 2.1 可视化理解



### 2.2 数学原理与核心技术

考虑：

$$
y=\operatorname{ReLU}(x+b).
$$

未融合时可能分别执行加法与激活：

$$
t=x+b,\qquad y=\max(t,0).
$$

融合后可在一个 Kernel 内逐元素计算：

$$
y_i=\max(x_i+b_i,0).
$$

**核心技术：**
- **Producer–Consumer Fusion：** 将产生中间值的算子与消费该值的算子合并。
- **Elementwise Fusion：** 融合逐元素运算、偏置和激活。
- **Reduction Fusion：** 在满足归约语义和资源条件时融合归约及后续操作。
- **Fusion Pattern / Region：** 通过模式匹配或图分区识别可融合子图。
- **Cost Model：** 估算启动开销、内存流量、寄存器压力和并行度，决定是否融合。
- **Code Generation：** 为融合区域生成统一 Kernel 或目标执行单元。

融合可能增大寄存器压力、降低 Occupancy、限制并行度或造成代码膨胀，因此并非融合越多越快。编译器需要在减少数据搬运与维持高效执行之间权衡。

## 03. 代表案例

| 项目 | 与本专题的关系 | 值得关注的内容 | 项目与资料 |
|---|---|---|---|
| TorchInductor | **直接案例：** 从 FX 图生成融合 Kernel | fusion scheduler、Triton codegen | [GitHub](https://github.com/pytorch/pytorch) · [Compiler 文档](https://docs.pytorch.org/docs/stable/torch.compiler.html) |
| OpenXLA XLA | **直接案例：** HLO fusion 与后端融合策略 | HLO Fusion、fusion emitter | [GitHub](https://github.com/openxla/xla) |
| TVM | **直接案例：** 通过图级和张量程序调度实现融合 | FuseOps、TensorIR schedule | [GitHub](https://github.com/apache/tvm) |
| TensorRT | **直接案例：** 推理引擎中的层融合与 Kernel 选择 | layer fusion、tactic selection | [GitHub](https://github.com/NVIDIA/TensorRT) |

这一专题研究“编译器怎样决定融合什么”；第二大类的算子融合则先建立融合本身的数学与性能动机。
