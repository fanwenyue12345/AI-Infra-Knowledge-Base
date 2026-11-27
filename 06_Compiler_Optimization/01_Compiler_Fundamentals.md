# Compiler 基础

**所属分类：** [06 编译器优化](README.md)  
**英文名称：** Compiler Fundamentals  
**专题编号：** 01

## 01. 一句话定义与核心目的

**一句话定义：** 编译器是将高级语言或计算表达式转换为目标平台可执行代码的系统，并在转换过程中分析程序语义、选择实现方式和执行优化。

**核心目的：** 在保持程序语义的前提下，把易于表达的计算转化为适合 CPU、GPU 或其他加速器执行的代码。编译器连接算法表达、计算图、硬件资源和最终指令，是高性能计算从“写出计算”到“高效执行”的关键环节。

## 02. 数学原理与核心技术

### 2.1 可视化理解

![01_Compiler_Fundamentals 可视化](../images/06_Compiler_Optimization/01_Compiler_Fundamentals.svg)

*图：源程序经过前端解析形成 IR，再经过优化、目标相关 Lowering 与代码生成，最终形成目标机器代码。真实编译器可能包含多轮 IR 和多条优化管线。*

### 2.2 数学原理与核心技术

### 2.1 可视化理解



### 2.2 数学原理与核心技术

编译器优化最重要的约束是**语义保持**。设源程序为 $P$，变换后的程序为 $P'$，对所有满足前置条件的输入 $x$，要求二者的可观察行为一致：

$$
\operatorname{Obs}(P,x)=\operatorname{Obs}(P',x).
$$

这里的可观察行为可能包括输出值、内存状态、异常和 I/O 等。浮点计算还需考虑舍入误差；某些快速数学优化只在允许近似或指定精度条件下成立。

**核心阶段与技术：**
- **Front End：** 词法/语法分析、类型检查、语义分析，将源代码转换为编译器内部表示。
- **IR（Intermediate Representation）：** 用统一结构表示运算、控制流、数据类型和依赖。
- **Optimization Passes：** 常量折叠、CSE、DCE、循环变换、向量化和算子融合等。
- **Lowering：** 将高层抽象逐步转成更具体的循环、内存访问、线程布局和目标操作。
- **Backend：** 指令选择、寄存器分配、指令调度、汇编和目标文件生成。
- **Runtime：** 某些系统还需要运行时负责 Kernel 启动、内存管理、设备调度和动态编译。

编译器并不是简单地把每一行高级代码翻译成一条机器指令。它通常会改变计算组织、数据布局和执行顺序，但必须满足语义约束。

## 03. 代表案例

| 项目 | 与本专题的关系 | 值得关注的内容 | 项目与资料 |
|---|---|---|---|
| LLVM | **直接案例：** 通用编译器基础设施，包含 IR、优化 Pass 和多目标后端 | Clang、LLVM IR、Pass、CodeGen | [GitHub](https://github.com/llvm/llvm-project) · [LLVM 文档](https://llvm.org/docs/) |
| MLIR | **直接案例：** 面向多层抽象和领域专用编译的可扩展 IR 框架 | Dialect、Rewrite、Conversion、Lowering | [官网](https://mlir.llvm.org/) · [GitHub](https://github.com/llvm/llvm-project) |
| GCC | **直接案例：** 成熟的多语言、多目标编译器 | GIMPLE、RTL、优化 Pass | [GitHub](https://github.com/gcc-mirror/gcc) · [文档](https://gcc.gnu.org/onlinedocs/) |
| Triton | **关联案例：** 面向 GPU Kernel 的领域专用编程语言与编译器 | Triton IR、GPU lowering、代码生成 | [GitHub](https://github.com/triton-lang/triton) |

建议用 LLVM 理解传统编译器分层，再用 MLIR 和 Triton 理解深度学习与 GPU 计算为什么需要多层 IR。
