---
title: MLSys Engineer — 推理系统 Q&A
description: MLSys Engineer — 推理系统 Q&A
published: true
date: 2026-09-27T09:17:34.000Z
tags: 学习资料
editor: markdown
dateCreated: 2026-09-27T09:17:34.000Z
---

# MLSys Engineer — 推理系统 Q&A

**Collection：** [MLSys Engineer](/学习资料/AI硬件工程师路线图/阶段6-面试准备/01-MLSys工程师/README) | **Up：** [Interview Preparation](/学习资料/AI硬件工程师路线图/阶段6-面试准备/README)

`#kv-cache` `#attention` `#speculative-decode` `#latency` `#trt-llm` `#edge`

---

> 这些答案按 senior/staff 级别撰写。每一条都包含问题陈述、机制、真正的工程洞见，以及至少一个生产环境坑点。在实际面试中，每个答案控制在 3–4 分钟——长到足以讲到关键取舍，短到面试官不会在你讲到之前打断你。

---

## Q1. PagedAttention 是如何工作的？

**它解决的问题：** 朴素的推理服务在分配时按请求预留一块连续的 `max_seq_len` KV 缓冲区。这会以两种方式浪费内存：内部碎片（预留块中未填充的槽位）和外部碎片（无法重排请求之间的空间）。vLLM 在这套模式下测得 60–80% 的有效内存浪费。

**机制：** PagedAttention 把 OS 虚拟内存分页应用到 KV cache 上。该缓存被划分为固定大小的 **block**（通常每个 block 存 16 个 token 的 K 和 V）。每个序列有一张 **block table**——从逻辑 token 位置 → 物理 block 索引的映射——由 CPU 侧的 block 管理器维护。attention kernel 被重写为通过这一间接层 gather K 和 V，而不是访问连续缓冲区。

**为什么这有帮助：**

- 内部碎片以一个 block 为界（每个序列最多浪费 15 个 token 槽位，而不是多达 `max_seq_len`）
- block 在物理内存中不必连续 → 分配器可以复用分散的页
- 内容相同的 block 可以 **copy-on-write 共享**：beam search 的分支在分叉前共享前缀；并行采样共享 prompt；常驻的 system-prompt 前缀跨所有请求共享（**前缀缓存**）
- KV 增长完全动态——无需预先承诺序列长度

**kernel 重写：** 核心变化在于 attention kernel 的 K/V 访问模式。连续 kernel 索引 `K[seq_offset + i]` 的地方，分页 kernel 转而解引用 `K[block_table[i // block_size] * block_stride + (i % block_size)]`。这一间接层会多花几个寄存器，但相对于 HBM 带宽瓶颈可以忽略。

**生产环境注意事项——前缀缓存的有效性：** 缓存条目以 token id 的哈希为键。BOS/EOS 或 system prompt 只要有一处不同，整个前缀就会失效。在多租户推理服务中，缓存命中率高度依赖工作负载——在宣称前缀缓存带来加速之前，先用你的实际流量做 benchmark。

**边缘场景：** 在单 stream 的边缘设备上，多租户带来的收益更小。我把预分配池的大小定为 `actual_context_budget`，而不是采用分页 block。但前缀共享的思路可以直接迁移：我那个常驻的跨轮次 KV 缓冲区（warm start 时从磁盘填充）是同一个洞见——不要对已经计算过的前缀重新做 prefill（首字前的整段计算）。

---


<details>
<summary>English original</summary>

**MLSys Engineer — Inference Systems Q&A**

**Collection:** [MLSys Engineer](/学习资料/AI硬件工程师路线图/阶段6-面试准备/01-MLSys工程师/README) | **Up:** [Interview Preparation](/学习资料/AI硬件工程师路线图/阶段6-面试准备/README)

`#kv-cache` `#attention` `#speculative-decode` `#latency` `#trt-llm` `#edge`

---

> These answers are written at senior/staff level. Each one has a problem statement, a mechanism, the real engineering insight, and at least one production gotcha. In an actual interview, aim for 3–4 minutes per answer — long enough to hit the key tradeoff, short enough that the interviewer doesn't interrupt before you get there.

---

**Q1. How does PagedAttention work?**

**The problem it solves:** naive serving reserves a contiguous `max_seq_len` KV buffer per request at allocation time. That wastes memory two ways: internal fragmentation (unfilled slots in the reserved block) and external fragmentation (you can't repack the space between requests). vLLM measured 60–80% effective memory waste in this model.

**The mechanism:** PagedAttention applies OS virtual-memory paging to the KV cache. The cache is partitioned into fixed-size **blocks** (typically 16 tokens of K and V per block). Each sequence has a **block table** — a mapping from logical token position → physical block index — that is maintained by a CPU-side block manager. The attention kernel is rewritten to gather K and V through that indirection rather than accessing a contiguous buffer.

**Why this helps:**

- Internal fragmentation is bounded by one block (≤15 wasted token slots per sequence instead of up to `max_seq_len`)
- Blocks need not be contiguous in physical memory → the allocator can reuse scattered pages
- Blocks with identical content can be **copy-on-write shared**: beam search branches share prefixes until they diverge; parallel samples share the prompt; persistent system-prompt prefixes are shared across all requests (**prefix caching**)
- KV growth is fully dynamic — no pre-commitment to a sequence length

**The kernel rewrite:** the core change is in the attention kernel's K/V access pattern. Where a contiguous kernel indexes `K[seq_offset + i]`, a paged kernel dereferences `K[block_table[i // block_size] * block_stride + (i % block_size)]`. This indirection costs a few registers but is negligible against the HBM bandwidth bound.

**Production caveat — prefix caching validity:** cache entries are keyed by token-id hashes. A single BOS/EOS or system-prompt variation invalidates the entire prefix. In multi-tenant serving, cache hit rate is heavily workload-dependent — benchmark it on your actual traffic before claiming prefix-cache speedup.

**Edge context:** on a single-stream edge device the multi-tenant win is smaller. I sized a pre-allocated pool to `actual_context_budget` instead of adopting paged blocks. But the prefix-sharing idea transfers directly: my persistent cross-turn KV buffer (hydrated from disk on warm start) is the same insight — don't re-prefill a prefix you already computed.

---

</details>

## Q2. 如何在 TensorRT-LLM 中实现 EAGLE-3？

**EAGLE-3 做了什么：** EAGLE 在特征层面而非 token 层面进行 draft。一个微小的 draft head 预测目标在下一位置的 **隐藏状态**，然后从 draft logits 中采样以构建候选 token 树。目标使用树 attention mask 在一次前向传播中验证整个树。EAGLE-3 特别去掉了 EAGLE-1/2 的特征回归损失约束，取而代之的是融合来自目标低层、中层和最终层的隐藏状态作为 draft 输入——这提高了接受长度，同时避免了回归约束的不稳定性。

**在 TRT-LLM 中具体实施，分阶段：**

**阶段 1 — 暴露目标隐藏状态。** 修改基础模型的 TRT engine，使其从三个层（早期 ≈ L/4、中期 ≈ L/2、接近最终层）输出隐藏状态作为额外的输出。在 TRT-LLM 的 Eagle 实现（`eagle_base`）中，这就是 `emit_hidden_states` 标志——基础 engine 在输出 logits 的同时，从这些检查点导出拼接后的隐藏状态。

**阶段 2 — draft head。** 一个小的 1–2 层 Transformer（即 EAGLE draft head）将拼接后的隐藏状态作为输入（无需上下文重新编码），并自回归地构建 draft token 树。该树具有可配置的宽度和深度；典型情况：深度 1 有 4–6 个候选，深度 2 有 2–3 个（总共 10–20 个节点）。

**阶段 3 — 树 attention。** 验证过程同时在所有树节点上运行目标一次。每个节点仅关注树中的祖先节点——通过自定义 `attention_mask`（在树内为上三角，具有适当的父子结构）加上自定义 `attention_pos_id` 实现，以使每个节点的位置编码与其在因果前缀中的深度匹配。TRT-LLM 的 attention plugin 接受 `attention_mask` 和 `position_ids` 覆盖参数，正是为了这种情况。

**阶段 4 — accept + KV 压缩。** 从根开始遍历树，接受最长的匹配前缀（与标准 spec-decode 相同的随机接受规则），仅保留被接受节点的 KV 条目，丢弃被拒绝的分支。将树 KV 压缩为线性 KV 是棘手部分：它是对 KV 缓冲区进行原地 gather，以被接受 token 的树路径为键。

**阶段 5 — 注册为 drafter。** 接入 TRT-LLM 的 spec-decode 调度循环：draft N 棵树 → 验证 batch → accept → 继续。在 batch=1 时收益很大，因为目标前向传播受权重读取限制——验证 K 个 token 的 HBM 开销大致与 decode（逐 token 生成阶段）1 个 token 相同，因此在良好的接受率下，你能以约 1 个 token 的带宽开销获得 K 个被接受的 token。

**注意：** EAGLE 的特征依赖性意味着 `draft(t+1)` 在 `verify(t)` 完成之前无法开始（它需要来自 t 的目标隐藏状态）。与 token 级外部 draft（1B 模型）不同，你无法跨步骤重叠 draft 和验证。在并行度低的边缘硬件上，这会减少 wall-clock 收益。

---


<details>
<summary>English original</summary>

**Q2. How would you implement EAGLE-3 in TensorRT-LLM?**

**What EAGLE-3 does:** EAGLE drafts at the feature level rather than the token level. A tiny draft head predicts the target's **hidden state** for the next position, then samples from the draft logits to build a tree of candidate tokens. The target verifies the entire tree in one forward pass using a tree attention mask. EAGLE-3 specifically drops the feature-regression loss constraint of EAGLE-1/2 and instead fuses hidden states from low, mid, and final target layers as the draft input — this raises acceptance length without the regression constraint's instability.

**Concretely in TRT-LLM, phased:**

**Phase 1 — expose target hidden states.** Modify the base model's TRT engine to emit hidden states from three layers (early ≈ L/4, mid ≈ L/2, near-final) as additional outputs. In TRT-LLM's Eagle implementation (`eagle_base`), this is the `emit_hidden_states` flag — the base engine exports concatenated hidden states from those checkpoints alongside the logits.

**Phase 2 — draft head.** A small 1–2 layer transformer (the EAGLE draft head) takes the concatenated hidden states as input (no context re-encoding) and autoregressively builds a draft token tree. The tree has configurable width and depth; typical: 4–6 candidates at depth 1, 2–3 at depth 2 (total 10–20 nodes).

**Phase 3 — tree attention.** Verification runs the target once on all tree nodes simultaneously. Each node attends only to its ancestors in the tree — implemented as a custom `attention_mask` (upper-triangular within the tree, with the appropriate parent-child structure) plus custom `attention_pos_id` so that each node's position encoding matches its depth in the causal prefix. TRT-LLM's attention plugin accepts `attention_mask` and `position_ids` overrides for exactly this case.

**Phase 4 — accept + KV compaction.** Walk the tree from root, accept the longest matching prefix (same stochastic acceptance rule as standard spec-decode), keep only the accepted nodes' KV entries, discard rejected branches. Compacting the tree KV → linear KV is the fiddly part: it's an in-place gather over the KV buffer, keyed by the accepted token's tree path.

**Phase 5 — register as a drafter.** Wire into TRT-LLM's spec-decode scheduling loop: draft N trees → verify batch → accept → continue. At batch=1 the payoff is large because the target forward is weight-read-bound — verifying K tokens costs roughly the same HBM as decoding 1, so you get K accepted tokens for ~1 token's bandwidth cost (at good acceptance rates).

**Gotcha:** EAGLE's feature dependence means `draft(t+1)` cannot start until `verify(t)` completes (it needs the target's hidden states from t). Unlike token-level external draft (1B model), you cannot overlap draft and verify across steps. On edge hardware with low parallelism this reduces the wall-clock benefit.

---

</details>

## Q3. 如何在内存受限设备上优化 KV cache？

**Context：** 我的 Orin 工作面向 8 GB 共享 LPDDR5 上的 Gemma 4 4B —— 每个字节都很重要。

**优先级顺序：**

**1. 将 KV 量化为 INT8。** 按 token 缩放（scale = `max(|K_row|) / 127`），在 attention softmax 之前反量化。相比 FP16 内存减少约 50%，且在典型激活值上的漂移以 FP16 ULP 为界。我在多数模型上将其作为默认方案发布。

*Gemma 4 例外：* Gemma 4 的 V 激活值在某些 layer 中有约 10× RMS 离群值。按行 INT8 会把它们压成截断值，导致长上下文摘要出现肉眼可见的质量下降。我对 Gemma 4 强制使用 FP16 KV，并记录在案。教训：离群值感知方案（保留少量按通道的 FP16 槽位）可以把 INT8 方法推广开 —— 但需按架构逐一验证。

**2. 利用 GQA/MQA。** 每减少一个 KV head，KV 内存就线性下降。Gemma 4 4B 使用 4 个 KV head（对比 8 个 Q head）→ 在任何量化之前，架构层面就已经带来 2× 的 KV 缩减。GQA 是 KV 内存方面单一杠杆最高的架构旋钮。

**3. 考虑 KV 共享的 layer。** Gemma 4 有 34 个 transformer layer，其中只有 15 个 **拥有**自己的 KV（后 19 个 layer 通过 `kv_shared_layers` 映射复用前面某个 layer 的 K/V）。按 15 个 layer 而非 34 个 layer 分配 KV pool —— 相比朴素计算，pool 大小减少 57%。

**4. 限制滑动窗口 KV。** Gemma 4 的局部 attention layer 只需要最后 W=1024 个 token，而非完整上下文。把每个局部 layer 的 KV buffer 实现为 W 个条目的环形结构。无论上下文长度多大，局部 layer 的 KV 都是 O(1)。

**5. 预分配 pool + OOM 防护。** 在边缘端，绝不要放任 KV 无界增长 —— 启动时按硬上限一次性分配整个 pool，pool 满时拒绝新请求，而不是冒生成中途 OOM 的风险。需要计入：

```text
pool_bytes = num_own_layers × num_kv_heads × max_ctx_tokens × head_dim × dtype_bytes
           + overhead (block tables, metadata)
```

**6. 持久化前缀（prefix caching）。** 热启动时从磁盘载入 system prompt / 持久上下文的 KV。这样可降低第 2–N 轮的 TTFT，并避免对已算过的上下文重复 prefill。实测收益：在 1261 token 的上下文上，冷启动 TTFT 877 ms → 热启动 TTFT 444 ms。

**7. 为合并访问设计布局。** 按 `[layer, head, token, dim]` 顺序存储 KV，使单个 warp 能连续读取一个 head 的 token 切片。用 `cache_head_dim` 步长字段处理参差不齐的 head 维度（Gemma 4 的 256 维滑动 K 为对齐存放在 512 宽的槽位中）。

**8. 驱逐（最后手段）。** 极端上下文用 StreamingLLM 式的 sink+recent 驱逐（保留前 S 个 token + 后 W 个 token）或 H2O（heavy hitters）。这些方法会降低输出质量 —— 记录这一取舍并实测。

---


<details>
<summary>English original</summary>

**Q3. How do you optimize KV cache on memory-constrained devices?**

**Context:** my Orin work targeted 8 GB shared LPDDR5 with Gemma 4 4B — every byte matters.

**Priority order:**

**1. Quantize KV to INT8.** Per-token scaling (scale = `max(|K_row|) / 127`), dequantize before attention softmax. ~50% memory reduction vs FP16, and FP16-ULP-bounded drift on typical activations. I shipped it as default for most models.

*Gemma 4 exception:* Gemma 4's V activations have ~10× RMS outliers in certain layers. Per-row INT8 collapses these into clipped values, causing visible quality degradation on long-context summarization. I force FP16 KV for Gemma 4 and document it. Lesson: outlier-aware schemes (keeping a few per-channel FP16 slots) generalize the INT8 approach — but validate per-architecture.

**2. Exploit GQA/MQA.** Each KV head reduction cuts KV memory linearly. Gemma 4 4B uses 4 KV heads (vs 8 Q heads) → 2× KV reduction at the architecture level, before any quantization. GQA is the single highest-leverage architectural knob for KV memory.

**3. Account for KV-sharing layers.** Gemma 4 has 34 transformer layers, of which only 15 **own** their own KV (the trailing 19 layers reuse a prior layer's K/V via the `kv_shared_layers` mapping). Allocate the KV pool for 15 layers, not 34 — a 57% reduction in pool size vs naive accounting.

**4. Cap sliding-window KV.** Gemma 4's local-attention layers only need the last W=1024 tokens, not the full context. Implement the KV buffer as a ring of W entries per local layer. Local layer KV is O(1) regardless of context length.

**5. Pre-allocate the pool + OOM guard.** On edge, never let KV grow unbounded — allocate the full pool at startup with a hard limit, refuse new requests if the pool is full rather than risking mid-generation OOM. Account for:

```text
pool_bytes = num_own_layers × num_kv_heads × max_ctx_tokens × head_dim × dtype_bytes
           + overhead (block tables, metadata)
```

**6. Persist prefixes (prefix caching).** Hydrate the system-prompt / persistent context KV from disk on warm start. This cuts TTFT on turns 2–N and avoids re-prefilling context you already computed. My measured win: 877 ms cold TTFT → 444 ms warm TTFT on a 1261-token context.

**7. Layout for coalesced access.** Store KV in `[layer, head, token, dim]` order so that a single warp reads one head's token slice contiguously. A `cache_head_dim` stride field handles ragged head dims (Gemma 4's 256-d sliding K stored in 512-wide slots for alignment).

**8. Eviction (last resort).** StreamingLLM-style sink+recent eviction (keep first S tokens + last W tokens) or H2O (heavy hitters) for extreme context. These degrade output quality — document the tradeoff and measure it.

---

</details>

## Q4. 如何编写一个融合 attention kernel？

**核心思路（FlashAttention 分块）：** 绝不物化完整的 N×N attention score 矩阵。对 Q 分块；循环遍历 K/V 分块；对每个分块：在寄存器或 SRAM 中计算 `S = Q·Kᵀ`，应用 online softmax，累加 `O += P·V`，最后一次性写出 O。

**Online softmax：** 为每个 query 维护运行最大值 `m` 和运行分母 `l`。当新的 K/V 分块产生新的最大值 `m_new > m_old` 时，在加入新分块的贡献之前，先用 `exp(m_old - m_new)` 重新缩放已有的 O 累加器和 `l`。这样无需物化 S 也能保持数值正确性。

**在真实硬件上至关重要的工程细节：**

**Tensor Core（MMA）：** 对 `Q·Kᵀ` 和 `P·V` 都使用 `m16n8k16` 或 `m16n8k32` MMA atom（fp16/bf16 输入，fp32 累加）。把 softmax 中间结果和 O 累加器保持为 fp32。

*我在 Gemma 4 上踩过的精度陷阱：* `P·V` 矩阵乘里的 fp16 累加在 V 离群值上会崩——累加和发生截断。fp16 输入/fp32 累加没问题（即使在 20× 离群值幅度下，实测 rel-RMS 为 3e-4）。规则：始终用 fp32 累加；只有 MMA 输入可以是 fp16/bf16。

**双缓冲 K/V 分块：** 在 tile `t` 的 MMA 执行时，为 tile `t+1` 发起 `cp.async` 加载。这会把全局内存延迟（HBM 访问）与计算重叠起来。在 Orin 上 D=512、4K 上下文实测：1289ms → 348ms——这是我这个 kernel 中迄今最大的单项延迟收益。

**跳过全被 mask 的分块：** 对滑动窗口 attention，最早相关的 K 分块从 `kt_start = ((q_pos + 1 - W) / KT) * KT` 开始。对完全落在窗口之外的分块，连加载都不要发起——它们对输出的贡献为零，而且能同时省下 HBM 带宽和 MMA 周期。

**Occupancy：** 把 O 累加器放在**寄存器**里，而不是共享内存。放在共享内存里的 O 会把 occupancy 限制到每个 SM 约 1 个活跃 block，因为每个 block 的 shared 分配太大。代价是 online-softmax 的重新缩放必须直接作用在 MMA 寄存器片段上——你需要知道你所用的具体 MMA atom 的 lane 到输出行映射（`lane gid` 拥有行 `{gid, gid+8}`、列 `{2t, 2t+1}`）。

**先看 roofline（性能上界模型）：** 在 Orin 的 8-SM iGPU 上，我实测自己的 attention kernel 是带宽受限的（算术强度 ≈ 16–32 FLOP/byte，远低于 ~190 FLOP/byte 的 ridge point）。真正的收益来自 **queries-per-block**：启动 16 个 query 共享每个已加载的 K/V 分块 = DRAM 流量摊薄 16×，而不是更快的 MMA。当瓶颈是字节时，不要伸手去用张量核心。

---

## Q5. 为什么 FlashAttention 能减少 HBM 流量？

**朴素 attention 的做法：** 把 `S = QKᵀ`（N×N fp16 矩阵）写入 HBM → 再读回来做 softmax → 写入 `P`（N×N）→ 读出 `P` 用于 `P·V`。在序列长度 N=4096、D=128 时，仅一层的 attention 矩阵就产生约 4 GB 的 HBM 流量——而 softmax 在其上完全是带宽受限的。

**FlashAttention 的做法：** 对计算分块，使 `S` 和 `P` 分块始终不离开 SRAM/寄存器。kernel 从 HBM 基本只读一次 Q、K、V，并只写一次 O。HBM 流量从 O(N² · d) 降到 O(N · d)。

**取舍：** 更多 FLOPs。online-softmax 的重新缩放是额外算术；反向传播重新计算 S 分块而不是把它们暂存下来。但关键洞见是：在典型的 N 和 D 下 attention 是**带宽受限的**，所以用充裕的 FLOP 换稀缺的带宽永远是正确的。瓶颈从 HBM 带宽上移开了。

**FlashAttention 何时不再有帮助：** 序列极短（N < 512）时，N×N 矩阵小到本来就能放进缓存；或者 `D` 极端（head_dim=256 或更大）时，Q·Kᵀ 的算术强度终于把天平推向算力受限。Gemma 4 的 head_dim=256 正好处于边缘——在 N=4096 时它仍是带宽受限，但在 N=512、D=256 时分块计算开始占主导。

**反向传播：** 沿用同样的思路。Flash 不再为反向传播存储 P（N×N 激活值），而是在反向传播过程中于 SRAM 中重新计算 S。反向传播的内存占用从 O(N²) 降到 O(N)。这也是 FlashAttention 能大幅提升长上下文训练内存效率的原因。

---


<details>
<summary>English original</summary>

**Q4. How would you write a fused attention kernel?**

**Core idea (FlashAttention tiling):** never materialize the full N×N attention score matrix. Tile Q; loop over K/V tiles; for each tile: compute `S = Q·Kᵀ` in registers or SRAM, apply online softmax, accumulate `O += P·V`, write O once at the end.

**Online softmax:** maintain per-query running max `m` and running denominator `l`. When a new K/V tile produces a new max `m_new > m_old`, rescale the existing O accumulator and `l` by `exp(m_old - m_new)` before adding the new tile's contribution. This keeps numerical correctness without materializing S.

**Engineering details that matter on real hardware:**

**Tensor cores (MMA):** use `m16n8k16` or `m16n8k32` MMA atoms (fp16/bf16 input, fp32 accumulate) for both `Q·Kᵀ` and `P·V`. Keep the softmax intermediates and O accumulator in fp32.

*Precision trap I hit on Gemma 4:* fp16 accumulation in the `P·V` matmul breaks on V-outliers — the accumulated sum clips. fp16-inputs/fp32-accumulate is fine (measured rel-RMS 3e-4 even at 20× outlier amplitudes). Rule: always accumulate in fp32; only the MMA inputs can be fp16/bf16.

**Double-buffer K/V tiles:** issue `cp.async` loads for tile `t+1` while tile `t`'s MMA is executing. This overlaps global-memory latency (HBM access) with compute. Measured impact on D=512, 4K context on Orin: 1289ms → 348ms — by far the single biggest latency win in my kernel.

**Skip fully-masked tiles:** for sliding-window attention, the earliest relevant K tile starts at `kt_start = ((q_pos + 1 - W) / KT) * KT`. Don't even issue a load for tiles that are entirely outside the window — they contribute zero to the output and you save both HBM bandwidth and MMA cycles.

**Occupancy:** keep the O accumulator in **registers**, not shared memory. Shared-memory O limits occupancy to ~1 active block per SM because the shared allocation per block is too large. The cost is that the online-softmax rescale must operate directly on MMA register fragments — you need to know the lane-to-output-row mapping for your specific MMA atom (`lane gid` owns rows `{gid, gid+8}`, columns `{2t, 2t+1}`).

**Roofline first:** on Orin's 8-SM iGPU I measured my attention kernel as bandwidth-bound (arithmetic intensity ≈ 16–32 FLOP/byte, far below the ~190 FLOP/byte ridge point). The real win was **queries-per-block**: launching 16 queries that share each loaded K/V tile = 16× amortization of DRAM traffic, not faster MMA. Don't reach for tensor cores when the bottleneck is bytes.

---

**Q5. Why does FlashAttention reduce HBM traffic?**

**What naive attention does:** write `S = QKᵀ` (N×N fp16 matrix) to HBM → read it back for softmax → write `P` (N×N) → read `P` for `P·V`. At sequence length N=4096, D=128, that's ~4 GB of HBM traffic just for the attention matrix on one layer — and softmax is purely memory-bound over it.

**What FlashAttention does:** tile the computation so that `S` and `P` tiles never leave SRAM/registers. The kernel reads Q, K, V essentially once from HBM and writes O once. HBM traffic drops from O(N² · d) to O(N · d).

**The tradeoff:** more FLOPs. The online-softmax rescale is extra arithmetic; the backward pass recomputes S tiles instead of stashing them. But the key insight is that attention is **memory-bound at typical N and D**, so trading abundant FLOP for scarce bandwidth is always correct. The bottleneck moves off HBM bandwidth.

**When FlashAttention stops helping:** very short sequences (N < 512) where the N×N matrix is small enough to fit in cache anyway, or extreme `D` (head_dim=256 or larger) where the Q·Kᵀ arithmetic intensity finally pushes toward compute-bound. Gemma 4's head_dim=256 is right at the edge — at N=4096 it's still bandwidth-bound, but at N=512 with D=256 the tile compute starts to dominate.

**Backward pass:** extends the same idea. Instead of storing P (N×N activations) for the backward, Flash recomputes S in SRAM during the backward pass. Memory for backward drops from O(N²) to O(N). This is why FlashAttention dramatically improves long-context training memory efficiency too.

---

</details>

## Q6. 如何在 Jetson 上调度投机解码？

**机会所在：** Jetson 上 batch=1 的 decode（逐 token 生成阶段）是权重读取受限的 —— GPU 在流式读取权重时空耗 LPDDR5 带宽。在一次 target 前向中验证 K 个 draft token，消耗的 HBM 与解码 1 个 token 大致相同，因为无论哪种方式，权重读取瓶颈都一样。如果平均有 K 个 token 被接受，就能以约 1× 的带宽代价换来 K× 的吞吐。

**draft 模型选择：** 一个与 target 共享 tokenizer 的极小 draft。1–2 层的 EAGLE feature-draft 或 Medusa heads 优于单独一个 1B 模型 —— 完整的第二个模型会争抢同一个 LPDDR5 带宽池，侵蚀收益。共享的 LPDDR5（CPU + draft + target）是 Jetson 上的关键约束，这在数据中心（NVLink 隔离的 HBM）中并不存在。

**经济性核算：** `net_win = accept_len × target_cost - (draft_cost + verify_cost)`。只有满足 `accept_len > 1 + draft_cost / target_cost` 时才有净收益。先在实际任务分布上测量接受率；再按测量值调整树深度与宽度。对代码/技术文本，接受长度通常为 3–4；对推理/数学则更低。

**内层循环：**

```text
while generating:
    draft K tokens (chain or tree) using tiny draft head
    one target forward with tree mask (verify all K in parallel)
    accept longest matching prefix
    repeat
```

**CUDA graphs：** 把 target 的 decode 步捕获为 CUDA graph，以消除 kernel 启动开销（Orin 上每步约 100–200 µs）。问题在于：接受长度每步都在变化，所以 graph 里的 KV 更新是变长的。应对办法是用填充后的定长树 + mask-out（被接受的路径门控 KV 写入），或者按接受数分别维护捕获好的 graph。对 Gemma 4 的逐 token PLE（Per-Layer Embeddings）我关闭 graph 捕获，因为 PLE 依赖位置，而动态位置会破坏静态 graph 捕获。

**功耗/热管理：** 投机解码会提高 GPU 利用率 —— draft + verify 两步都要用 GPU，而纯 decode 在两次 HBM 取数之间让 GPU 处于内存空闲状态。在 Jetson 15–25W 的功耗预算下，这可能引发热降频。报告 tok/s 时务必用持续锁定频率下的值（`jetson_clocks --store; jetson_clocks`），而不是瞬时峰值。

**实际数字：** 在良好的接受率下，2–4B 的带宽受限模型可获得约 1.5–2.5× 的 decode 吞吐。对于 Orin 上的 Gemma 4 4B + E2B drafter：实测约 90 tok/s，基线约 55 tok/s。

---


<details>
<summary>English original</summary>

**Q6. How would you schedule speculative decoding on Jetson?**

**The opportunity:** Jetson decode at batch=1 is weight-read-bound — the GPU idles on LPDDR5 bandwidth while streaming weights. Verifying K draft tokens in one target forward costs roughly the same HBM as decoding 1 token, because the weight-read bottleneck is the same either way. If K tokens are accepted on average, you get K× throughput for ~1× bandwidth cost.

**Draft model selection:** a tiny draft that shares the target's tokenizer. A 1–2-layer EAGLE feature-draft or Medusa heads are better than a separate 1B model — a full second model competes for the same LPDDR5 bandwidth pool, eroding the gain. The shared LPDDR5 (CPU + draft + target) is the key constraint on Jetson that doesn't exist in datacenter (NVLink-isolated HBM).

**Economics check:** `net_win = accept_len × target_cost - (draft_cost + verify_cost)`. Net win only if `accept_len > 1 + draft_cost / target_cost`. Measure acceptance rate first on your actual task distribution; tune tree depth and width to the measured value. For code/technical text, acceptance is typically 3–4; for reasoning/math it's lower.

**The inner loop:**

```text
while generating:
    draft K tokens (chain or tree) using tiny draft head
    one target forward with tree mask (verify all K in parallel)
    accept longest matching prefix
    repeat
```

**CUDA graphs:** capture the target decode step as a CUDA graph to eliminate kernel-launch overhead (~100–200 µs on Orin per step). The problem: accepted length varies each step, so the graph has variable-length KV updates. Work around it with a padded fixed-length tree + mask-out (accepted path gates the KV write), or maintain separate captured graphs per accept-count. I guard graph capture off for Gemma 4's per-token PLE (Per-Layer Embeddings) because PLE is position-dependent and the dynamic position breaks static graph capture.

**Power/thermal:** speculative decode increases GPU utilization — the draft + verify steps both use the GPU, versus decode-only which leaves the GPU memory-idle between HBM fetches. On Jetson's 15–25W budget this can cause thermal throttling. Always report tok/s at sustained locked clock (`jetson_clocks --store; jetson_clocks`), not peak burst.

**Realistic numbers:** ~1.5–2.5× decode throughput on a 2–4B memory-bound model at good acceptance rates. For Gemma 4 4B + E2B drafter on Orin: measured ~90 tok/s vs ~55 tok/s baseline.

---

</details>

## Q7. 如何在 Orin Nano 上降低 TTFT？

**TTFT = 模型加载时间 + prompt prefill 时间。** 两者都是杠杆；对非平凡 prompt 而言 prefill 占主导。

**Prefill 吞吐（主要杠杆）：**

我发现并修复的关键失效点：对超过 1024 token 的 prompt 走了**逐 token 回退**路径。系统对长 prompt 在循环中调用 `decode_one_token()`，而不是调用 `prefill_batch()`——这是 O(N) 次串行前向传播，而不是一次批处理 pass。修复方式：把任意 prompt 切成 `scratch_budget` 大小的片段，每片作为一个批传入。实测结果：在 1261 token 的 Gemma 4 prompt 上，152 → 620 tok/s。

其他 prefill 收益点：
- **Tensor Core GEMM：** 量化投影（Q、K、V、out、gate、up、down）用 INT8 MMQ，attention 用 fp16/fp32。这把 GEMM 从标量 CUDA 核心搬到 Tensor Core 上。
- **Tensor Core attention：** 把标量的 `QKᵀ` 循环换成 cuBLAS 批 GEMM（fp16 输入、fp32 累加）。prefill 时 batch=1 但 N>512，此时这条占主导。
- **批量 RoPE + KV 存储：** 在一次 kernel 调用中对所有 prompt token 施加 RoPE，并在一趟中存好所有 K/V。避免在 Python 循环里逐 token 做 RoPE。
- **跳过被完全 mask 的滑窗分块**（见 Q4）。

**前缀缓存（对 chat 而言单项收益最大）：**

从磁盘恢复系统提示词/对话的 KV，而不是每轮都重新 prefill。我的实测：1261 token 上下文下，冷 TTFT 877 ms → 热 TTFT 444 ms。实现：prefill 之后把 KV 序列化到 memory-mapped 文件；热启动时 mmap + `cudaHostRegister`，直接从 page cache 喂给 GPU（页面命中时即零拷贝）。

**模型加载时间：**

对 GGUF 文件做 `mmap` 并对权重页做 `cudaHostRegister` → GPU 可直接从映射文件读取（零拷贝 DMA）。冷加载受限于 NVMe；热加载（page cache 命中时重启进程）对 Gemma 4 4B INT4 约 1.3 s。保持推理服务进程常驻——请求之间不要重启。

**时钟：**

锁定 MAXN_SUPER + `jetson_clocks`，避免第一次推理突发时的 DVFS 升频。不锁时钟时，首次 prefill 要付出约 200ms 的 DVFS 升频开销，而 llama-bench 的 warm-up pass 会把它丢掉——使 benchmark 看起来比生产环境更好。

**Prompt 压缩：**

把 persona/系统/工具定义内容烧进 LoRA 权重（适配器），让更少 token 进入 prefill 路径。在 Orin 上，500 token 的系统提示词约花 50ms；编码进 LoRA 适配器后，它占 0 个 prefill token。

**务必测量真实的冷启动路径**——`llama-bench pp` 会丢掉一个 warmup 步骤，但你的用户付出的是真实的冷启动代价。在冷进程中用 `nsys profile --trace cuda,osrt` 做 profile。

---


<details>
<summary>English original</summary>

**Q7. How would you reduce TTFT on Orin Nano?**

**TTFT = model load time + prompt prefill time.** Both are levers; prefill dominates for non-trivial prompts.

**Prefill throughput (primary lever):**

The key failure I found and fixed: a **per-token fallback** for prompts over 1024 tokens. The system was calling `decode_one_token()` in a loop instead of `prefill_batch()` for long prompts — that's O(N) sequential forward passes instead of one batched pass. Fixed by chunking any prompt into `scratch_budget`-sized pieces and passing each as a batch. Measured result: 152 → 620 tok/s on a 1261-token Gemma 4 prompt.

Other prefill wins:
- **Tensor-core GEMMs:** INT8 MMQ for quantized projections (Q, K, V, out, gate, up, down), fp16/fp32 for attention. This moves the GEMM from scalar CUDA cores to tensor cores.
- **Tensor-core attention:** replace the scalar `QKᵀ` loop with cuBLAS batched GEMM (fp16-in, fp32-accumulate). At prefill batch=1 but N>512, this dominates.
- **Batched RoPE+KV-store:** apply RoPE to all prompt tokens in one kernel call and store all K/V in one pass. Avoid per-token RoPE in a Python loop.
- **Skip fully-masked sliding-window tiles** (see Q4).

**Prefix caching (biggest single win for chat):**

Hydrate the system-prompt / conversation KV from disk instead of re-prefilling on every turn. My measurement: 877 ms cold TTFT → 444 ms warm TTFT on a 1261-token context. Implementation: after prefill, serialize KV to a memory-mapped file; on warm start, mmap + `cudaHostRegister` to feed GPU directly from the page cache (zero-copy if pages are hot).

**Model load time:**

`mmap` the GGUF file and `cudaHostRegister` the weight pages → GPU can read directly from the mapped file (zero-copy DMA). Cold load is NVMe-bound; warm load (process restarts with hot page cache) is ~1.3 s for Gemma 4 4B INT4. Keep the serving process warm — don't restart between requests.

**Clocks:**

Lock MAXN_SUPER + `jetson_clocks` to avoid DVFS ramp-up on the first inference burst. Without clock locking, the first prefill pays a ~200ms DVFS ramp that llama-bench's warm-up pass discards — making benchmarks look better than production.

**Prompt compression:**

Bake persona/system/tool-definition content into LoRA weights (adapter) so fewer tokens hit the prefill path. A 500-token system prompt costs ~50ms on Orin; encoded in a LoRA adapter it costs 0 prefill tokens.

**Always measure the actual cold path** — `llama-bench pp` discards a warmup step, but your users pay a real cold start. Profile with `nsys profile --trace cuda,osrt` from a cold process.

---

</details>

## Q8. 如何将新的模型架构集成进 TensorRT-LLM？

**背景：** 这正是我完成的 Gemma 4 → TensorRT-Edge-LLM 移植，包含那些分叉特性：按 layer 类型不同的 head dims、双 RoPE、QK+V norm、单位 attention scaling、Per-Layer Embeddings、KV-sharing、GeGLU、soft-capped logits。

**阶段 1 — 识别 + 配置解析。**

把 `config.json` `model_type` 映射为一个解析后的配置对象。字段检查清单：head dims（按 layer 类型可能不同）、RoPE 参数（`rope_theta`、`rope_scaling`，是部分还是全量）、layer 类型标注（交错的 local/global 用 `attention_type` 列表）、norm 类型（RMSNorm vs LayerNorm，以及位置）、KV-sharing（`kv_shared_layers` 映射）、soft-cap（`attn_logit_softcapping`）。注册 `model_type → model_class`。遇到无法识别的特性必须大声报错——一个被静默当成 Llama 错误构建的检查点能跑起来、产出垃圾，然后耗掉几个小时去调试。

**阶段 2 — Python 模型定义。**

用框架的模块组装模型：`QuantizedLinear`、attention plugin、RoPE op、RMSNorm。只有在架构确实分叉的地方才写自定义代码。

*Gemma 4 的分叉点及其解法：*

| 特性 | 解法 |
|---------|---------|
| 按 layer 类型不同的 head dims（local：256，global：512） | 在 attention plugin 调用中把 `head_dim` 参数化；传入 `layer_type` 索引 |
| 双 RoPE（两张独立的 cos/sin 表） | 两个模型输入；在 runtime 初始化时预计算两张表 |
| QK-norm + V-norm | 在 attention 点积之前对 Q 和 K 插入 RMSNorm；对投影之后的 V 做 RMSNorm |
| 单位 attention scaling | 在 RoPE 之前把 Q 预乘 `√head_dim`，以抵消 plugin 内置的 `1/√head_dim`；验证 RoPE 与该标量可交换（确实可以——RoPE 是旋转，scaling 是标量乘法） |
| Per-Layer Embeddings | 第二条 embedding 通路：一个额外的整型输入（PLE token）+ 图内 embedding 查表 + 加到 residual stream |
| KV-sharing（尾部 layer 复用之前的 KV） | 传入源 layer 的物理 KV buffer 地址，而不新分配 KV；在 pool 中通过 `kv_shared_layers` 别名实现 |
| GeGLU | gate proj + up proj 组成双倍宽度的 linear，然后在单个融合 kernel 里做 `gate ⊙ gelu(up)` |
| Soft-cap | 在 lm_head 之后施加 `logits = tanh(logits / cap) * cap` |

**阶段 3 — 导出契约。**

对 ONNX→TRT 技术栈：定义 forward 签名——输入、dtype、动态轴、名字。新的架构特性会成为 runtime 必须提供的新图输入（我加了第二个 RoPE 表输入、一个 PLE 整型输入，以及图中作为常量的 `layer_type`）。用 dynamo exporter 配合自定义 TRT op 转换表导出；在写下第一行 runtime C++ 之前先做结构校验（实例化 + 导出 → 用 `onnx.checker.check_model` 检查 ONNX 图）。

**阶段 4 — Plugin + runtime。**

把新的 op 映射到 TRT plugin：双 RoPE 预计算（扩展 RoPE fuser）、PLE 查表（新的 embedding plugin）、KV-share 别名（扩展 cache manager）。每个新 plugin 都是一个风险点——要单独对参考实现（HuggingFace 或 numpy）做单元测试。

**阶段 5 — 权重加载。**

把 HuggingFace 检查点的 key 名 → TRT-LLM 参数名做映射。注意：tied embeddings（lm_head 与 `embed_tokens.weight` 共享）、按 layer 类型的权重 shape（local head != global head）、KV-sharing layer 上被丢弃的权重（这些 layer 在检查点里没有 KV 投影）。

**阶段 6 — 验证。**

贪心一致性检查：在 5 条事实型 prompt 上，用 `temperature=0` 与 HuggingFace 逐 token 对比。按 layer 做张量对比以定位回归（`model.forward(return_all_hidden_states=True)` 对比是最快的调试工具）。在最大支持长度上做长上下文测试。然后看性能：prefill（首字前的整段计算）吞吐、decode（逐 token 生成阶段）tok/s、KV 显存核算。

**降风险策略：** 我先在一个更简单的 GGUF runtime 上做到与 `llama.cpp` 贪心一致。这在独立于 TRT 机制的前提下验证了数学部分。之后 TRT 移植的风险就是机械性的（图构建、plugin 接线）而非数学性的——调试起来容易得多。

**每个 PR 只解决一个问题：** 识别 → 建模+导出 → runtime → 性能。不要把权重加载的 bug 和 attention plugin 的 bug 混在一起——你将无法隔离它们。

---


<details>
<summary>English original</summary>

**Q8. How would you integrate a new model architecture into TensorRT-LLM?**

**Context:** this is exactly the Gemma 4 → TensorRT-Edge-LLM port I completed, including divergent features: per-layer-type head dims, dual RoPE, QK+V norm, unit attention scaling, Per-Layer Embeddings, KV-sharing, GeGLU, soft-capped logits.

**Phase 1 — Recognition + config parsing.**

Map `config.json` `model_type` → a parsed config object. Field checklist: head dims (may differ per layer type), RoPE params (`rope_theta`, `rope_scaling`, whether partial or full), layer type annotations (`attention_type` list for interleaved local/global), norm types (RMSNorm vs LayerNorm, placement), KV-sharing (`kv_shared_layers` mapping), soft-cap (`attn_logit_softcapping`). Register `model_type → model_class`. Fail loud on unrecognized features — a checkpoint that silently mis-builds as Llama will run, produce garbage, and take hours to debug.

**Phase 2 — Python model definition.**

Compose the model from the framework's modules: `QuantizedLinear`, the attention plugin, RoPE op, RMSNorm. Only write custom code where the arch genuinely diverges.

*Gemma 4 divergences and their solutions:*

| Feature | Solution |
|---------|---------|
| Per-layer-type head dims (local: 256, global: 512) | Parameterize `head_dim` in the attention plugin call; pass `layer_type` index |
| Dual RoPE (two separate cos/sin tables) | Two model inputs; precompute both tables at runtime init |
| QK-norm + V-norm | Insert RMSNorm on Q and K before attention dot-product; RMSNorm on V after projection |
| Unit attention scaling | Pre-scale Q by `√head_dim` before RoPE to cancel the plugin's built-in `1/√head_dim`; verify RoPE commutes with this scalar (it does — RoPE is a rotation, scaling is a scalar multiply) |
| Per-Layer Embeddings | Second embedding pathway: an extra integer input (PLE token) + in-graph embedding lookup + add to the residual stream |
| KV-sharing (trailing layers reuse prior KV) | Pass the physical KV buffer address of the source layer instead of allocating new KV; implement via `kv_shared_layers` aliasing in the pool |
| GeGLU | Gate proj + up proj double-wide linear, then `gate ⊙ gelu(up)` in a single fused kernel |
| Soft-cap | `logits = tanh(logits / cap) * cap` applied after the lm_head |

**Phase 3 — Export contract.**

For ONNX→TRT stacks: define the forward signature — inputs, dtypes, dynamic axes, names. New arch features become new graph inputs that the runtime must supply (I added a second RoPE table input, a PLE integer input, and `layer_type` as a constant in the graph). Export via the dynamo exporter with the custom TRT op translation table; structurally validate (instantiate + export → check the ONNX graph with `onnx.checker.check_model`) before writing a single line of runtime C++.

**Phase 4 — Plugins + runtime.**

Map new ops to TRT plugins: dual-RoPE precompute (extend the RoPE fuser), PLE lookup (new embedding plugin), KV-share aliasing (extend the cache manager). Each new plugin is a risk — unit-test each one in isolation against a reference (HuggingFace or numpy).

**Phase 5 — Weight loading.**

Map HuggingFace checkpoint key names → TRT-LLM parameter names. Watch for: tied embeddings (lm_head shares `embed_tokens.weight`), per-layer-type weight shapes (local head != global head), dropped weights on KV-sharing layers (those layers have no KV projections in the checkpoint).

**Phase 6 — Validate.**

Greedy-identical check: compare token-by-token against HuggingFace on 5 factual prompts with `temperature=0`. Per-layer tensor comparison to isolate regressions (a `model.forward(return_all_hidden_states=True)` comparison is the fastest debug tool). Long-context test at max supported length. Then perf: prefill throughput, decode tok/s, KV memory accounting.

**De-risk strategy:** I got greedy-identical to `llama.cpp` in a simpler GGUF runtime first. That validated the math independently of TRT mechanics. The TRT port's risk then was mechanical (graph construction, plugin wiring) not mathematical — much easier to debug.

**One concern per PR:** recognition → modeling+export → runtime → perf. Don't mix weight-loading bugs with attention-plugin bugs — you won't be able to isolate them.

---

</details>

## 快问快答校准题

这些题考察你能否给出精确的 30 秒回答——相当于面试里检查单位是否写对。

**Q：decode（逐 token 生成阶段）阶段的 GEMM（矩阵-矩阵乘）算术强度是多少，它意味着什么？**

A：batch=1（M=1）时 AI = `2·M·K·N / (M·K + M·N + K·N) bytes`：`2·K·N / (K + N + K·N) ≈ 2 / (1/K + 1/N)`。K=N=4096 时，AI ≈ 2 FLOP/byte。H100 的 ridge point 约 300 FLOP/byte——decode GEMM 比 ridge 低约 150×。它完全是内存带宽受限的。含义：更快的计算单元没用；更多的 HBM 带宽或权重量化（要 stream 的字节更少）才有用。

**Q：张量并行和流水线并行有什么区别？**

A：TP 把每层的权重矩阵切分到各设备上（MLP 用列并行 + 行并行；attention 把 Q/K/V head 分散到各设备）——每一层都在所有设备上运行，每层之后用 `AllReduce` 通信。对延迟友好，对带宽开销大。PP 切分模型深度——不同的 layer 位于不同设备上，用点对点 `send`/`recv` 相连。通信量更低，但会引入流水线气泡（设备在等待本阶段输入时空闲）。对于大 batch 推理服务，PP 降低每步的 AllReduce 压力；对于延迟敏感的单个请求路径，TP 通常更优，因为它没有流水线空闲。

**Q：为什么 GQA 的 KV 内存比 MHA 小？**

A：MHA 为每个 query head 存一个 K 和 V head。GQA 把 G 个 query head 分组共享一个 K/V head。内存：MHA → `num_heads × 2 × seq_len × head_dim`；GQA → `num_kv_heads × 2 × seq_len × head_dim`，其中 `num_kv_heads = num_heads / G`。对于 Gemma 4 4B（G=2）：KV 内存是 MHA 的一半。吞吐：每步的 KV HBM 读取量减半 → 仅靠带宽，decode 吞吐上限就提升 2×。

**Q：GGUF 中 Q8_0 和 Q4_K_M 有什么区别？**

A：Q8_0 是 8-bit 整数分块量化，每 32 元素块一个 FP32 scale。每个权重有效约 8.5 bit。质量高，压缩中等。Q4_K_M 是 k-quant：4-bit（M = mixed，某些关键 layer 用 6-bit），每 256 元素块有分组的 scales 和 mins，scale 本身用 6-bit 子量化。每个权重有效约 4.5 bit。由于 scale 做了子量化，多数模型上的质量损失低得出奇。Q4_K_M 是边缘部署的标准推荐——它把 Gemma 4 4B 装进 2.2 GB，而 BF16 需要 8.2 GB。

---

*上级：[MLSys Engineer](/学习资料/AI硬件工程师路线图/阶段6-面试准备/01-MLSys工程师/README) | 返回：[Interview Preparation](/学习资料/AI硬件工程师路线图/阶段6-面试准备/README)*


<details>
<summary>English original</summary>

**Quick-fire calibration questions**

These test your ability to give a precise 30-second answer — the interview equivalent of checking your units.

**Q: What is the arithmetic intensity of a decode-phase GEMM, and what does it imply?**

A: AI = `2·M·K·N / (M·K + M·N + K·N) bytes` at batch=1 (M=1): `2·K·N / (K + N + K·N) ≈ 2 / (1/K + 1/N)`. For K=N=4096, AI ≈ 2 FLOP/byte. The H100's ridge point is ~300 FLOP/byte — decode GEMM is ~150× below the ridge. It's entirely memory-bandwidth-bound. Implication: faster math units do not help; more HBM bandwidth or weight quantization (fewer bytes to stream) does.

**Q: What's the difference between tensor parallelism and pipeline parallelism?**

A: TP splits each layer's weight matrices across devices (column-parallel + row-parallel for MLP; Q/K/V heads across devices for attention) — every layer runs on all devices, communicating with `AllReduce` after each layer. Latency-friendly, bandwidth-expensive. PP splits the model depth — different layers live on different devices, connected by point-to-point `send`/`recv`. Lower communication volume but adds pipeline bubbles (devices idle while waiting for their stage's input). For large-batch serving, PP reduces the AllReduce pressure per step; for latency-critical single-request paths, TP typically wins because it has no pipeline idle.

**Q: Why is GQA KV memory smaller than MHA?**

A: MHA stores one K and V head per query head. GQA groups G query heads to share one K/V head. Memory: MHA → `num_heads × 2 × seq_len × head_dim`; GQA → `num_kv_heads × 2 × seq_len × head_dim` where `num_kv_heads = num_heads / G`. For Gemma 4 4B (G=2): half the KV memory vs MHA. Throughput: the KV HBM read per step is halved → 2× decode throughput ceiling from bandwidth alone.

**Q: What is the difference between Q8_0 and Q4_K_M in GGUF?**

A: Q8_0 is 8-bit integer per-block quantization with one FP32 scale per 32-element block. ~8.5 bits per weight effective. High quality, moderate compression. Q4_K_M is a k-quant: 4-bit (M = mixed, some critical layers use 6-bit), grouped scales and mins per 256-element block using a 6-bit sub-quantization for the scales. ~4.5 bits per weight effective. Quality loss is surprisingly low on most models because of the scale sub-quantization. Q4_K_M is the standard recommendation for edge deployment — it fits Gemma 4 4B in 2.2 GB vs 8.2 GB BF16.

---

*Up: [MLSys Engineer](/学习资料/AI硬件工程师路线图/阶段6-面试准备/01-MLSys工程师/README) | Back to: [Interview Preparation](/学习资料/AI硬件工程师路线图/阶段6-面试准备/README)*

</details>

---

> 原文：[`Phase 6 - Interview Preparation/1. MLSys Engineer/01-Inference-Systems-QA.md`](https://github.com/ai-hpc/ai-hardware-engineer-roadmap/blob/main/Phase%206%20-%20Interview%20Preparation/1.%20MLSys%20Engineer/01-Inference-Systems-QA.md)（上游仓库 ai-hpc/ai-hardware-engineer-roadmap，MIT License）。本页为中文翻译，术语保留英文原词；与原文不一致时以原文为准。
