# Auto-Tuning（自动调优）

**所属分类：** [06 编译器优化](README.md)  
**英文名称：** Auto-Tuning  
**专题编号：** 07

## 01. 一句话定义与核心目的

**一句话定义：** Auto-Tuning 是自动搜索 Kernel 或计算程序的配置参数与实现策略，并根据目标硬件上的测量结果选择性能较好的方案。

**核心目的：** 减少人工试错，在不同输入形状、数据类型和硬件上找到合适的 Tile、线程配置、流水阶段或算法实现。自动调优本质上是一个受约束的搜索与测量过程，不等同于编译器常规的静态优化。

## 02. 数学原理与核心技术

### 2.1 可视化理解

![Auto-Tuning 搜索流程](../images/06_Compiler_Optimization/07_Auto_Tuning.svg)

*图：调优器生成候选配置，编译并在目标硬件上测量，通过正确性和性能筛选保留较优方案。实际系统还可能使用缓存、剪枝或机器学习模型减少搜索成本。*

### 2.2 数学原理与核心技术

设配置空间为 $\mathcal C$，目标硬件为 $H$，输入工作负载为 $X$，调优目标可以写成：

$$
c^*=\arg\min_{c\in\mathcal C}T(c;H,X),
$$

其中 $T$ 是候选配置的实测执行时间。候选配置必须满足正确性、资源和精度约束：

$$
c\in\mathcal C_{valid}.
$$

因此实际问题是：

$$
c^*=\arg\min_{c\in\mathcal C_{valid}}T(c;H,X).
$$

**核心技术：**
- **Search Space：** 定义 Tile 尺寸、Warp 数、Stages、算法路径等可调参数。
- **Grid Search：** 枚举候选配置，简单但搜索成本可能很高。
- **Random / Bayesian Search：** 用随机采样或代理模型减少试验数量。
- **Cost Model：** 在真实运行前估计候选性能，用于筛选或排序。
- **Measurement：** 进行预热、重复计时、同步和统计，控制测量噪声。
- **Autotune Cache：** 缓存某种硬件与输入形状下的最佳配置，避免重复搜索。
- **Correctness Validation：** 每个候选都应先通过数值正确性和必要的精度验证。

自动调优结果通常与 GPU 型号、驱动、编译器版本、输入 Shape 和数据类型绑定。只报告某个配置的最快一次测量并不可靠，应使用稳定的统计方法并保留正确性约束。

## 03. 代表案例

| 项目 | 与本专题的关系 | 值得关注的内容 | 项目与资料 |
|---|---|---|---|
| TVM Ansor | **经典直接案例：** 基于搜索和成本模型自动优化张量程序 | Search space、program sampling、cost model | [GitHub](https://github.com/apache/tvm) · [论文](https://arxiv.org/abs/2006.06762) |
| Triton | **直接案例：** 通过 autotune 配置搜索 Kernel 参数 | Config、key、num_warps、num_stages | [GitHub](https://github.com/triton-lang/triton) · [教程](https://triton-lang.org/main/python-api/generated/triton.autotune.html) |
| OpenTuner | **直接案例：** 通用自动调优框架 | 多种搜索技术、参数配置空间 | [GitHub](https://github.com/jansel/opentuner) |
| CUTLASS Profiler | **直接工具：** 枚举并测量 GEMM Kernel 配置 | tile shape、data type、split-K、throughput | [GitHub](https://github.com/NVIDIA/cutlass) |

建议区分“编译器静态优化”和“基于目标设备实测的自动调优”：前者主要分析程序，后者需要执行候选并测量真实性能。
