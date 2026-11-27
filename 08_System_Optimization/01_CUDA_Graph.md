# CUDA Graph

**所属分类：** [08 系统与分布式优化](README.md)  
**英文名称：** CUDA Graph  
**专题编号：** 01

## 01. 一句话定义与核心目的

**一句话定义：** CUDA Graph 是将一组 GPU 操作及其依赖关系捕获为可重复执行的图，并通过 Graph Launch 一次提交整组工作。

**核心目的：** 减少 CPU 多次发起 Kernel、Memcpy 和同步操作的调度开销，尤其适合执行结构相对固定、重复次数很多的训练步骤和推理迭代。它主要优化主机端提交与调度开销，不会自动减少 Kernel 本身的计算量或显存流量。

## 02. 数学原理与核心技术

### 2.1 可视化理解

![01_CUDA_Graph 可视化](../images/08_System_Optimization/01_CUDA_Graph.svg)

*图：传统方式由 CPU 逐个提交多个 GPU 操作；CUDA Graph 捕获操作及依赖后，通过一次 Graph Launch 重放整张执行图，降低重复提交开销。*

### 2.2 数学原理与核心技术

### 2.1 可视化理解



### 2.2 数学原理与核心技术

假设一次迭代包含 $n$ 个 GPU 操作，每个操作的设备执行时间为 $t_i$，主机提交开销为 $l_i$。传统逐项提交的简化时间为：

$$
T_{eager}\approx\sum_{i=1}^{n}(l_i+t_i).
$$

若图已完成捕获和实例化，重放时的主机提交开销可近似为一次图提交开销 $L_g$：

$$
T_{graph}\approx L_g+\sum_{i=1}^{n}t_i.
$$

这里忽略依赖等待、图内部调度、捕获与实例化成本；CUDA Graph 的收益来自重复执行时摊薄提交成本，而非保证设备执行时间本身下降。

**核心技术：**
- **Capture：** 捕获 Stream 上的 Kernel、Memcpy 和支持的操作及依赖。
- **Graph / Node / Edge：** Graph 表示执行图，Node 表示操作，Edge 表示先后依赖。
- **Instantiation：** 将捕获图实例化为可执行 Graph Exec。
- **Replay：** 重复启动已实例化的图，减少逐项提交开销。
- **Graph Update：** 在满足限制时更新部分节点参数，避免每次完全重新构图。
- **Memory Address Stability：** 某些图捕获场景要求指针和内存生命周期稳定，缓冲区管理必须符合图执行要求。

CUDA Graph 并非所有程序都适合。动态控制流、变化的内存地址、频繁改变的工作规模或捕获不支持的操作，可能限制使用方式。通常先确认程序是否受 CPU Launch 开销限制，再判断 Graph 是否值得采用。

## 03. 代表案例

| 项目 | 与本专题的关系 | 值得关注的内容 | 项目与资料 |
|---|---|---|---|
| CUDA Programming Guide | **权威资料：** 定义 Graph 创建、捕获、实例化和执行机制 | Graph API、stream capture、graph update | [官方文档](https://docs.nvidia.com/cuda/cuda-c-programming-guide/) |
| PyTorch CUDA Graphs | **直接案例：** 将训练或推理中的重复 GPU 工作捕获并重放 | torch.cuda.CUDAGraph、静态输入/输出缓冲区 | [文档](https://docs.pytorch.org/docs/stable/notes/cuda.html#cuda-graphs) |
| NVIDIA Transformer Engine | **关联案例：** Transformer 工作负载可结合图捕获降低重复调度开销 | CUDA Graph 与训练执行集成 | [GitHub](https://github.com/NVIDIA/TransformerEngine) |
| vLLM | **关联案例：** 推理执行中可对适合的固定形状/执行片段采用图捕获 | CUDA Graph、执行调度与形状管理 | [GitHub](https://github.com/vllm-project/vllm) |
