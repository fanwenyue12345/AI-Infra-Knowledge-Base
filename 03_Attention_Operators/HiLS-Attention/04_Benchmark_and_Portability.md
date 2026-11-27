# 04 — HiLS 实验、Benchmark 与跨硬件适配计划

## 1. 研究目标

建立从算子正确性到端到端模型性能的分层验证体系。目标硬件尚未指定，因此先建立硬件无关的数学接口和基线。

## 2. 分阶段实施

### Phase 0：环境与基线

记录 HiLS 仓库 commit、Python/PyTorch/CUDA/Triton/TileLang 版本、GPU 型号和显存、驱动、模型配置及权重来源。运行官方最小示例，确认环境可复现。未运行结果不得记为基线。

### Phase 1：算子正确性

优先顺序：
1. Chunk Pooling；
2. Top-K / Softmax；
3. Sparse Attention；
4. RoPE；
5. 完整 HiLS Attention。

比较参考实现与目标实现的 forward output、backward gradients、最大绝对误差、相对误差、NaN/Inf、边界 shape、GQA、causal mask 和不完整 Chunk。

### Phase 2：性能基线

建议实验网格（不是官方默认值）：

| 维度 | 候选值 |
|---|---|
| Batch | 1、2、4、8 |
| Sequence Length | 2K、4K、8K、16K、32K |
| Chunk Size | 32、64、128 |
| Top-K | 4、8、16、32 |
| dtype | FP16、BF16 |
| Head 数 | 覆盖 MHA 与 GQA |

### Phase 3：性能剖析

分别测量 Q/K/V 投影、RoPE、Chunk Pooling、Top-K、Sliding Window Attention、Sparse Attention、分支融合、Cache 更新、Kernel launch 与同步。

先用 Nsight Systems 定位端到端时间线，再用 Nsight Compute 分析热点 Kernel。

### Phase 4：跨硬件适配

采用 Backend 分层：

```text
HiLS Model API
      |
Operator Contract
      |
  +---+----------------+
  |                    |
CUDA/Triton Backend   Target Backend
  |                    |
NVIDIA GPU            国产 GPU / NPU
```

统一接口明确输入输出 shape、dtype/累积精度、layout/stride、mask 语义、GQA Head 映射、workspace、stream/异步执行语义及 backward 支持。

优先适配 Chunk Pooling、Top-K、Sparse Attention，再考虑完整模型服务。

## 3. 统一算子接口建议

以 Chunk Pooling 为例：

```python
def chunk_pool(
    landmark_q,
    chunk_k,
    *,
    scale,
    layout,
    output_dtype,
):
    # Returns compressed_k and entropy_bias
    ...
```

接口定义数学语义和数据约定，不把特定 CUDA/Triton/TileLang 的 Tile、Warp 或线程数暴露给模型层。

## 4. 适配风险

- 目标编译器不支持现有 TileLang/Triton 语法或操作；
- 动态 shape 或归约模式不受支持；
- GQA 隐式广播产生额外内存开销；
- Top-K 排序/归约原语性能不足；
- 不规则 gather 造成访存效率下降；
- FP16/BF16 累积精度与 NVIDIA 路径不同；
- 自定义 backward 未移植；
- Cache、stream、同步语义与推理框架不兼容。

## 5. 评价指标

算子级：

$$
Speedup=T_{baseline}/T_{new}
$$

报告 median latency、p90/p99（适用时）、throughput、memory footprint、correctness error、有效带宽/计算利用率。

模型级：Prefill latency、Decode latency/TPOT、TTFT、tokens/s、最大上下文、峰值显存、任务质量/困惑度、不同 Batch/并发下吞吐。

算子级与模型级指标必须分开报告。

## 6. 候选研究问题

- **R1：Chunk Pooling 硬件感知分块。** 不同 GPU 的片上资源和矩阵计算单元不同，研究自动调优与性能模型。
- **R2：Top-K 检索负载均衡。** 不同 Query 选择不同 Chunk，研究分桶、排序、批量 gather 和调度。
- **R3：检索开销与稀疏收益交叉点。** 改变序列长度、Chunk Size、Top-K，建立实测性能模型。
- **R4：跨硬件统一算子抽象。** 保持数学契约一致，允许 Backend 使用不同 layout、tiling 和调度。

以上是待验证研究方向，不是已获得的实验结论。

## 7. 实验记录模板

| 字段 | 记录 |
|---|---|
| Date / Commit | |
| Hardware | |
| Driver / Compiler | |
| PyTorch / Triton / TileLang | |
| Operator | |
| Shape / Layout | |
| dtype / accumulation dtype | |
| Parameters | |
| Baseline | |
| Latency | |
| Throughput | |
| Memory | |
| Correctness Error | |
| Profile Observation | |
| Conclusion | |

## 8. 当前下一步

1. 本地固定 HiLS 仓库 commit，保存环境信息。
2. 复现 _chunk_attn_pool_impl 的 PyTorch 参考计算。
3. 对照 chunk_attn_pool_tilelang.py 检查 forward/backward。
4. 建立独立 benchmark，不先修改模型。
5. 完成 Top-K 与 Sparse Attention 的正确性测试。
6. 确认目标国产芯片和编译栈后再选定移植路径。
