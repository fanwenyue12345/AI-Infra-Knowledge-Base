# 图编译

**所属分类：** [06 编译器优化](README.md)  
**英文名称：** Graph Compilation  
**专题编号：** 04

## 01. 一句话定义与核心目的

**一句话定义：** 图编译是将模型或程序的计算图捕获为编译器可分析的图表示，再通过图级优化、分区和代码生成形成可执行程序的过程。

**核心目的：** 在单个算子之外观察更大的计算上下文，识别跨算子融合、常量传播、冗余计算和设备分区机会，并减少动态图运行时的调度开销。

## 02. 数学原理与核心技术

### 2.1 可视化理解

![图编译过程](../images/06_Compiler_Optimization/04_Graph_Compilation.svg)

*图：框架中的算子调用被捕获成计算图，编译器在图上进行变换，再将子图编译为目标设备可执行代码。*

### 2.2 数学原理与核心技术

设计算图为：

$$
G=(V,E),
$$

其中 $V$ 表示算子节点，$E$ 表示数据依赖。图编译通过语义保持的变换 $T$ 生成新图：

$$
G'=T(G),
$$

目标是在保持程序行为的前提下，降低执行成本：

$$
C(G')<C(G),
$$

这里 $C$ 是编译器使用的成本估计或目标函数，实际性能仍需运行测量验证。

**核心技术：**
- **Graph Capture：** 捕获模型执行中的算子与依赖关系。
- **Graph Canonicalization：** 规范化图结构，便于后续模式匹配。
- **Graph Rewrite：** 对子图进行融合、常量折叠、CSE 等变换。
- **Partitioning：** 将图分区到不同设备或执行后端。
- **Shape / Type Inference：** 推断张量形状与数据类型，辅助合法性检查和代码生成。
- **AOT / JIT Compilation：** Ahead-of-Time 在运行前编译；Just-in-Time 在运行时根据实际信息编译。

动态图中存在 Python 控制流、动态形状和数据依赖时，图捕获可能需要限制、专门处理或回退到解释执行。

## 03. 代表案例

| 项目 | 与本专题的关系 | 值得关注的内容 | 项目与资料 |
|---|---|---|---|
| torch.compile / TorchDynamo | **直接案例：** 捕获 PyTorch 程序并交给后端编译优化 | Dynamo graph capture、guards、graph breaks | [GitHub](https://github.com/pytorch/pytorch) · [文档](https://docs.pytorch.org/docs/stable/torch.compiler.html) |
| OpenXLA | **直接案例：** 将模型计算转成 HLO 并执行图级优化 | HLO、StableHLO、XLA compilation | [GitHub](https://github.com/openxla/xla) |
| TensorFlow XLA | **直接案例：** 将 TensorFlow 子图编译为优化后的设备程序 | jit_compile、cluster、XLA HLO | [官方文档](https://www.tensorflow.org/xla) |
| ONNX Runtime | **关联案例：** 通过图优化和 Execution Provider 将模型图映射到后端 | graph optimization、partitioning、EP | [GitHub](https://github.com/microsoft/onnxruntime) |
