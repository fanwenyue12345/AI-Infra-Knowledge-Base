# 张量并行

**所属分类：** [04 并行优化](README.md)  
**英文名称：** Tensor Parallelism  
**专题编号：** 04

## 01. 一句话定义与核心目的

**一句话定义：** 张量并行是把单个模型层中的权重矩阵、激活张量或运算维度切分到多个设备上，让多个设备协同完成同一层的计算。

**核心目的：** 当单层矩阵或模型参数无法高效放入单张 GPU，或希望利用多 GPU 的聚合计算能力时，将层内计算分散到多设备。代价是设备之间需要执行 AllReduce、AllGather 或 ReduceScatter 等通信。

## 02. 数学原理与核心技术

### 2.1 可视化理解

![04_Tensor_Parallelism 可视化](../images/04_Parallelism_Optimization/04_Tensor_Parallelism.svg)

*图：列切分让设备产生不同输出分片；行切分产生局部部分和，需要归约组合。*

### 2.2 数学原理与核心技术

### 2.1 可视化理解

![张量并行矩阵切分](../images/04_Parallelism_Optimization/04_Tensor_Parallelism.svg)

*图：矩阵乘法按输出列切分时，各设备计算局部输出列；按输入维度切分时，各设备计算局部部分和，通常需要归约通信。*

### 2.2 数学原理与核心技术

考虑线性层：

$$
Y=XW,
$$

其中 $X\in\mathbb R^{B\times H}$，$W\in\mathbb R^{H\times O}$。

**Column Parallel（按输出维切分）：**

将权重按列切分：

$$
W=[W_1,W_2,\ldots,W_p].
$$

各设备独立计算：

$$
Y_r=XW_r,
$$

最后按列拼接得到 $Y$，通常对应 AllGather，或让后续消费者直接使用分片输出。

**Row Parallel（按输入维切分）：**

将权重按行切分，并将输入按对应维度分片：

$$
Y=\sum_{r=1}^{p}X_rW_r.
$$

各设备得到局部部分和，需要通过 AllReduce 或 ReduceScatter 等操作组合结果。

**核心技术：**
- **Column Parallel Linear：** 沿输出特征维度切分权重，产生分片输出。
- **Row Parallel Linear：** 沿输入特征维度切分，局部结果需要归约。
- **Attention Head Parallelism：** 将不同 Attention Heads 分配给不同设备。
- **Collective Communication：** 使用 AllReduce、AllGather、ReduceScatter 交换或组合张量。
- **Sequence / Context Parallel：** 在序列维度切分，属于相关但目标不同的并行策略。

张量并行通信频繁，通常更依赖高速互联；切分粒度、层结构、Batch/Sequence 长度和通信拓扑都会影响扩展效率。

## 03. 代表案例

| 项目 | 与本专题的关系 | 值得关注的内容 | 项目与资料 |
|---|---|---|---|
| Megatron-LM | **经典直接案例：** Transformer 层内张量模型并行 | Column/Row Parallel Linear、Attention 与 MLP 切分 | [GitHub](https://github.com/NVIDIA/Megatron-LM) · [论文](https://arxiv.org/abs/1909.08053) |
| DeepSpeed | **直接案例：** 提供 Tensor Parallel 等大模型分布式训练能力 | TP、通信与模型切分 | [GitHub](https://github.com/deepspeedai/DeepSpeed) |
| vLLM | **关联案例：** 推理时支持张量并行部署大型模型 | TP worker、权重切分、分布式执行 | [GitHub](https://github.com/vllm-project/vllm) |
| Colossal-AI | **直接案例：** 提供多维并行训练策略 | 1D/2D/2.5D/3D Tensor Parallelism | [GitHub](https://github.com/hpcaitech/ColossalAI) |
