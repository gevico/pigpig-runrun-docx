---
title: ai编辑器解析-以tensetorrent为例
description: ai编辑器解析-以tensetorrent为例
published: true
date: 2026-09-25T05:51:40.729Z
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