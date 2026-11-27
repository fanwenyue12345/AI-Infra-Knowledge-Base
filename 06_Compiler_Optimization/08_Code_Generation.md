# 代码生成

**所属分类：** [06 编译器优化](README.md)  
**英文名称：** Code Generation  
**专题编号：** 08

## 01. 一句话定义与核心目的

**一句话定义：** 代码生成是编译器把优化后的中间表示转换为目标平台可执行代码的阶段，包括指令选择、寄存器分配、指令调度和目标文件生成。

**核心目的：** 将抽象计算落实为目标 ISA 和硬件能够执行的指令，同时满足调用约定、寄存器、内存和设备执行模型等约束。

## 02. 数学原理与核心技术

### 2.1 可视化理解

![08_Code_Generation 可视化](../images/06_Compiler_Optimization/08_Code_Generation.svg)

*图：低层 IR 经指令选择、寄存器分配和指令调度，生成目标 ISA；CUDA 路径中 PTX 与 SASS 需要区分。*

### 2.2 数学原理与核心技术

### 2.1 可视化理解

![代码生成过程](../images/06_Compiler_Optimization/08_Code_Generation.svg)

*图：低层 IR 经过指令选择、寄存器分配和指令调度，生成目标汇编或机器码。GPU 编译链还可能经过 PTX、目标机器码等设备相关阶段。*

### 2.2 数学原理与核心技术

代码生成需要将 IR 操作映射到目标指令集合。对于表达式：

$$
c=a\times b+d,
$$

目标架构若支持 FMA，编译器可能将乘法和加法映射为融合乘加指令：

$$
c=\operatorname{fma}(a,b,d).
$$

该变换是否允许还取决于浮点语义和编译选项。

**核心技术：**
- **Instruction Selection：** 将 IR 操作匹配到目标指令。
- **Register Allocation：** 将活跃变量分配到有限寄存器，必要时 Spill。
- **Instruction Scheduling：** 在依赖约束下重排指令，改善流水执行和延迟隐藏。
- **Peephole Optimization：** 在局部指令窗口中识别并替换低效模式。
- **Assembly / Machine Code Emission：** 输出汇编、目标文件或设备机器码。
- **Linking：** 将目标文件、库和运行时依赖组合成可执行产物。

GPU 开发中需要区分 PTX 与 SASS：PTX 是 NVIDIA 的虚拟 ISA/中间汇编形式，最终还需由后端生成具体 GPU 架构的机器指令；不同架构的指令和资源特征可能不同。

## 03. 代表案例

| 项目 | 与本专题的关系 | 值得关注的内容 | 项目与资料 |
|---|---|---|---|
| LLVM CodeGen | **直接案例：** 从 LLVM IR 生成目标机器指令 | SelectionDAG、GlobalISel、register allocation | [GitHub](https://github.com/llvm/llvm-project) · [文档](https://llvm.org/docs/CodeGenerator.html) |
| NVIDIA PTX ISA | **权威资料：** 描述 GPU 虚拟 ISA 与指令语义 | PTX instructions、memory model、MMA | [官方文档](https://docs.nvidia.com/cuda/parallel-thread-execution/) |
| CUDA Compiler Toolchain | **直接案例：** 将 CUDA C++ 编译为 PTX 并生成设备机器码 | nvcc、ptxas、fatbinary、SASS | [CUDA 文档](https://docs.nvidia.com/cuda/cuda-compiler-driver-nvcc/) |
| MLIR LLVM Dialect | **关联案例：** 将较高层 IR 转换为 LLVM IR，接入 LLVM 后端 | LLVM IR translation、conversion passes | [MLIR 文档](https://mlir.llvm.org/) |
