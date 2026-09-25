---
title: qwen2-5解析
description: qwen2-5解析
published: true
date: 2026-09-25T08:20:58.448Z
tags: 
editor: markdown
dateCreated: 2026-09-25T06:38:42.808Z
---

# Qwen2.5-0.5B-Instruct 解析

## 一、简略代码

```python
#!/usr/bin/env python3

from __future__ import annotations

import argparse
import json
import os
import sys
from dataclasses import dataclass
from pathlib import Path

import torch
import torch.nn as nn
import torch.nn.functional as F


MODEL_DIR = Path(os.environ.get("INFER_MODELS", "/data/cgbc/models")) / "Qwen2.5-0.5B-Instruct"


@dataclass
class Qwen2Config:
    hidden_size: int = 896
    intermediate_size: int = 4864
    num_attention_heads: int = 14
    num_key_value_heads: int = 2
    head_dim: int = 64
    rms_norm_eps: float = 1e-6
    rope_theta: float = 1_000_000.0
    hidden_act: str = "silu"
    max_position_embeddings: int = 32768

    @classmethod
    def from_pretrained(cls, model_dir: Path) -> "Qwen2Config":
        raw = json.loads((Path(model_dir) / "config.json").read_text())
        heads = raw["num_attention_heads"]
        return cls(
            hidden_size=raw["hidden_size"],
            intermediate_size=raw["intermediate_size"],
            num_attention_heads=heads,
            num_key_value_heads=raw.get("num_key_value_heads", heads),
            head_dim=raw.get("head_dim") or raw["hidden_size"] // heads,
            rms_norm_eps=raw["rms_norm_eps"],
            rope_theta=raw["rope_theta"],
            hidden_act=raw["hidden_act"],
            max_position_embeddings=raw["max_position_embeddings"],
        )


class Qwen2RMSNorm(nn.Module):
    def __init__(self, hidden_size: int, eps: float = 1e-6) -> None:
        super().__init__()
        self.weight = nn.Parameter(torch.ones(hidden_size))
        self.variance_epsilon = eps

    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        input_dtype = hidden_states.dtype
        hidden_states = hidden_states.to(torch.float32)
        variance = hidden_states.pow(2).mean(-1, keepdim=True)
        hidden_states = hidden_states * torch.rsqrt(variance + self.variance_epsilon)
        return self.weight * hidden_states.to(input_dtype)

    def extra_repr(self) -> str:
        return f"{tuple(self.weight.shape)}, eps={self.variance_epsilon}"


class Qwen2RotaryEmbedding(nn.Module):
    def __init__(self, config: Qwen2Config) -> None:
        super().__init__()
        self.config = config
        dim = config.head_dim
        inv_freq = 1.0 / (config.rope_theta ** (torch.arange(0, dim, 2, dtype=torch.float) / dim))
        self.register_buffer("inv_freq", inv_freq, persistent=False)

    @torch.no_grad()
    def forward(self, x: torch.Tensor, position_ids: torch.Tensor):
        inv_freq_expanded = self.inv_freq[None, :, None].float().expand(position_ids.shape[0], -1, 1)
        position_ids_expanded = position_ids[:, None, :].float()

        device_type = x.device.type if isinstance(x.device.type, str) and x.device.type != "mps" else "cpu"
        with torch.autocast(device_type=device_type, enabled=False):
            freqs = (inv_freq_expanded.float() @ position_ids_expanded.float()).transpose(1, 2)
            emb = torch.cat((freqs, freqs), dim=-1)
            cos = emb.cos()
            sin = emb.sin()

        return cos.to(dtype=x.dtype), sin.to(dtype=x.dtype)


def rotate_half(x: torch.Tensor) -> torch.Tensor:
    x1 = x[..., : x.shape[-1] // 2]
    x2 = x[..., x.shape[-1] // 2 :]
    return torch.cat((-x2, x1), dim=-1)


def apply_rotary_pos_emb(q, k, cos, sin, unsqueeze_dim: int = 1):
    cos = cos.unsqueeze(unsqueeze_dim)
    sin = sin.unsqueeze(unsqueeze_dim)
    q_embed = (q * cos) + (rotate_half(q) * sin)
    k_embed = (k * cos) + (rotate_half(k) * sin)
    return q_embed, k_embed


def repeat_kv(hidden_states: torch.Tensor, n_rep: int) -> torch.Tensor:
    batch, num_key_value_heads, slen, head_dim = hidden_states.shape
    if n_rep == 1:
        return hidden_states
    hidden_states = hidden_states[:, :, None, :, :].expand(batch, num_key_value_heads, n_rep, slen, head_dim)
    return hidden_states.reshape(batch, num_key_value_heads * n_rep, slen, head_dim)


class Qwen2Attention(nn.Module):
    def __init__(self, config: Qwen2Config) -> None:
        super().__init__()
        self.config = config
        self.head_dim = config.head_dim
        self.num_key_value_groups = config.num_attention_heads // config.num_key_value_heads
        self.scaling = self.head_dim**-0.5
        self.is_causal = True

        self.q_proj = nn.Linear(config.hidden_size, config.num_attention_heads * self.head_dim, bias=True)
        self.k_proj = nn.Linear(config.hidden_size, config.num_key_value_heads * self.head_dim, bias=True)
        self.v_proj = nn.Linear(config.hidden_size, config.num_key_value_heads * self.head_dim, bias=True)
        self.o_proj = nn.Linear(config.num_attention_heads * self.head_dim, config.hidden_size, bias=False)

    def forward(
        self,
        hidden_states: torch.Tensor,
        position_embeddings: tuple[torch.Tensor, torch.Tensor],
        attention_mask: torch.Tensor | None = None,
    ) -> torch.Tensor:
        input_shape = hidden_states.shape[:-1]
        hidden_shape = (*input_shape, -1, self.head_dim)

        query_states = self.q_proj(hidden_states).view(hidden_shape).transpose(1, 2)
        key_states = self.k_proj(hidden_states).view(hidden_shape).transpose(1, 2)
        value_states = self.v_proj(hidden_states).view(hidden_shape).transpose(1, 2)

        cos, sin = position_embeddings
        query_states, key_states = apply_rotary_pos_emb(query_states, key_states, cos, sin)

        key_states = repeat_kv(key_states, self.num_key_value_groups)
        value_states = repeat_kv(value_states, self.num_key_value_groups)

        attn_weights = torch.matmul(query_states, key_states.transpose(2, 3)) * self.scaling
        if attention_mask is not None:
            attn_weights = attn_weights + attention_mask

        attn_weights = F.softmax(attn_weights, dim=-1, dtype=torch.float32).to(query_states.dtype)
        attn_output = torch.matmul(attn_weights, value_states)

        attn_output = attn_output.transpose(1, 2).contiguous()
        attn_output = attn_output.reshape(*input_shape, -1).contiguous()
        return self.o_proj(attn_output)


_ACT2FN = {
    "gelu": F.gelu,
    "relu": F.relu,
    "silu": F.silu,
    "gelu_pytorch_tanh": lambda x: F.gelu(x, approximate="tanh"),
}


class Qwen2MLP(nn.Module):
    def __init__(self, config: Qwen2Config) -> None:
        super().__init__()
        self.gate_proj = nn.Linear(config.hidden_size, config.intermediate_size, bias=False)
        self.up_proj = nn.Linear(config.hidden_size, config.intermediate_size, bias=False)
        self.down_proj = nn.Linear(config.intermediate_size, config.hidden_size, bias=False)
        self.act_fn = _ACT2FN[config.hidden_act]

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        return self.down_proj(self.act_fn(self.gate_proj(x)) * self.up_proj(x))


class Qwen2DecoderLayer(nn.Module):
    def __init__(self, config: Qwen2Config) -> None:
        super().__init__()
        self.config = config
        self.hidden_size = config.hidden_size
        self.self_attn = Qwen2Attention(config)
        self.mlp = Qwen2MLP(config)
        self.input_layernorm = Qwen2RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
        self.post_attention_layernorm = Qwen2RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
        self.rope = Qwen2RotaryEmbedding(config)

    def forward(
        self,
        hidden_states: torch.Tensor,
        position_embeddings: tuple[torch.Tensor, torch.Tensor] | None = None,
        position_ids: torch.LongTensor | None = None,
        attention_mask: torch.Tensor | None = None,
    ) -> torch.Tensor:
        if position_embeddings is None:
            if position_ids is None:
                seq = hidden_states.shape[1]
                position_ids = torch.arange(seq, device=hidden_states.device).unsqueeze(0)
            position_embeddings = self.rope(hidden_states, position_ids)

        if attention_mask is None:
            attention_mask = build_causal_mask(hidden_states.shape[1], hidden_states.dtype, hidden_states.device)

        residual = hidden_states
        hidden_states = self.input_layernorm(hidden_states)
        hidden_states = self.self_attn(
            hidden_states=hidden_states,
            position_embeddings=position_embeddings,
            attention_mask=attention_mask,
        )
        hidden_states = residual + hidden_states

        residual = hidden_states
        hidden_states = self.post_attention_layernorm(hidden_states)
        hidden_states = self.mlp(hidden_states)
        hidden_states = residual + hidden_states
        return hidden_states


def build_causal_mask(seq_len: int, dtype: torch.dtype, device) -> torch.Tensor:
    min_dtype = torch.finfo(dtype).min
    mask = torch.full((1, 1, seq_len, seq_len), min_dtype, dtype=dtype, device=device)
    return torch.triu(mask, diagonal=1)


def load_layer_from_safetensors(layer: Qwen2DecoderLayer, model_dir: Path, layer_idx: int) -> None:
    from safetensors.torch import load_file

    ckpt = model_dir / "model.safetensors"
    sd = load_file(str(ckpt), device="cpu")

    prefix = f"model.layers.{layer_idx}."
    sub = {k[len(prefix):]: v for k, v in sd.items() if k.startswith(prefix)}
    if not sub:
        raise KeyError(f"safetensors 中没有前缀 {prefix!r} 的权重，检查 layer_idx")

    missing, unexpected = layer.load_state_dict(sub, strict=False)
    if missing:
        raise RuntimeError(f"缺失权重: {missing}")
    if unexpected:
        raise RuntimeError(f"多余权重: {unexpected}")


def max_abs_err(a: torch.Tensor, b: torch.Tensor) -> float:
    return (a - b).abs().max().item()


def verify(layer_idx: int, seq_len: int, prompt: str, tol: float = 1e-6) -> int:
    from transformers import AutoModelForCausalLM, AutoTokenizer

    torch.set_grad_enabled(False)
    torch.set_num_threads(int(os.environ.get("OMP_NUM_THREADS", "32")))

    cfg = Qwen2Config.from_pretrained(MODEL_DIR)

    our_layer = Qwen2DecoderLayer(cfg).eval()
    load_layer_from_safetensors(our_layer, MODEL_DIR, layer_idx)

    hf = AutoModelForCausalLM.from_pretrained(
        str(MODEL_DIR), dtype=torch.float32, attn_implementation="eager"
    ).eval()

    tok = AutoTokenizer.from_pretrained(str(MODEL_DIR))
    ids = tok(prompt, return_tensors="pt").input_ids[:, :seq_len]
    S = ids.shape[1]

    out = hf(ids, output_hidden_states=True)
    x = out.hidden_states[layer_idx]

    position_ids = torch.arange(S).unsqueeze(0)
    mask = build_causal_mask(S, x.dtype, x.device)

    hf_cos, hf_sin = hf.model.rotary_emb(x, position_ids)
    want = hf.model.layers[layer_idx](
        x, attention_mask=mask, position_embeddings=(hf_cos, hf_sin), position_ids=position_ids
    )

    got = our_layer(x, position_ids=position_ids, attention_mask=mask)
    worst = max_abs_err(got, want)

    hf_layer = hf.model.layers[layer_idx]

    ln1 = our_layer.input_layernorm(x)
    hf_ln1 = hf_layer.input_layernorm(x)
    mid_err = max_abs_err(ln1, hf_ln1)

    cos_, sin_ = our_layer.rope(x, position_ids)
    mid_err = max(mid_err, max_abs_err(cos_, hf_cos), max_abs_err(sin_, hf_sin))

    attn = our_layer.self_attn(ln1, (cos_, sin_), mask)
    hf_attn_out, _ = hf_layer.self_attn(
        hidden_states=hf_ln1,
        position_embeddings=(hf_cos, hf_sin),
        attention_mask=mask,
    )
    mid_err = max(mid_err, max_abs_err(attn, hf_attn_out))

    mlp_in = our_layer.post_attention_layernorm(x + attn)
    hf_mlp_in = hf_layer.post_attention_layernorm(x + hf_attn_out)
    mid_err = max(mid_err, max_abs_err(mlp_in, hf_mlp_in))

    mlp_out = our_layer.mlp(mlp_in)
    hf_mlp_out = hf_layer.mlp(hf_mlp_in)
    mid_err = max(mid_err, max_abs_err(mlp_out, hf_mlp_out))

    worst = max(worst, mid_err)
    return 0 if worst < tol else 1


def main() -> int:
    ap = argparse.ArgumentParser(description="Qwen2.5 Decoder Layer + HF 对拍")
    ap.add_argument("--layer", type=int, default=0, help="验证第几层 (0-23)")
    ap.add_argument("--seq", type=int, default=21, help="序列长度")
    ap.add_argument("--prompt", default="请用简洁的中文解释一下什么是 KV Cache，它为什么能加速大模型的推理过程？")
    args = ap.parse_args()
    return verify(args.layer, args.seq, args.prompt)


if __name__ == "__main__":
    sys.exit(main())
```

## 二、对应架构图

### 2.1 整体架构图

```
                          input_ids  (1, S)  int64
                                    │
                                    ▼
            ┌───────────────────────────────────────────┐
            │  embed_tokens                             │
            │  Embedding(151936 → 896)                  │  136.1 M   27.6%
            └──────────────────────┬────────────────────┘
                                   │  (1, S, 896)
                                   ▼
            ╔═══════════════════════════════════════════╗
            ║                                           ║
            ║           Decoder Layer   ×  24           ║  357.9 M   72.4%
            ║                                           ║
            ╚═══════════════════════┬═══════════════════╝
                                   │  (1, S, 896)
                                   ▼
            ┌───────────────────────────────────────────┐
            │  model.norm                               │
            │  RMSNorm(896, eps=1e-6)                   │  896
            └──────────────────────┬────────────────────┘
                                   │  (1, S, 896)
                                   ▼
            ┌───────────────────────────────────────────┐
            │  lm_head                                  │
            │  Linear(896 → 151936)                     │  与 embed_tokens 共享
            │  ← 权重绑定 (tied)，不额外占参数           │
            └──────────────────────┬────────────────────┘
                                   │
                                   ▼
                        logits  (1, S, 151936)

```

### 2.2 decode layer

```
                    x  (1, S, 896)
                    │
        ┌───────────┴──────────────────────────────────────┐
        │                                                  │  残差 ①
        ▼                                                  │
  ┌─────────────────────────┐                              │
  │   input_layernorm        │  RMSNorm(896, eps=1e-6)     │
  └────────────┬─────────────┘                             │
               │  (1, S, 896)                              │
               ▼                                           │
  ╔═════════════════════════════════════════════════════╗  │
  ║                     self_attn                        ║  │
  ║                                                      ║  │
  ║   ┌──────────┐                                       ║  │
  ║   │  q_proj  │ 896→896   →  (1, S, 14, 64)          ║  │
  ║   ├──────────┤                                       ║  │
  ║   │  k_proj  │ 896→128   →  (1, S,  2, 64)  ─┐       ║  │
  ║   ├──────────┤                               │       ║  │
  ║   │  v_proj  │ 896→128   →  (1, S,  2, 64)  ─┤RoPE   ║  │
  ║   └──────────┘                               │ θ=1e6 ║  │
  ║                                              ▼       ║  │
  ║                          repeat_kv  × 7              ║  │
  ║                             (1, 2, S, 64)            ║  │
  ║                                  → (1, 14, S, 64)    ║  │
  ║                                              │       ║  │
  ║                                              ▼       ║  │
  ║           SDPA (causal)                              ║  │
  ║           softmax( q·kᵀ · (1/√64) + mask )           ║  │
  ║                          │                           ║  │
  ║                          ▼                           ║  │
  ║                 合并多头 → (1, S, 896)                ║  │
  ║                          │                           ║  │
  ║                          ▼                           ║  │
  ║   ┌──────────┐                                       ║  │
  ║   │  o_proj  │ 896→896   (无 bias)                   ║  │
  ║   └──────────┘                                       ║  │
  ╚══════════════════════════┬══════════════════════════╝  │
                             │                             │
                             ▼                             │
                         ┌───────┐                         │
                         │  ＋   │ ◄───────────────────────┘
                         └───┬───┘
                             │  h  (1, S, 896)
        ┌────────────────────┴────────────────────────────┐
        │                                                 │  残差 ②
        ▼                                                 │
  ┌─────────────────────────────┐                         │
  │  post_attention_layernorm   │  RMSNorm(896, eps=1e-6) │
  └──────────────┬──────────────┘                         │
                 │  (1, S, 896)                            │
                 ▼                                        │
  ╔═════════════════════════════════════════════════╗     │
  ║                  mlp  (SwiGLU)                  ║     │
  ║                                                 ║     │
  ║   gate_proj 896→4864 ──► SiLU ──┐               ║     │
  ║                                 ├──► ×  ──► down_proj 4864→896
  ║   up_proj   896→4864 ───────────┘               ║     │
  ║                                                 ║     │
  ╚═════════════════════════┬═══════════════════════╝     │
                            │  (1, S, 896)                │
                            ▼                             │
                        ┌───────┐                         │
                        │  ＋   │ ◄───────────────────────┘
                        └───┬───┘
                            │
                            ▼
                     out  (1, S, 896)
```

## 三、逐组件解析

### 3.1 Embedding

经过分词器
原文      : "我讨厌调研"    5 个汉字
input_ids : （1，[35946, 108749, 104218])    3 个 token

Embedding 层本质为查词表。那么经过分词器切分之后的 token ID 进入 Embedding 层之后，会查找token ID 与 896 维向量对应的词表，把对应的 token ID 转化成 896 维的向量，对应结果为
```python
[
  [
    W_embed[35946],     # 长度 896
    W_embed[108749],    # 长度 896
    W_embed[104218]     # 长度 896
  ]
]
```

### 3.2 进入decode layer的input laynorm前

会准备一下三个量 
position_ids
causal_mask
position_embeddings
这三个变量并不会进入到 input laynorm 层，而是直接输入到 self-attention 层

#### 3.2.1 position_ids

position_ids：即当前 KV cache 和当前 token 对应的位置。这里要分“有 KV cache”和“无 KV cache”两种情况：

	1. 无 KV cache：
	它只需要判断一次输入的 prompt 的位置。例如“我讨厌调研”，其中“调研”是在第三个，那么这里的 position ID 数值就是 3。
	2. 有 KV cache：
	生成第 n 个 token 的时候，前面已经有 past_seen_tokens 个 token，那么 position ID 就要等于past_seen_tokens （已处理的 token 数）加上当前 prompt 中属于第几个 token 的数值

#### 3.2.2 causal_mask

causal_mask：本质上是一个二维矩阵，其中上三角部分全都为负无穷大，其余全都为 0。

目的呢，是在 softmax 之前，把未来 token 的注意力分数压成极小值，让未来 token 的 softmax 权重接近于 0
```
[[  0, -inf, -inf, -inf, -inf],
 [  0,   0, -inf, -inf, -inf],
 [  0,   0,   0, -inf, -inf],
 [  0,   0,   0,   0, -inf],
 [  0,   0,   0,   0,   0]]
```

#### 3.2.3 position_embeddings:

`position_embeddings` 是 **RoPE（Rotary Position Embedding，旋转位置编码）** 预计算出来的两个张量：

```python
position_embeddings = (cos, sin)
```

- `cos`：形状 `(1, S, 64)`，每个位置、每个维度上的余弦值
- `sin`：形状 `(1, S, 64)`，每个位置、每个维度上的正弦值

它们的作用是：**在注意力计算之前，对 Query 和 Key 向量施加一个与位置相关的旋转。**

具体操作：

$$
q_{\text{rot}} = q \cdot \cos + \text{rotate\_half}(q) \cdot \sin
$$

$$
k_{\text{rot}} = k \cdot \cos + \text{rotate\_half}(k) \cdot \sin
$$

本质上是把 Q/K 的每一对维度 $(x, y)$ 看作二维平面上的向量，按位置 $p$ 旋转一个角度 $\theta_{p,i}$：

$$
\begin{bmatrix} x' \\ y' \end{bmatrix}
=
\begin{bmatrix} \cos\theta & -\sin\theta \\ \sin\theta & \cos\theta \end{bmatrix}
\begin{bmatrix} x \\ y \end{bmatrix}
$$

其中：

$$
\theta_{p,i} = p \cdot \Theta^{-\frac{2i}{d}}
$$

- $p$：token 的位置
- $i$：第几对维度
- $\Theta = 10^6$：`rope_theta`
- $d = 64$：`head_dim`

解决了什么问题：
原始版本的 Attention，如果对其输入打乱序列顺序，输出会表现为置换等变。

比如说，预期的输出是“我讨厌调研”，那么如果我的输入序列是反过来的，这个时候就会变成“调研讨厌我”。这可以证明，最原始版本的 Attention 机制并不会因为输入 token 的序列变化而改变其输出，只是会改变其输出的顺序。这在自然语言处理中显然是不可接受的

因此，回头来看 RoPE 位置编码，它就是为每一个位置上附加了一个，让注意力分数依赖相对位置，让模型训练过程当中，让其学习到自然语言的顺序对于语义的影响。

在这之前，其实也有几个同样的解决方法，包括：
• 绝对位置编码
• 相对位置编码

## RMSNorm
对每一个 token 的 896 维向量做均方根归一化，

RMSNorm 的计算公式是：

$$
\text{RMSNorm}(x) = \frac{x}{\sqrt{\text{mean}(x^2) + \epsilon}} \cdot w
$$
