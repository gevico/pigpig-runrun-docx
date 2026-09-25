---
title: roofline模型分析llm推理的decode和prefill
description: roofline模型分析llm推理的decode和prefill
published: true
date: 2026-09-25T18:56:58.561Z
tags: 
editor: markdown
dateCreated: 2026-09-25T18:56:58.561Z
---

# 基于 Roofline 模型的 Qwen2.5-0.5B 在 V100 上的推理性能分析

> 本文以 NVIDIA V100-SXM2-32GB 为目标硬件，用 Roofline 模型分析 Qwen2.5-0.5B 在 **prefill** 和 **decode** 两个阶段的访存字节数（Bytes）与浮点运算量（FLOPs），并判定各自的瓶颈性质。

---

## 一、硬件参数与模型常量

### 1.1 硬件（V100-SXM2-32GB，全部实测）

| 符号 | 含义 | 值 |
|---|---|---|
| BW | 实测显存带宽 | 819.27 GB/s |
| PEAK | 实测峰值算力（FP16 Tensor Core） | 99.50 TFLOPS |
| RIDGE | 拐点 = PEAK / BW | **121.4 FLOP/Byte** |

**RIDGE 是一切判据的核心：**

- 运算强度 AI **< 121.4**：带宽受限
- 运算强度 AI **> 121.4**：算力受限

---

### 1.2 模型常量（Qwen2.5-0.5B）

| 符号 | 含义 | 值 |
|---|---|---|
| L | Decoder 层数 | 24 |
| HID | 隐藏维度 | 896 |
| INT | MLP 中间维度 | 4864 |
| H | Query head 数 | 14 |
| KV | KV head 数 | 2 |
| D | head 维度 | 64 |
| V | 词表大小 | 151936 |
| BYTES | 每参数字节数（FP16） | 2 |
| P_lin | Decoder 线性层参数 | 357,826,560 |
| P_lm | lm_head 参数 | 136,134,656 |
| P_lin + P_lm | 总参数 | 493,961,216 |

**参数量验证：**

单层线性层参数量：

| 模块 | 参数量 |
|---|---|
| q_proj | HID² = 802,816 |
| k_proj | HID × KV × D = 114,688 |
| v_proj | HID × KV × D = 114,688 |
| o_proj | HID² = 802,816 |
| gate_proj | HID × INT = 4,358,144 |
| up_proj | HID × INT = 4,358,144 |
| down_proj | INT × HID = 4,358,144 |
| **单层合计** | **14,909,440** |

$$
P_{\text{lin}} = 24 \times 14{,}909{,}440 = 357{,}826{,}560
$$

$$
P_{\text{lm}} = V \times \text{HID} = 151936 \times 896 = 136{,}134{,}656
$$

---

## 二、Prefill 阶段分析

### 2.1 Prefill 的特点

Prefill 一次性处理整个 prompt，输入形状为 `(N, S)`：

- N：batch size（并发序列数）
- S：每条序列的 token 数
- 一次前向要处理 `N × S` 个 token

### 2.2 Bytes 分步计算

按“最小必需访存”计算：权重读一遍，输入激活读一遍，注意力矩阵写+读，KV 写入。

#### (a) 权重读取

$$
\text{Bytes}_{\text{weight}} = (P_{\text{lin}} + P_{\text{lm}}) \cdot \text{BYTES}
$$

- 与 N、S **无关**
- 每个前向，所有权重从 HBM 读一遍
- 权重只读不写，所以没有额外的 ×2

**Qwen2.5-0.5B：**

$$
= 493{,}961{,}216 \times 2 = 987{,}922{,}432 \text{ B} \approx 942.2 \text{ MiB}
$$

#### (b) 输入激活读取

$$
\text{Bytes}_{\text{input}} = N \cdot S \cdot \text{HID} \cdot \text{BYTES}
$$

**Qwen2.5-0.5B（N=1, S=512）：**

$$
= 1 \times 512 \times 896 \times 2 = 917{,}504 \text{ B} \approx 0.875 \text{ MiB}
$$

#### (c) 注意力矩阵写入 + 读取

$$
\text{Bytes}_{\text{attn}} = 2 \cdot N \cdot H \cdot S^2 \cdot \text{BYTES}
$$

- 每个 head 的注意力矩阵是 `S × S`
- `× 2`：写一次 + 读一次

**Qwen2.5-0.5B（N=1, S=512）：**

$$
= 2 \times 1 \times 14 \times 512^2 \times 2 = 14{,}680{,}064 \text{ B} \approx 14.0 \text{ MiB}
$$

#### (d) KV Cache 写入

$$
\text{Bytes}_{\text{KV}} = 2 \cdot N \cdot S \cdot L \cdot KV \cdot D \cdot \text{BYTES}
$$

- `KV × D`：每个 token 的 K 或 V 向量长度
- `× 2`：K 和 V 各一份
- `× L`：每一层都要写 KV

**Qwen2.5-0.5B（N=1, S=512）：**

$$
= 2 \times 1 \times 512 \times 24 \times 2 \times 64 \times 2 = 6{,}291{,}456 \text{ B} \approx 6.0 \text{ MiB}
$$

#### (e) Prefill Bytes 汇总

$$
\text{Bytes}(N, S) = \text{BYTES} \cdot (P_{\text{lin}} + P_{\text{lm}}) + \text{BYTES} \cdot N \cdot [S \cdot \text{HID} + 2HS^2 + 2L \cdot KV \cdot D \cdot S]
$$

**Qwen2.5-0.5B（N=1, S=512）：**

| 项目 | 字节数 | 占比 |
|---|---|---|
| 权重 | 987,922,432 B | 97.8% |
| 输入激活 | 917,504 B | 0.09% |
| 注意力矩阵 | 14,680,064 B | 1.45% |
| KV 写入 | 6,291,456 B | 0.62% |
| **总计** | **1,009,811,456 B** | **≈ 963 MiB** |

**结论：prefill 的访存几乎全是权重，激活和 KV 占比极小。**

---

### 2.3 FLOPs 分步计算

#### (a) 线性层（q, k, v, o, gate, up, down）

$$
\text{FLOPs}_{\text{linear}} = 2 \cdot P_{\text{lin}} \cdot N \cdot S
$$

- 每个参数一次乘加（FMA）= 2 FLOP
- `N × S`：总 token 数

**Qwen2.5-0.5B（N=1, S=512）：**

$$
= 2 \times 357{,}826{,}560 \times 512 = 366{,}414{,}397{,}440 \text{ FLOP} \approx 366.4 \text{ GFLOP}
$$

#### (b) 注意力打分 Q·Kᵀ

$$
\text{FLOPs}_{QK} = 2 \cdot N \cdot H \cdot S^2 \cdot D
$$

**Qwen2.5-0.5B（N=1, S=512）：**

$$
= 2 \times 1 \times 14 \times 512^2 \times 64 = 469{,}762{,}048 \text{ FLOP} \approx 0.47 \text{ GFLOP}
$$

#### (c) 注意力加权 A·V

$$
\text{FLOPs}_{AV} = 2 \cdot N \cdot H \cdot S^2 \cdot D
$$

与 (b) 结构相同。

**Qwen2.5-0.5B：** 469,762,048 FLOP ≈ 0.47 GFLOP

#### (d) lm_head 输出投影

$$
\text{FLOPs}_{\text{lm}} = 2 \cdot P_{\text{lm}} \cdot N \cdot S
$$

**Qwen2.5-0.5B（N=1, S=512）：**

$$
= 2 \times 136{,}134{,}656 \times 512 = 139{,}401{,}887{,}744 \text{ FLOP} \approx 139.4 \text{ GFLOP}
$$

#### (e) Prefill FLOPs 汇总

$$
\text{FLOPs}(N, S) = 2 \cdot N \cdot S \cdot (P_{\text{lin}} + P_{\text{lm}}) + 4 \cdot N \cdot H \cdot S^2 \cdot D
$$

**Qwen2.5-0.5B（N=1, S=512）：**

| 项目 | FLOPs | 占比 |
|---|---|---|
| 线性层 | 366.4 GFLOP | 72.3% |
| Q·Kᵀ | 0.47 GFLOP | 0.09% |
| A·V | 0.47 GFLOP | 0.09% |
| lm_head | 139.4 GFLOP | 27.5% |
| **总计** | **506.75 GFLOP** | **100%** |

**结论：prefill 的计算量几乎全是线性层和 lm_head，注意力只占极小比例（短序列时）。**

---

### 2.4 Prefill 运算强度与 Roofline 判定

$$
\text{AI}(N, S) = \frac{\text{FLOPs}(N, S)}{\text{Bytes}(N, S)}
$$

**Qwen2.5-0.5B（N=1, S=512）：**

$$
\text{AI} = \frac{506{,}755{,}809{,}280}{1{,}009{,}811{,}456} \approx 501.8 \text{ FLOP/Byte}
$$

对比拐点：

$$
501.8 \gg 121.4
$$

**结论：Qwen2.5-0.5B 在 prefill 阶段，batch=1 时就已经是算力受限。**

---

### 2.5 Prefill 随 N 变化的趋势

代入 S=512，令 N 变化：

$$
\text{AI}(N) = \frac{N \cdot A}{B + N \cdot C}
$$

其中：

- $A$：每个 token 的 FLOPs
- $B$：权重字节数（常数）
- $C$：每个 token 的激活 + KV 字节数

**Qwen2.5-0.5B：**

$$
\text{AI}(N) = \frac{N \cdot 5.068 \times 10^{11}}{9.879 \times 10^8 + N \cdot 2.189 \times 10^7}
$$

| N | Bytes (MB) | FLOPs (GFLOP) | AI (FLOP/Byte) | 瓶颈 |
|---|---|---|---|---|
| 1 | 1009.8 | 506.8 | 501.8 | 算力 |
| 16 | 1340.0 | 8108.1 | 6050 | 算力 |
| 256 | 6525.4 | 129,737 | 19,882 | 算力 |
| ∞ | — | — | 23,151 | 算力 |

**结论：N 越大，权重被摊薄，AI 越来越高，始终算力受限。**

---

### 2.6 Prefill 算力受限时需要的带宽

算力受限时，可达性能 = PEAK = 99.50 TFLOPS。

$$
\text{BW}_{\text{need}} = \frac{\text{PEAK}}{\text{AI}}
$$

**Qwen2.5-0.5B：**

| N | AI | 需要的带宽 | 占 V100 带宽比例 |
|---|---|---|---|
| 1 | 501.8 | 198.3 GB/s | 24% |
| 16 | 6050 | 16.4 GB/s | 2% |
| 256 | 19,882 | 5.1 GB/s | 0.6% |

**这解释了为什么 prefill 的实测带宽利用率低得离谱（0.4%）却完全正常——因为瓶颈在算力，不在带宽。**

---

## 三、Decode 阶段分析

### 3.1 Decode 的特点

Decode 阶段每次只生成 1 个新 token：

- 输入形状：`(N, 1)`
- 需要读取所有已缓存的 KV
- 上下文长度为 S（已生成的 token 数）

与 prefill 的关键区别：

| 项目 | Prefill | Decode |
|---|---|---|
| 输入 token 数 | N × S | N × 1 |
| 权重读取 | 一次 | 每步一次 |
| KV Cache | 写入 | 读取 + 写入 |
| 注意力矩阵 | S × S | 1 × S |

---

### 3.2 Decode Bytes 分步计算

#### (a) 权重读取

与 prefill 相同，每步都要读一遍所有权重：

$$
\text{Bytes}_{\text{weight}} = (P_{\text{lin}} + P_{\text{lm}}) \cdot \text{BYTES}
$$

**Qwen2.5-0.5B：** 987,922,432 B ≈ 942.2 MiB

#### (b) 输入激活读取

只有 1 个新 token：

$$
\text{Bytes}_{\text{input}} = N \cdot 1 \cdot \text{HID} \cdot \text{BYTES}
$$

**Qwen2.5-0.5B：** $1 \times 1 \times 896 \times 2 = 1{,}792 \text{ B}$

#### (c) 注意力矩阵写入 + 读取

新 token 关注 S 个缓存 token，矩阵是 `1 × S`：

$$
\text{Bytes}_{\text{attn}} = 2 \cdot N \cdot H \cdot 1 \cdot S \cdot \text{BYTES}
$$

**Qwen2.5-0.5B（S=512）：** $2 \times 1 \times 14 \times 512 \times 2 = 28{,}672 \text{ B}$

#### (d) KV Cache 读取

需要读取所有已缓存的 K 和 V：

$$
\text{Bytes}_{\text{KV\_read}} = 2 \cdot N \cdot S \cdot L \cdot KV \cdot D \cdot \text{BYTES}
$$

**Qwen2.5-0.5B（S=512）：** $2 \times 1 \times 512 \times 24 \times 2 \times 64 \times 2 = 6{,}291{,}456 \text{ B} \approx 6.0 \text{ MiB}$

#### (e) 新 KV 写入

$$
\text{Bytes}_{\text{KV\_write}} = 2 \cdot N \cdot 1 \cdot L \cdot KV \cdot D \cdot \text{BYTES}
$$

**Qwen2.5-0.5B：** $2 \times 1 \times 1 \times 24 \times 2 \times 64 \times 2 = 12{,}288 \text{ B}$

#### (f) Decode Bytes 汇总

**Qwen2.5-0.5B（N=1, S=512）：**

| 项目 | 字节数 | 占比 |
|---|---|---|
| 权重 | 987,922,432 B | 99.36% |
| 输入激活 | 1,792 B | 0.0002% |
| 注意力矩阵 | 28,672 B | 0.003% |
| KV 读取 | 6,291,456 B | 0.63% |
| 新 KV 写入 | 12,288 B | 0.001% |
| **总计** | **994,256,640 B** | **≈ 948.2 MiB** |

**结论：decode 的访存几乎全是权重，KV Cache 在短上下文时占比也极小。**

---

### 3.3 Decode FLOPs 分步计算

#### (a) 线性层

$$
\text{FLOPs}_{\text{linear}} = 2 \cdot P_{\text{lin}} \cdot N \cdot 1
$$

**Qwen2.5-0.5B：** $2 \times 357{,}826{,}560 = 715{,}653{,}120 \text{ FLOP} \approx 0.72 \text{ GFLOP}$

#### (b) 注意力打分 Q·Kᵀ

$$
\text{FLOPs}_{QK} = 2 \cdot N \cdot H \cdot 1 \cdot S \cdot D
$$

**Qwen2.5-0.5B（S=512）：** $2 \times 1 \times 14 \times 512 \times 64 = 917{,}504 \text{ FLOP}$

#### (c) 注意力加权 A·V

与 (b) 相同。

**Qwen2.5-0.5B：** 917,504 FLOP

#### (d) lm_head

$$
\text{FLOPs}_{\text{lm}} = 2 \cdot P_{\text{lm}} \cdot N \cdot 1
$$

**Qwen2.5-0.5B：** $2 \times 136{,}134{,}656 = 272{,}269{,}312 \text{ FLOP} \approx 0.27 \text{ GFLOP}$

#### (e) Decode FLOPs 汇总

**Qwen2.5-0.5B（N=1, S=512）：**

| 项目 | FLOPs | 占比 |
|---|---|---|
| 线性层 | 0.716 GFLOP | 72.3% |
| Q·Kᵀ | 0.0009 GFLOP | 0.09% |
| A·V | 0.0009 GFLOP | 0.09% |
| lm_head | 0.272 GFLOP | 27.5% |
| **总计** | **0.99 GFLOP** | **100%** |

---

### 3.4 Decode 运算强度与 Roofline 判定

**Qwen2.5-0.5B（N=1, S=512）：**

$$
\text{AI} = \frac{989{,}757{,}440}{994{,}256{,}640} \approx 0.995 \text{ FLOP/Byte}
$$

对比拐点：

$$
0.995 \ll 121.4
$$

**结论：Qwen2.5-0.5B 在 decode 阶段是极度带宽受限。**

---

### 3.5 Decode 随上下文长度变化的趋势

在 decode 中，上下文长度 S 会随生成而增长。分析 S → ∞ 时的极限：

$$
\text{AI}_{\text{decode}} \to \frac{4HD}{2L \cdot KV \cdot D} = \frac{2H}{L \cdot KV}
$$

**Qwen2.5-0.5B：**

$$
\frac{2 \times 14}{24 \times 2} = 0.583 \text{ FLOP/Byte}
$$

**结论：decode 的 AI 随着上下文增长反而下降，始终远低于拐点，带宽受限的性质不会改变。**

---

## 四、两阶段对比与结论

### 4.1 Prefill vs Decode 对比

| 维度 | Prefill | Decode |
|---|---|---|
| 输入 token 数 | N × S | N × 1 |
| 主导访存 | 权重 | 权重 |
| 主导计算 | 线性层 + lm_head | 线性层 + lm_head |
| 运算强度 | 501.8 | 0.995 |
| 瓶颈 | **算力受限** | **带宽受限** |
| 带宽利用率 | 极低（正常） | 接近峰值 |
| 算力利用率 | 接近峰值 | 极低（正常） |

### 4.2 关键洞察

1. **Prefill 和 Decode 的瓶颈完全相反：**
   - Prefill：算力受限，带宽大量闲置
   - Decode：带宽受限，算力大量闲置

2. **权重的双重身份：**
   - Prefill 中，权重被 N × S 个 token 摊薄，AI 很高
   - Decode 中，权重每步只服务 1 个 token，AI 极低

3. **KV Cache 的影响：**
   - 短上下文时，KV Cache 占比极小
   - 长上下文时，KV Cache 逐渐显著，但权重仍占主导

### 4.3 优化方向

| 阶段 | 瓶颈 | 优化方向 |
|---|---|---|
| Prefill | 算力 | 提高 Tensor Core 利用率、增大 batch、使用 FP8/INT8 |
| Decode | 带宽 | 权重量化（INT4/INT8）、KV Cache 量化、投机解码、连续批处理 |

### 4.4 一句话总结

> 在 V100 上，Qwen2.5-0.5B 的 **prefill 是算力受限，decode 是带宽受限**。Prefill 的访存 97% 以上是权重，计算 99% 以上是线性层和 lm_head，运算强度 501.8，远超拐点 121.4；Decode 的访存 99% 以上是权重，运算强度 0.995，远低于拐点。两个阶段对硬件资源的需求完全相反，这解释了为什么 prefill 的带宽利用率极低、decode 的算力利用率极低，也解释了为什么推理优化需要针对两个阶段分别设计策略。