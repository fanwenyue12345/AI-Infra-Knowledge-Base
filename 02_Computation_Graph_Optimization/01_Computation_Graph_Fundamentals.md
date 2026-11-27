# 计算图基础

**所属分类：** [02 计算图优化](README.md)  
**英文名称：** Computation Graph Fundamentals  
**专题编号：** 01

## 01. 一句话定义与核心目的

**一句话定义：** 计算图是用节点表示变量或运算、用有向边表示数据依赖关系的图结构，用来描述一个程序或模型如何从输入计算出输出。

**核心目的：** 将复杂计算显式表示为依赖关系，使框架或编译器能够分析哪些操作必须先执行、哪些操作可以并行、哪些结果可以复用，以及哪些节点可以融合、删除或重新安排。深度学习中的计算图也是自动微分构建反向传播关系的重要基础。

## 02. 数学原理与核心技术

### 2.1 可视化理解

![OpenXLA HLO graph visualization](https://openxla.org/static/xprof/images/trace_viewer_to_hlo_graph_2.png)

*图源：OpenXLA XProf 官方文档。展示实际 HLO 图中的算子、张量边和融合区域。参见[官方 Trace Viewer 文档](https://openxla.org/xprof/trace_viewer)。*

*图：节点表示输入或运算，有向边表示数据依赖。a=x+y 必须先于 z=a×w 执行；互不依赖的分支可以独立调度。*

### 2.2 数学原理与核心技术

计算图通常可表示为有向图：

$$
G=(V,E)
$$

其中 $V$ 是节点集合，$E$ 是有向边集合。边 $u\to v$ 表示节点 $v$ 的计算依赖节点 $u$ 的结果。对于无循环依赖的计算图，它通常是有向无环图（DAG）。

例如：

$$
z=(x+y)\times w
$$

可以拆成：

$$
a=x+y,\qquad z=a\times w.
$$

节点 $a$ 必须先于 $z$ 执行；若另有独立分支 $b=f(p)$，只要没有数据依赖，$b$ 就可以与 $a$ 并行计算。

**核心技术与概念：**

- **Node（节点）：** 输入、常量、张量或运算。
- **Edge（边）：** 表示数据依赖和执行先后约束。
- **DAG（有向无环图）：** 表示无循环依赖的计算关系；循环程序可通过展开、循环节点或其他表示处理。
- **Topological Order（拓扑序）：** 满足所有依赖边方向的合法执行顺序。
- **Graph Capture（计算图捕获）：** 从程序运行或模型定义中提取图表示。
- **Graph Rewrite（图重写）：** 在保持语义的前提下替换子图，例如融合、公共子表达式消除或死代码删除。
- **Forward / Backward Graph：** 分别描述前向计算和梯度计算关系。

计算图不仅描述“算什么”，还描述“依赖什么”。优化器可以据此识别独立分支、重复表达式、无用节点，以及适合融合的连续操作。图变换必须保持可观察语义；随机数、状态更新、原地修改、异常和 I/O 等副作用会限制重排或删除。

## 03. 代表案例

| 项目 | 与本专题的关系 | 值得关注的内容 | 项目与资料 |
|---|---|---|---|
| PyTorch Autograd | **直接案例：** 动态记录张量操作并构建反向求导图 | Tensor 的 `grad_fn`、Node、`next_functions` 与反向遍历 | [GitHub](https://github.com/pytorch/pytorch) · [Autograd mechanics](https://docs.pytorch.org/docs/stable/notes/autograd.html) |
| PyTorch FX | **直接案例：** 将 Python 模块捕获为可检查、可变换的 Graph | Symbolic Tracing、Graph、Node 与 GraphModule | [GitHub](https://github.com/pytorch/pytorch) · [FX 文档](https://docs.pytorch.org/docs/stable/fx.html) |
| OpenXLA HLO | **直接案例：** 使用高层操作表示模型计算并执行图级优化 | HLO 操作、数据依赖、优化 Pass | [GitHub](https://github.com/openxla/xla) · [HLO Passes](https://openxla.org/xla/hlo_passes) |
| ONNX | **关联案例：** 用标准化计算图表示模型，支持跨框架交换和图优化 | Operator、Tensor、Graph 与模型 IR | [GitHub](https://github.com/onnx/onnx) · [文档](https://onnx.ai/onnx/intro/) |

建议先通过 PyTorch FX 或 Autograd 查看一个简单模型的图，再理解编译器如何在图上执行融合、复用和删除等变换。
