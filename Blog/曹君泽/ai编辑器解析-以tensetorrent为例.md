---
title: ai编辑器解析-以tensetorrent为例
description: ai编辑器解析-以tensetorrent为例
published: true
date: 2026-09-25T06:00:32.641Z
tags: 
editor: markdown
dateCreated: 2026-09-25T05:51:40.729Z
---

# ai编辑器解析-以tensetorrent为例

## 编译流程
```
Your Model (PyTorch / JAX / ONNX)
        │
        ▼
┌──────────────────────┐
│   Frontend Layer     │
│  ┌────────────────┐  │
│  │   TT-XLA       │  │  ← PyTorch (via torch_xla) and JAX models
│  │   (PJRT)       │  │     Produces StableHLO graphs
│  └────────────────┘  │
│  ┌────────────────┐  │
│  │ TT-Forge-ONNX  │  │  ← ONNX, TensorFlow, PaddlePaddle
│  │   (TVM-based)  │  │     Produces TTIR directly
│  └────────────────┘  │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│   TT-MLIR Compiler   │
│                      │
│  StableHLO → TTIR   │  ← Common intermediate representation
│  TTIR → Graph Passes │  ← Fusing, layout transforms, sharding
│  TTIR → TTNN-IR      │  ← Maps to TTNN library ops
│  TTIR → TTKernel-IR  │  ← Custom kernels (advanced)
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│   TT-Metalium        │
│   (TTNN + TTMetal)   │  ← Runtime: dispatches ops to hardware
└──────────┬───────────┘
           │
           ▼
   Wormhole / Blackhole
       (your card)
```

## 逐阶段分析

### pytorch输入
这里以 Qwen2.5-0.5B-Instruct 简略结构 为例
```
class QwenDecoderLayer(nn.Module):
    def __init__(self, cfg):
        self.input_layernorm = nn.RMSNorm(cfg.hidden_size, eps=1e-6)
        self.q_proj = nn.Linear(896, 896, bias=True)
        self.k_proj = nn.Linear(896, 128, bias=True)   # GQA: 2 个 KV head
        self.v_proj = nn.Linear(896, 128, bias=True)
        self.o_proj = nn.Linear(896, 896, bias=False)
        self.gate_proj = nn.Linear(896, 4864, bias=False)
        self.up_proj   = nn.Linear(896, 4864, bias=False)
        self.down_proj = nn.Linear(4864, 896, bias=False)

    def forward(self, x):
        h = x + self.attn(self.input_layernorm(x))
        return h + self.mlp(self.post_attention_layernorm(h))
```