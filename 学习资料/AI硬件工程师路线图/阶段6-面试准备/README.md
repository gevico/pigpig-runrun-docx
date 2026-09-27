---
title: 阶段 6 — 面试准备
description: 阶段 6 — 面试准备
published: true
date: 2026-09-27T09:12:30.000Z
tags: 学习资料
editor: markdown
dateCreated: 2026-09-27T09:12:30.000Z
---

# 阶段 6 — 面试准备

<div class="course-identity" markdown="1">
<div class="course-identity__icon">INT</div>
<div markdown="1">
<p class="course-identity__eyebrow">阶段 6 · 职业进阶</p>
<p class="course-identity__title">面向 AI 硬件、推理与系统工程师的岗位针对性面试准备 —— 资深级别的真实问答，而非零碎知识点。</p>
<p class="course-identity__meta">产物：扎根实战经验、自信而具体的回答 · 衡量标准：能否为取舍辩护，而不只是复述定义</p>
</div>
</div>

> *资深级别下「通过」与「录用」的差别，不在于知不知道 FlashAttention 是什么 —— 而在于能否讲清它搬走了哪个瓶颈、做出了什么取舍，以及真正动手实现时会撞到什么。*

本阶段不是一套抽认卡题库。这里的每个回答都以真正把系统交付上线的人的水准写成：具体的数字、真实的失效模式、经得起推敲的取舍，以及只有亲手做过才会有的洞察。这正是 staff/资深面试所期待的。

---

## 覆盖的岗位

| # | 岗位 | 重点 | 状态 |
|---|------|-------|--------|
| [1](/学习资料/AI硬件工程师路线图/阶段6-面试准备/01-MLSys工程师/README) | **MLSys 工程师** | 推理系统、attention kernel、KV cache、投机 decode（逐 token 生成阶段）、TRT-LLM 架构集成 | ✅ 进行中 |

更多岗位即将加入：边缘 AI 工程师、CUDA Kernel 工程师、AI 芯片架构师、机器人 AI 工程师。

---

## 如何使用本材料

**不要背 —— 要重建。** 读一遍答案，合上，然后用自己的话讲回来。目标是面试时能从第一性原理推导出答案，而不是背诵。

**校准级别。** 每个问答都按资深/staff 级别写成。如果你面的是 L4/E4，不需要每个细节 —— 但每个回答都需要第一性原理的推理和一个具体的取舍。

**用你自己的数字扩展。** 答案中引用了具体测量值（例如 prefill（首字前的整段计算）152→620 tok/s，双缓冲下 1289ms→348ms）。把这些换成你自己项目的数字。面试官能分辨出泛泛而谈的回答和扎根于真实工作的回答。

**先广度，后深度。** 通读所有问题，校准面试官关心哪些主题，然后在你最熟的两三个上深入。其余的诚实以对。

---

## 按公司类型划分的面试形式

| 公司类型 | 典型形式 | 考察点 |
|---|---|---|
| 推理初创公司（Groq、Cerebras、Together、Fireworks） | 深度系统设计，1 到 2 轮编码，不刷 LC | 瓶颈分析、kernel 编写、延迟计算 |
| NVIDIA（NIM、TRT-LLM、cuBLAS） | 设计 + 编码 + 硬件题混合 | PTX、occupancy、插件 API、TRT engine 生命周期 |
| Google（XLA、TPU、Google AI） | 偏设计，少量编码，ML 广度 | 分布式系统、编译器 IR、量化理论 |
| Meta（PyTorch、AITER、infra） | 编码 + 设计 + infra | Python 扩展 API、CUDA/Triton、分布式训练 |
| 边缘 / Jetson（NVIDIA EGX、Qualcomm、Apple） | 端到端系统设计 | 内存预算、延迟预算、runtime 可移植性 |
| 大厂 ML infra | LC 编码 + 广谱系统设计 | 硬件相关更少，分布式、调度、成本更多 |

---

*从这里开始：[MLSys 工程师面试准备](/学习资料/AI硬件工程师路线图/阶段6-面试准备/01-MLSys工程师/README)*


<details>
<summary>English original</summary>

**Phase 6 — Interview Preparation**

<div class="course-identity" markdown="1">
<div class="course-identity__icon">INT</div>
<div markdown="1">
<p class="course-identity__eyebrow">Phase 6 · Career Progression</p>
<p class="course-identity__title">Role-specific interview preparation for AI hardware, inference, and systems engineers — real Q&A at senior level, not trivia.</p>
<p class="course-identity__meta">Artifact: confident, specific answers grounded in hands-on experience · Measure: can you defend the tradeoffs, not just recall the definition</p>
</div>
</div>

> *The difference between a pass and a hire at senior level is not knowing what FlashAttention is — it's explaining which bottleneck it moves, which tradeoff it makes, and what you hit when you actually implemented it.*

This phase is not a flashcard bank. Each answer here is written at the level of someone who has shipped the system: specific numbers, real failure modes, tradeoffs defended, and the insight you only get by doing the work. That is exactly what a staff/senior interview expects.

---

**Roles Covered**

| # | Role | Focus | Status |
|---|------|-------|--------|
| [1](/学习资料/AI硬件工程师路线图/阶段6-面试准备/01-MLSys工程师/README) | **MLSys Engineer** | Inference systems, attention kernels, KV cache, speculative decode, TRT-LLM architecture integration | ✅ Active |

More roles coming: Edge AI Engineer, CUDA Kernel Engineer, AI Chip Architect, Robotics AI Engineer.

---

**How to use this material**

**Don't memorize — reconstruct.** Read an answer once, close it, and say it back in your own words. The goal is to be able to derive the answer from first principles during the interview, not recite it.

**Calibrate level.** Each Q&A is written at senior/staff level. If you're interviewing at L4/E4, you don't need every detail — but you do need the first-principles reasoning and one concrete tradeoff per answer.

**Extend with your own numbers.** The answers reference specific measurements (e.g., 152→620 tok/s prefill, 1289ms→348ms with double-buffer). Swap those for your own project numbers. Interviewers can tell generic answers from ones grounded in real work.

**Breadth first, then depth.** Read all questions to calibrate what topics an interviewer cares about, then go deep on the two or three you know best. Be honest about the others.

---

**Interview format by company type**

| Company type | Typical format | What they probe |
|---|---|---|
| Inference startup (Groq, Cerebras, Together, Fireworks) | Deep system design, 1 or 2 coding, no LC grind | Bottleneck analysis, kernel writing, latency math |
| NVIDIA (NIM, TRT-LLM, cuBLAS) | Mix of design + coding + HW questions | PTX, occupancy, plugin APIs, TRT engine lifecycle |
| Google (XLA, TPU, Google AI) | Design heavy, some coding, ML breadth | Distributed systems, compiler IR, quantization theory |
| Meta (PyTorch, AITER, infra) | Coding + design + infra | Python extension APIs, CUDA/Triton, distributed training |
| Edge / Jetson (NVIDIA EGX, Qualcomm, Apple) | End-to-end system design | Memory budget, latency budget, runtime portability |
| Big tech ML infra | LC coding + broad system design | Less HW-specific, more distributed, scheduling, cost |

---

*Start here: [MLSys Engineer Interview Prep](/学习资料/AI硬件工程师路线图/阶段6-面试准备/01-MLSys工程师/README)*

</details>

---

> 原文：[`Phase 6 - Interview Preparation/README.md`](https://github.com/ai-hpc/ai-hardware-engineer-roadmap/blob/main/Phase%206%20-%20Interview%20Preparation/README.md)（上游仓库 ai-hpc/ai-hardware-engineer-roadmap，MIT License）。本页为中文翻译，术语保留英文原词；与原文不一致时以原文为准。
