# 01 — HiLS-Attention 算法与数学原理

## 1. 问题背景

标准 scaled dot-product attention：

$$
Q=XW_Q,\quad K=XW_K,\quad V=XW_V
$$

$$
A=\operatorname{Softmax}(QK^T/\sqrt d),\qquad O=AV
$$

序列长度为 L 时，稠密注意力分数矩阵规模为 L×L。FlashAttention 主要通过 IO-aware 分块减少 HBM 访问，并不消除精确稠密 Attention 的二次算术规模。

Chunk-based Sparse Attention 将历史序列切块，再选择少量相关 Chunk。

## 2. Chunk 的精确注意力质量

设 Chunk c 包含 Key：

$$
\mathcal K_c=\{k_1,\ldots,k_S\}
$$

对 Query q，该 Chunk 的未归一化注意力质量：

$$
Z_c(q)=\sum_{j=1}^{S}\exp(q^Tk_j/\sqrt d)
$$

其对数形式：

$$
g_c(q)=\log Z_c(q)
$$

若每个 Query 都与所有 Chunk 内 Token 计算分数，可能抵消稀疏检索的收益。因此需要低成本的 Chunk 级近似表示。

## 3. Landmark Query、压缩 Key 与熵偏置

给定先验/地标 Query μc，定义 Chunk 内分布：

$$
p_j=\operatorname{Softmax}_j(\mu_c^Tk_j/\sqrt d)
$$

压缩 Key：

$$
k'_c=\sum_{j=1}^{S}p_jk_j
$$

熵偏置：

$$
b'_c=-\sum_{j=1}^{S}p_j\log p_j
$$

其中 p_j 是 Landmark Query 对 Chunk 内 Key 的注意力分布。k' 是加权 Key 汇总，b' 描述分布熵。

## 4. 一阶 Taylor 近似

对：

$$
g_c(q)=\log\sum_j\exp(q^Tk_j/\sqrt d)
$$

在 q=μc 附近展开：

$$
g_c(q)\approx g_c(\mu_c)+\nabla g_c(\mu_c)^T(q-\mu_c)
$$

梯度为：

$$
\nabla g_c(\mu_c)=\frac{1}{\sqrt d}\sum_jp_jk_j
=\frac{k'_c}{\sqrt d}
$$

因此：

$$
g_c(q)\approx q^Tk'_c/\sqrt d+b'_c
$$

因为：

$$
g_c(\mu_c)-\mu_c^Tk'_c/\sqrt d
=-\sum_jp_j\log p_j=b'_c
$$

这使 Chunk 的近似 log-mass 可以通过一次 Query 与压缩 Key 的内积加偏置得到。

**适用边界：** 这是在 Landmark Query 附近的一阶局部近似。Query 偏离展开点时误差可能增大。应通过检索召回、排序质量和最终任务质量验证。

## 5. 分层注意力结构

计算拆成两个层次：

1. Chunk 内部：在选中的 Chunk 内对原始 Token 做细粒度注意力。
2. Chunk 之间：为选中的 Chunk 分配权重，并与局部窗口分支融合。

需要区分：
- 压缩 Key 用于 Chunk 检索/估计 Chunk 质量。
- 被选中 Chunk 的原始 K/V 用于细粒度 Attention 输出。
- Sliding Window Attention 提供近邻上下文通路。

## 6. 源码实现细节

在 models/FlashHiLS/hils_attention.py 的 _chunk_attn_pool_impl 中：

- Landmark Query 和分块 Key 转为 FP32 进行池化计算。
- GQA 的 KV Head 会扩展以匹配 Query Head。
- 计算 Chunk 内 logits 并乘以 1/sqrt(d) 缩放。
- 代码将每个 Chunk 的最后一个 Token 位置 logits mask 为 -inf。
- 沿 Chunk 内 Token 维做 Softmax。
- 计算加权 Key 与熵偏置。
- 压缩 Key 转回原 Key dtype，bias 保留 FP32。

调用方通常从每个 Chunk 的最后 Token 位置取 Landmark Query；池化实现同时屏蔽 Chunk 内最后一个 Key 位置，避免先验 Query 与同位置 Key 直接自匹配。应结合具体实验配置核对。

源码中 enable_chunk_pooling=True 时还存在均值池化替代路径：

$$
k'_c=\frac1S\sum_jk_j
$$

该选项适合用作消融对照，不应与注意力池化混为一谈。

## 7. 复杂度讨论

设总长度 L，Chunk Size S，Chunk 数 Nc=L/S，每个 Query 选择 K 个 Chunk。粗略而言：

- Chunk 表示构造涉及 Chunk 内池化；
- Chunk 检索需要每个 Query 与 Nc 个压缩表示比较；
- 细粒度稀疏 Attention 访问选中的约 KS 个 Token（另有局部窗口）；
- Top-K、索引 gather、缓存管理和不规则访存也有成本。

不能只用单一复杂度项描述全部开销。应分别分析表示构造、检索、稀疏 Attention、局部 Attention、Cache 和 Kernel Launch。

## 8. 与相关方法的区别

- Full Attention：所有 Query 与所有 Key 计算注意力。
- FlashAttention：保持精确稠密 Attention 数学结果，优化 IO 和执行组织。
- Sliding Window：限制每个 Query 关注的局部窗口。
- Block Sparse Attention：以块为单位选择稀疏连接。
- HiLS：使用 Chunk 级表示检索，再对选中 Chunk 做细粒度计算，并与局部窗口分支融合。

## 9. 待核对问题

1. 论文对 Hierarchical Softmax 的完整定义与实现细节。
2. Top-K kernel 中 scores 与 lse_sum 的具体数值约定。
3. enable_softmax1 两种归一化路径的数学差异。
4. Landmark Query 的不同配置与训练效果。
5. Chunk 最后位置屏蔽对因果性和检索质量的作用。
6. 训练、Prefill、Decode 下 Chunk 表示和缓存更新的差异。

## 10. 参考资料

- [HiLS-Attention 官方仓库](https://github.com/Tencent-Hunyuan/HiLS-Attention)
- [HiLS-Attention 论文](https://arxiv.org/abs/2607.02980)
