# Auto-Tuning（自动调优）

**所属分类：** [06 编译器优化](README.md)  
**英文名称：** Auto-Tuning  
**专题编号：** 07

## 01. 一句话定义与核心目的

**一句话定义：** Auto-Tuning（自动调优）是通过自动生成、搜索、评估和选择不同实现配置，为给定计算任务与目标硬件寻找高性能执行方案的方法。

**核心目的：** 同一个数学运算往往存在多种实现方式，而最佳方案会随张量形状、数据类型、硬件架构和运行环境变化。自动调优把原本依赖工程师手工试参数的过程转化为系统化搜索，在满足正确性和资源约束的前提下，自动选择更合适的 Tile 大小、线程组织、流水级数、数据布局或算法实现。

自动调优不等于简单地“多跑几次取最快”。完整系统还需要定义搜索空间、候选生成策略、正确性检查、性能测量、搜索算法，以及结果缓存和复用机制。

## 02. 数学原理与核心技术

### 2.1 可视化理解

![Auto-Tuning 自动调优搜索过程](../images/06_Compiler_Optimization/07_Auto_Tuning.svg)

*图：自动调优从参数空间生成候选配置，编译并运行候选程序，测量正确性与延迟，再利用搜索策略或成本模型选择下一批候选，最终保存最优的有效配置。*

### 2.2 数学原理与核心技术

可以把自动调优抽象为一个受约束的优化问题。设配置参数为：

$$
\theta=(B_M,B_N,B_K,W,S,L,\ldots),
$$

其中参数可以代表 Tile 尺寸、Warp 数量、流水级数、数据布局等。给定工作负载 $x$ 和硬件平台 $h$，希望找到：

$$
\theta^*=
\arg\min_{\theta\in\Theta}
T(\theta;x,h)
$$

其中：
- $\Theta$ 是候选配置构成的搜索空间。
- $T(\theta;x,h)$ 是配置 $\theta$ 在工作负载 $x$ 和硬件 $h$ 上的执行时间。
- $\theta^*$ 是测得或预测性能最好的配置。

实际搜索还必须满足正确性和资源约束，例如：

$$
\operatorname{Correct}(\theta)=\text{True},
\qquad
\operatorname{ResourceUsage}(\theta)\leq R_{\max}.
$$

这里的目标函数不一定只看延迟。训练场景可能关心吞吐量，推理场景可能同时关心延迟和吞吐，嵌入式设备还可能关心能耗与内存占用。

**核心技术：**

- **Search Space（搜索空间）：** 定义可以调整的参数及取值范围，例如 Tile 大小、num_warps、num_stages、向量化宽度和算法变体。搜索空间过小可能错过好方案，过大则增加搜索成本。
- **Candidate Generation（候选生成）：** 根据搜索规则生成待测试的配置，可能采用网格枚举、随机采样、规则约束或进化式生成。
- **Search Strategy（搜索策略）：** 决定先测试哪些候选、如何利用已有结果。常见方法包括 Grid Search、Random Search、Evolutionary Search、Bayesian Optimization 和 Successive Halving。
- **Cost Model（成本模型）：** 根据硬件特征、程序结构或历史测量预测候选性能，减少昂贵的真实编译和运行次数。成本模型可以是规则模型，也可以是机器学习模型。
- **Measurement & Validation（测量与验证）：** 对候选进行编译、正确性检查和性能测量。需要控制预热、重复次数、时钟波动、并发干扰和测试输入。
- **Pruning（剪枝）：** 提前淘汰明显不合法、资源超限或性能不可能有竞争力的候选。
- **Tuning Cache（调优缓存）：** 将最优配置与工作负载特征、硬件型号、软件版本等关联保存，避免重复搜索。

### 2.3 自动调优与人工调参的区别

人工调参通常由开发者根据经验选择少量配置，再反复测试；自动调优则把参数空间、候选生成、测试和选择过程系统化。它并不保证找到数学意义上的全局最优解：实际通常受搜索预算、成本模型误差和搜索空间限制，只能找到当前预算内表现较好的有效配置。

还要区分几个相近概念：

- **Auto-Tuning：** 自动搜索并选择性能配置，是总体方法。
- **Auto-Scheduling：** 自动决定计算如何映射到循环、Tile、线程和硬件执行结构，常作为自动调优的一部分。
- **Compiler Optimization：** 编译器可以提供候选变换或代码生成能力；自动调优负责探索不同选择并评估结果。
- **Runtime Adaptation：** 运行时根据负载变化动态选择策略，可能使用调优结果，但不等同于离线 Auto-Tuning。

## 03. 代表案例

| 项目 | 与本专题的关系 | 值得关注的内容 | 项目与资料 |
|---|---|---|---|
| Apache TVM / MetaSchedule | **直接案例：** 面向张量程序的自动调度与搜索系统 | Search Space、Schedule Rule、Cost Model、Tuning Database、数据库复用 | [GitHub](https://github.com/apache/tvm) · [MetaSchedule 文档](https://tvm.apache.org/docs/arch/meta_schedule.html) |
| Triton | **直接案例：** 通过 `@triton.autotune` 对多个 Kernel 配置进行基准测试并选择配置 | `triton.Config`、`num_warps`、`num_stages`、key 参数和调优缓存 | [GitHub](https://github.com/triton-lang/triton) · [官方 Autotune 文档](https://triton-lang.org/main/python-api/generated/triton.autotune.html) |
| OpenXLA XLA | **直接案例：** 对 GPU 算子实现（例如 GEMM）进行候选算法选择与自动调优 | HLO 中的自定义调用、GEMM 算法配置、调优结果缓存 | [GitHub](https://github.com/openxla/xla) · [XLA GPU 文档](https://openxla.org/xla) |
| NVIDIA CUTLASS | **关联案例：** 提供大量 GEMM Kernel 配置与 Profiler 工具，可枚举和测量不同实现 | Tile Shape、Cluster Shape、Warp 数量、Stages、Profiler | [GitHub](https://github.com/NVIDIA/cutlass) · [Profiler 文档](https://docs.nvidia.com/cutlass/latest/media/docs/cpp/profiler.html) |
| Ansor（TVM） | **经典研究案例：** 通过进化搜索与学习型成本模型探索张量程序调度 | Sketch Generation、Evolutionary Search、Learned Cost Model | [论文](https://arxiv.org/abs/2006.06762) · [TVM GitHub](https://github.com/apache/tvm) |

**建议的学习顺序：** 先看 Triton 的 `@triton.autotune`，理解“给定若干配置—逐一测量—选择最快配置”；再看 TVM MetaSchedule，理解搜索空间、成本模型和调优数据库；最后看 CUTLASS Profiler，理解真实 GEMM Kernel 配置空间有多大，以及硬件参数如何影响性能。

**进一步阅读：**
- [TVM MetaSchedule：架构与工作流程](https://tvm.apache.org/docs/arch/meta_schedule.html)
- [Triton：Auto-Tuning](https://triton-lang.org/main/python-api/generated/triton.autotune.html)
- [Ansor 论文：Generating High-Performance Tensor Programs for Deep Learning](https://arxiv.org/abs/2006.06762)
- [NVIDIA CUTLASS：GEMM Profiler](https://docs.nvidia.com/cutlass/latest/media/docs/cpp/profiler.html)
