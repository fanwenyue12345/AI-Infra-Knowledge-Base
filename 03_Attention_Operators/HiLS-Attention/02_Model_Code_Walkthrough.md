# 02 — HiLS 模型源码导读：hils_attention.py

源码：https://github.com/Tencent-Hunyuan/HiLS-Attention/blob/main/models/FlashHiLS/hils_attention.py

本文按源码结构解释模型层职责。张量形状以符号表示，具体值由配置决定。

## 1. 模块定位

HiLSAttention(nn.Module) 是 Transformer 中的 Attention 子模块，不是完整语言模型。输入输出通常为：

$$
X:[B,L,D],\qquad Y:[B,L,D]
$$

模块内部投影 Q/K/V，生成 Landmark Query，执行局部窗口 Attention、Chunk 检索和稀疏 Attention，再合并输出并执行输出投影。

## 2. 初始化：__init__

主要定义：
- 模型维度 d_model、head_dim；
- Query/KV Head 数 h_q、h_kv；
- q_proj、k_proj、v_proj、o_proj；
- 可选 Landmark Query 投影与 norm；
- Q/K norm、RoPE 开关；
- Chunk Size、Top-K、Sliding Window；
- 底层函数 chunk_attn_pool_tilelang、online_softmax_topk_head、HiLS_block_M_head、flex_attn_tl。

GQA group size：

$$
G=H_q/H_{kv}
$$

要求 Hq 能被 Hkv 整除。GQA 是基础模型注意力结构，HiLS 需要适配它。

## 3. _project_qkv：特征投影

输入 X:[B,L,D]，普通投影：

$$
Q=XW_Q,\quad K=XW_K,\quad V=XW_V
$$

重排后：

$$
Q:[B,L,H_q,d],\quad K,V:[B,L,H_{kv},d]
$$

若启用独立 Landmark Query 投影：

$$
Q_{lmk}=XW_{lmk}
$$

可选低秩投影配置，并可能与普通 Q 投影结果做残差相加。之后进行对应归一化。

## 4. _apply_hils_rope：位置编码

当 apply_hils_rope 开启时，对普通 Q/K 应用 RoPE；独立 Landmark Query 也通过对应路径旋转。关闭时，相关张量保持原表示。

RoPE 使内积包含相对位置信息。它不是 HiLS 的 Chunk 检索算法本身，而是与基础 Transformer 位置编码兼容的处理。

## 5. _compute_landmark_keys：构造 Chunk 表示

设 Chunk Size 为 S，完整 Chunk 数：

$$
N_c=\lfloor L/S\rfloor
$$

将 K 从 [B,L,Hkv,d] 重排为 [B,Nc,S,Hkv,d]。

启用 prior query 时，通常从每个 Chunk 的最后 Token 位置取 Landmark Query：

$$
\mu_c=Q_{lmk}[:,S-1::S,:,:]
$$

再调用 Chunk Pooling 获得压缩 Key 和 bias：

$$
(K'_c,b'_c)=\operatorname{Pool}(\mu_c,K_c)
$$

若使用 GQA，源码通过 _repeat_k_heads_for_gqa 扩展 KV Head，以匹配 Landmark Query Head。

该函数还处理：
- Prefill/训练时的完整 Chunk 构造；
- Decode/Cache continuation 时只对新增完整 Chunk 计算表示；
- 将先前压缩表示保存在 past_key_values._hils_prior_cache；
- enable_chunk_pooling 开启时以均值池化替代注意力池化；
- 不启用 prior query 时的备用 Landmark 构造路径；
- 不完整尾部 Chunk 不应被误认为已经完成池化的完整 Chunk。

## 6. forward：总体前向流程

按源码顺序概括：

1. 读取输入形状和位置 Embedding。
2. 调用 _project_qkv。
3. 调用 _apply_hils_rope。
4. 在推理 Cache 模式下更新普通 K/V Cache。
5. 调用 flex_attn_tl 计算 Sliding Window Attention 输出及归一化相关量。
6. 调用 _compute_landmark_keys 构造 Chunk 表示。
7. 调用 topk_func，得到选中 Chunk 的 indices 和 scores。
8. 若有 prior bias，根据 indices gather 对应 bias 并加到分数中。
9. 将 Chunk 分数与局部窗口归一化信息组合，计算分支权重。
10. 调用 hils_func，使用选中 Chunk 的原始 Q/K/V 做稀疏细粒度 Attention。
11. 合并局部窗口输出与 HiLS 输出。
12. 合并 Head、执行 o_proj，返回结果。

概念链路：

hidden_states → Q/K/V + Landmark Q → RoPE/Cache → SWA → Chunk Pooling → Top-K → Sparse HiLS Attention → Branch Merge → o_proj

具体路径受训练/推理模式、Cache、序列长度和配置开关影响。

## 7. 重要辅助函数

| 函数 | 作用 |
|---|---|
| _repeat_k_heads_for_gqa | 将 KV Head 扩展到对应 Query Head 数 |
| _compute_landmark_keys | 生成每个 Chunk 的压缩表示和可选 bias |
| _apply_hils_rope | 按配置应用 RoPE |
| chunk_attn_pool | 校验张量形状并调用 TileLang 池化实现 |
| repeat_kv | Eager Attention 路径中的 GQA KV 扩展 |
| eager_attention_forward | 标准 PyTorch Attention 参考实现 |

## 8. 关键张量形状

| 张量 | 典型形状 |
|---|---|
| hidden_states | [B,L,D] |
| Q | [B,L,Hq,d] |
| K/V | [B,L,Hkv,d] |
| k_chunked | [B,Nc,S,Hkv,d] |
| lmk_k | [B,Nc,Hq,d]（GQA 展开后） |
| prior_b | [B,Nc,Hq] |
| indices | 源码注释中为 [B,L,Hkv,K] |
| scores | 源码注释中为 [B,L,Hq,K] |

具体 Kernel 内部可能采用不同布局，需以实际实现为准。

## 9. 需要进一步逐行验证的细节

- Top-K kernel 对 GQA Head 的索引布局与广播方式；
- lse_sum 的定义、形状和数值含义；
- enable_softmax1 的归一化公式；
- mask_last_token=True 在稀疏 Attention Kernel 中的精确语义；
- Cache continuation 时新增 Chunk 边界的计算；
- 训练模式下反向传播经过哪些自定义 Kernel。

本文是公开源码的结构化导读，不是独立运行验证报告。源码更新后应重新核对。
