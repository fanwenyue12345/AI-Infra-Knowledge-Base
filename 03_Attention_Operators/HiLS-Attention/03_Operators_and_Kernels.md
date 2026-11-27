# 03 — HiLS 算子与 GPU Kernel 研究

## 1. 从模型函数到 GPU Kernel

模型层组织数据流，算子层执行张量运算，Kernel 层决定设备上的线程、Tile、访存和调度。

Chunk Pooling 的数学链路：

$$
(Q_{lmk},K_c)\rightarrow QK_c^T
\rightarrow Softmax\rightarrow PK_c\rightarrow(K'_c,b'_c)
$$

PyTorch 参考实现适合验证数学正确性；融合 Kernel 目标是减少中间张量写回、Kernel Launch 和重复访存。

## 2. 核心算子清单

| 源码入口 | 任务 | 研究重点 |
|---|---|---|
| ops/chunk_attn_pool_tilelang.py | Chunk 内注意力池化、压缩 Key 和熵偏置 | GEMM、Softmax、Entropy、数据复用、反向 |
| ops/topk_head_softmax.py | Chunk 检索及在线 Top-K/Softmax | 排序、归约、分数稳定性、索引布局 |
| ops/hils_fwd_bwd_head.py | 选中 Chunk 的稀疏 Attention 前向/反向 | 不规则访存、块调度、梯度 |
| ops/flex_attn_tilelang.py | 局部窗口 Attention | causal/window mask、在线 Softmax |
| ops/rope_tilelang_fp32.py | RoPE 相关计算 | 位置索引、精度、融合机会 |

具体函数名、Kernel 分派和支持形状需阅读相应文件核实。

## 3. Chunk Pooling 的参考计算

设 mu_q:[B,Nc,Hq,d]，k_chunked:[B,Nc,S,Hkv,d]。GQA 下需要按组对齐 Head。简化参考：

```python
logits = einsum(mu_q, k_chunked) / sqrt(d)
p = softmax(logits, dim=chunk_token_dim)
k_landmark = sum(p * k_chunked, dim=chunk_token_dim)
entropy = -sum(p * log(p), dim=chunk_token_dim)
```

源码中的 _chunk_attn_pool_impl 还包含 FP32 计算、GQA Head 扩展、最后位置 mask 和 dtype cast。完整行为以源码为准。

## 4. Kernel Fusion 的收益与代价

未融合的高层实现可能依次执行：
1. GEMM：计算 logits；
2. Softmax；
3. GEMM：计算加权 Key；
4. Entropy 归约。

这些阶段之间可能产生中间张量写回与重新读取。融合 Kernel 可让中间数据尽可能留在寄存器或 Shared Memory 中，减少 HBM 往返和 Launch 开销。

但融合并非越多越好。需要评估 Register pressure、Shared Memory、Occupancy、Warp/CTA 数量、数据复用、编译时间和不同形状下的性能稳定性。

## 5. GPU Kernel 核心概念

| 概念 | 在 HiLS 中可能涉及的任务 |
|---|---|
| Tile / Tiling | Chunk 内 QK 与加权汇总分块 |
| CTA / Thread Block | 分配 Chunk、Head 或 Query 子任务 |
| Warp | 协同完成矩阵片段、归约与搬运 |
| Register | 保存局部 logits、累积量和中间结果 |
| Shared Memory | 缓存重复使用的 K/V 或中间 Tile |
| Global Memory | 存放输入、KV Cache、索引和输出 |
| Coalescing | 优化连续 Token/Head 的内存访问 |
| Reduction | Softmax max/sum、Entropy 求和 |
| Pipeline | 重叠加载与计算 |
| Occupancy | 资源约束下可同时驻留的 Warp/CTA 数 |

## 6. 推荐源码阅读顺序

A. 在 hils_attention.py 检查 _chunk_attn_pool_impl：einsum 维度、GQA 扩展、FP32、mask、输出 dtype。

B. 阅读 chunk_attn_pool_tilelang.py：输入输出布局、Tile 维度、GEMM1/Softmax/Entropy/GEMM2、Shared Memory、Warp 布局、边界处理和 backward。

C. 阅读 topk_head_softmax.py：分数来源、lse_sum 和 bias、Top-K 更新、indices/scores shape、causal/window 约束、GQA。

D. 阅读 hils_fwd_bwd_head.py：indices 如何定位 Chunk、非连续访存如何处理、前向注意力和 backward 如何实现。

## 7. 性能分析框架

每个 Kernel 至少记录：
- 输入 shape：B、L、N_chunk、S、H、d；
- dtype：FP16、BF16、FP32；
- GPU 型号与架构；
- 编译器及版本；
- warmup 次数、重复次数；
- latency（中位数及分位数）；
- throughput / 有效带宽；
- 显存峰值；
- 数值误差；
- 与参考实现的正确性比较。

Nsight Systems 用于查看完整时间线和 Launch；Nsight Compute 用于分析单 Kernel 的带宽、计算利用率、Occupancy、寄存器和访存行为。

## 8. 常见误区

- FLOPs 少，不代表 Kernel 一定快。
- Kernel 融合，不代表一定减少总延迟。
- Occupancy 高，不代表性能一定最好。
- Top-K 只选少量块，不代表检索本身没有成本。
- 单 Kernel 加速，不代表端到端 Prefill/Decode 加速。
- 数值接近，不代表训练梯度一定正确。

## 9. 工具

- [TileLang](https://github.com/tile-ai/tilelang)
- [Triton](https://github.com/triton-lang/triton)
- [CUDA C++ Programming Guide](https://docs.nvidia.com/cuda/cuda-c-programming-guide/)
- [Nsight Compute](https://developer.nvidia.com/nsight-compute)
- [Nsight Systems](https://developer.nvidia.com/nsight-systems)
