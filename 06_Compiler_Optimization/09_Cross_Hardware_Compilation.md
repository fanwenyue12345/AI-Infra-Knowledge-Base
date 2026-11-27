# 跨硬件编译

**所属分类：** [06 编译器优化](README.md)  
**英文名称：** Cross-Hardware Compilation  
**专题编号：** 09

## 01. 一句话定义与核心目的

**一句话定义：** 跨硬件编译是通过统一或分层的程序表示与后端接口，将同一计算逻辑映射到不同厂商、不同架构的 CPU、GPU 或专用加速器上。

**核心目的：** 减少为每种硬件单独重写算法的工作量，同时保留利用目标设备特性的能力。核心挑战是统一抽象与硬件专用优化之间的平衡，而不是简单地把同一份二进制放到所有设备运行。

## 02. 数学原理与核心技术

### 2.1 可视化理解

![跨硬件编译架构](../images/06_Compiler_Optimization/09_Cross_Hardware_Compilation.svg)

*图：共享前端和中间表示连接多个硬件后端；每个后端负责目标相关 Lowering、代码生成和运行时集成。不同设备仍可能需要专门的算子实现与调优。*

### 2.2 数学原理与核心技术

设同一计算任务为 $F$，目标设备为 $H_i$。编译器需要为每个目标生成实现：

$$
C_i=\operatorname{Compile}(F,H_i).
$$

理想情况下，各实现保持相同的数学语义：

$$
\operatorname{Semantics}(C_i)\approx F,
$$

其中近似符号用于强调不同浮点格式、指令和数值实现可能带来允许范围内的数值差异；具体正确性标准由框架和应用定义。

**核心技术：**
- **Portable IR：** 使用相对硬件无关的中间表示承载计算语义。
- **Target Backend：** 为不同 ISA、内存模型和设备执行模型实现专门代码生成。
- **Capability Detection：** 识别设备支持的数据类型、矩阵指令、内存和同步能力。
- **Kernel Library / Dispatch：** 对不同设备选择已有优化 Kernel 或回退实现。
- **Runtime Abstraction：** 统一设备内存、Kernel 启动、事件和同步接口。
- **Numerical Validation：** 跨设备验证误差、精度和边界行为。
- **Performance Portability：** 不仅要求程序能运行，还要评估不同设备上的性能是否合理。

跨硬件编译不保证同一 Kernel 在所有设备上性能相同。不同 GPU 的 Warp/Wavefront、矩阵指令、内存层级、编译器和运行时差异，往往要求后端重写布局、Tile 和调度策略。

## 03. 代表案例

| 项目 | 与本专题的关系 | 值得关注的内容 | 项目与资料 |
|---|---|---|---|
| MLIR | **基础设施案例：** 通过多层 Dialect 与 Conversion 连接多个硬件后端 | target dialect、conversion、LLVM translation | [官网](https://mlir.llvm.org/) |
| IREE | **直接案例：** 面向 CPU、GPU 和专用加速器的端到端编译与运行时 | HAL、target backend、dispatch、runtime | [GitHub](https://github.com/iree-org/iree) |
| Apache TVM | **直接案例：** 以统一张量程序表示和编译流程支持多类硬件 | TensorIR、target、runtime、BYOC | [GitHub](https://github.com/apache/tvm) |
| SYCL / oneAPI | **直接案例：** 提供面向不同加速器的异构编程模型和工具链 | SYCL、DPC++、device backend | [GitHub](https://github.com/intel/llvm) · [oneAPI](https://www.oneapi.io/) |
| Triton | **关联案例：** 以 GPU Kernel DSL 和后端适配探索跨架构执行 | backend、compiler IR、target-specific lowering | [GitHub](https://github.com/triton-lang/triton) |

研究国产 GPU 或 NPU 适配时，建议把工作拆成语言/IR、后端代码生成、运行时、算子库和数值验证几个层次，避免把“能编译”误认为“已实现性能可移植”。
