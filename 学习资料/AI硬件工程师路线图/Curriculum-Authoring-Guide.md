---
title: 课程编写指南
description: 课程编写指南
published: true
date: 2026-09-27T09:17:33.000Z
tags: 学习资料
editor: markdown
dateCreated: 2026-09-27T09:17:33.000Z
---

# 课程编写指南

> *如何在不稀释项目的前提下新增或扩展路线图内容。*

只有当每个新页面都做好三件事时，本仓库才处于最佳状态：

- 讲授真实的系统或硬件概念
- 产出可见的工程产物
- 与 8 层技术栈及下游角色清晰衔接

新增模块、深入专题、实验或项目页面时，请参照本指南。

---

## 核心规则

### 1. 坚持硬件优先

这不是通用的 AI 课程。

新内容至少应回答以下问题之一：

- 硬件需要运行什么工作负载？
- 软件如何映射到硬件资源上？
- 性能、功耗、内存或部署会受到什么影响？
- 这讲清了 AI 芯片技术栈中的哪个角色？

如果某个主题有意思，却无助于强化硬件直觉，那它大概不属于这里。

### 2. 优先动手构建型内容

避免只列书单或只做概念总结的页面。

每篇有分量的指南都应推动学习者走向：

- 代码
- 性能剖析
- 调试
- 测量
- 部署
- 架构推演

### 3. 产出产物，而不只是笔记

每个模块都应给出一个或多个可供其他工程师检查的产物：

- benchmark 表
- 性能剖析 trace
- 板级 bring-up（上电点亮/调通）检查清单
- device-tree 补丁
- CUDA kernel
- MLIR pass 演示
- FPGA 时序报告
- 系统图

### 4. 明确标注角色映射

每篇指南都应让人能轻松回答：

- 这属于哪一层？
- 这有助于胜任哪个角色？
- 下一步该进入哪个阶段或方向？

### 5. 定义「完成」的含义

每个有分量的课程页面都应让完成情况可观测。学习者不应以「我读过了」作为完成模块的标志，而应以证据完成：

- 能跑起来的代码
- 构建日志
- 性能分析器 trace
- benchmark 表
- 调试抓取记录
- 设计报告
- 部署检查清单
- 与测量数据对应的系统图

如果某个页面说不出自己的产物是什么，那它大概还只是一份主题清单。

---

## 课程质量标准

评审旧模块或新增模块时，请对照本标准。

| 标准 | 薄弱页面 | 优秀页面 |
|----------|-----------|-------------|
| 目的 | 罗列主题 | 说明工程问题及其重要性 |
| 结构 | 冗长的资源堆砌 | 引导学习者走完构建、使用、测量、交付 |
| 硬件相关性 | 隐含 | 明确把工作负载、runtime、内存、功耗、时序或部署与硬件关联起来 |
| 实践 | 「试试 X」 | 定义含命令、输入与预期输出的具体实验 |
| 测量 | 可有可无 | 指明能证明进展的指标 |
| 完成 | 含糊 | 定义可供其他工程师检查的产物 |
| 导航 | 孤立 | 链接前置要求与后续模块 |
| 角色映射 | 泛泛而谈 | 指明所准备的角色及其日常工作 |

### 最小可用课程页面

每个有分量的 `Guide.md` 都应包含：

- 一句话目的
- 层级映射
- 前置要求
- 后续内容
- 目标角色
- 该模块存在的理由
- 课程产出目标
- 单元图或学习路径
- 构建/实验工作
- 需采集的指标
- 最终产物
- 达成标准

简短的参考页面可以更轻量，但仍应说明该页面为何存在，以及学习者该拿它做什么。

### 产物阶梯

学习者应从简单产物逐步走向可入作品集的产物：

| 层级 | 产物质量 |
|-------|------------------|
| 1 | 笔记、图与命令日志 |
| 2 | 可运行的代码、RTL 或配置 |
| 3 | 可复现的构建/测试/benchmark 脚本 |
| 4 | 含原始数据与解读的测量报告 |
| 5 | 其他工程师可直接运行的可复用子系统、实验或案例研究 |

多数模块应以第 3 级或第 4 级为目标，capstone 应以第 5 级为目标。

---

## 标准课时模板

新增指南时，除非有充分理由，否则请采用以下结构：

## 为何重要

说明实际问题与硬件相关性。

## 心智模型

在讲实现细节之前，先用平实语言解释核心概念。

## 构建它

从零实现该概念，或贴近硬件底层实现。

## 在真实技术栈中使用

用真实工具、框架、驱动或板卡展示生产版本。

## 测量它

要求给出具体指标：延迟、吞吐、occupancy、带宽、功耗、准确率、利用率、内存、启动时间、热管理表现。

## 交付它

定义能证明学习者完成本单元的最终产物。

## 达成标准

列出学习者在继续前进之前必须能够解释、构建、测量或调试的内容。

---


<details>
<summary>English original</summary>

**Curriculum Authoring Guide**

> *How to add or expand roadmap content without diluting the project.*

This repository is strongest when every new page does three things well:

- teaches a real systems or hardware concept
- produces a visible engineering artifact
- connects cleanly to the 8-layer stack and downstream roles

Use this guide when adding modules, deep dives, labs, or project pages.

---

**Core Rules**

**1. Stay hardware-first**

This is not a generic AI course.

New content should answer at least one of these:

- What workload does the hardware need to run?
- How does software map onto hardware resources?
- How is performance, power, memory, or deployment affected?
- What role in the AI chip stack does this teach?

If a topic is interesting but does not sharpen hardware intuition, it probably does not belong here.

**2. Prefer build-first content**

Avoid pages that are only reading lists or concept summaries.

Every substantial guide should push the learner toward:

- code
- profiling
- debugging
- measurement
- deployment
- architecture reasoning

**3. Produce artifacts, not just notes**

Each module should suggest one or more artifacts that another engineer could inspect:

- benchmark table
- profiling trace
- board bring-up checklist
- device-tree patch
- CUDA kernel
- MLIR pass demo
- FPGA timing report
- system diagram

**4. Keep the role mapping explicit**

Every guide should make it easy to answer:

- Which layer does this belong to?
- What role does this help with?
- What phase or track should come next?

**5. Define what "done" means**

Every substantial course page should make completion observable. A learner should not finish a module by saying "I read it." They should finish with evidence:

- code that runs
- a build log
- a profiler trace
- a benchmark table
- a debug capture
- a design report
- a deployment checklist
- a system diagram tied to measurements

If a page cannot name the artifact, the page is probably still only a topic list.

---

**Course Quality Standard**

Use this standard when reviewing old modules or adding new ones.

| Standard | Weak page | Strong page |
|----------|-----------|-------------|
| Purpose | Lists topics | States the engineering problem and why it matters |
| Structure | Long resource dump | Guides the learner through build, use, measure, ship |
| Hardware relevance | Implied | Explicitly connects workload, runtime, memory, power, timing, or deployment to hardware |
| Practice | "Try X" | Defines a concrete lab with commands, inputs, and expected outputs |
| Measurement | Optional | Names the metrics that prove progress |
| Completion | Vague | Defines an artifact another engineer can inspect |
| Navigation | Isolated | Links prerequisites and next modules |
| Role mapping | Generic | Names roles and daily work this prepares for |

**Minimum viable course page**

Every substantial `Guide.md` should include:

- one-sentence purpose
- layer mapping
- prerequisites
- what comes after
- role targets
- why the module exists
- course outcomes
- unit map or learning path
- build/lab work
- metrics to collect
- final artifact
- exit criteria

Short reference pages can be lighter, but they should still state why the page exists and what the learner should do with it.

**Artifact ladder**

Learners should move from simple artifacts to portfolio-grade artifacts:

| Level | Artifact quality |
|-------|------------------|
| 1 | Notes, diagrams, and command logs |
| 2 | Working code, RTL, or configuration |
| 3 | Reproducible build/test/benchmark script |
| 4 | Measurement report with raw data and interpretation |
| 5 | Reusable subsystem, lab, or case study another engineer can run |

Most modules should target Level 3 or Level 4. Capstones should target Level 5.

---

**Standard Lesson Template**

For new guides, use this structure unless there is a strong reason not to:

**Why it matters**

State the practical problem and the hardware relevance.

**Mental model**

Explain the core concept in plain language before implementation details.

**Build it**

Implement the concept from scratch or close to the metal.

**Use it in the real stack**

Show the production version with real tools, frameworks, drivers, or boards.

**Measure it**

Ask for concrete metrics: latency, throughput, occupancy, bandwidth, power, accuracy, utilization, memory, boot time, thermal behavior.

**Ship it**

Define the final artifact that proves the learner completed the unit.

**Exit criteria**

List what the learner must be able to explain, build, measure, or debug before moving on.

---

</details>

## 优秀的模块成果是什么样的

| 领域 | 薄弱成果 | 优秀成果 |
|------|--------------|----------------|
| CUDA | “读一读 warp 的资料” | Nsight 报告，对比 coalesced 与 uncoalesced kernel |
| 嵌入式 Linux | “学 Yocto” | 可复现的镜像构建 + 启动日志 + 软件包差异 |
| Jetson | “试试 TensorRT” | 带延迟、RAM、功耗的 FP16 vs INT8 benchmark |
| FPGA | “学 HLS” | 综合出的 kernel，附利用率与时序结果 |
| ML 编译器 | “理解 MLIR” | 最小的 pass 或后端 lowering demo |

---

## 页面头部检查清单

在内容充实的指南顶部，应包含：

- 一句话说明目的
- layer 映射
- 前置要求
- 接下来学什么
- 若页面面向特定角色，列出角色目标

这样能让来自不同背景的学习者都能沿着路线图找到路径。

---

## 写作风格

- 像工程师教另一位工程师那样写。
- 多用具体论断，少用鼓舞性措辞。
- 表格用于对比，不用于装饰。
- 解释取舍，而不只是给定义。
- 指出实践中会在哪里失败。
- 资源列表要精选；不要堆砌大量未整理的链接。

---

## 深度准则

大致按以下划分：

- 概览类指南：定位、角色映射、模块结构、项目思路
- 子指南：实现细节、性能机理、调试流程
- 实验/项目：分步执行与可度量的产出

不要把每个实现细节都塞进概览类指南，应改为向下链接。

---

## 贡献检查清单

合并新的课程内容前，逐项确认：

- 该页面明确属于路线图的一部分
- 与硬件的关联是明确的
- 要求学习者动手构建某个东西
- 度量是完成标准的一部分
- 定义了最终产物
- 链接与前置要求正确
- 内容没有不必要地重复已有页面

---

## 推荐的配套文件

当模块内容变得充实，可考虑添加：

- 实验页面
- 产物跟踪表
- benchmark 模板
- 完整项目或案例研究

---

## 相关页面

- [首页 / 路线图概览](/学习资料/AI硬件工程师路线图/README)


<details>
<summary>English original</summary>

**What Good Module Outcomes Look Like**

| Area | Weak outcome | Strong outcome |
|------|--------------|----------------|
| CUDA | "Read about warps" | Nsight report comparing coalesced vs uncoalesced kernels |
| Embedded Linux | "Learn Yocto" | Reproducible image build + boot log + package diff |
| Jetson | "Try TensorRT" | FP16 vs INT8 benchmark with latency, RAM, power |
| FPGA | "Study HLS" | Synthesized kernel with utilization and timing results |
| ML Compiler | "Understand MLIR" | Minimal pass or backend lowering demo |

---

**Page Header Checklist**

At the top of a substantial guide, include:

- one-sentence purpose
- layer mapping
- prerequisites
- what comes after
- role targets if the page is specialized

This keeps the roadmap navigable for learners entering from different backgrounds.

---

**Writing Style**

- Write like an engineer teaching another engineer.
- Prefer concrete claims over inspirational language.
- Use tables for comparison, not for decoration.
- Explain tradeoffs, not just definitions.
- Show where things fail in practice.
- Keep resource lists curated; do not dump large unsorted link lists.

---

**Depth Guidelines**

Use this rough split:

- Overview guides: orientation, role mapping, module structure, project ideas
- Sub-guides: implementation detail, performance mechanics, debugging workflow
- Labs/projects: step-by-step execution and measurable outputs

Do not overload overview guides with every implementation detail. Link downward instead.

---

**Contribution Checklist**

Before merging new curriculum content, verify:

- The page clearly belongs in the roadmap
- Hardware relevance is explicit
- The learner is asked to build something
- Measurement is part of completion
- A final artifact is defined
- Links and prerequisites are correct
- The content does not duplicate an existing page unnecessarily

---

**Recommended Companion Files**

When a module becomes substantial, consider adding:

- a lab page
- an artifact tracker
- a benchmark template
- a worked project or case study

---

**Related Pages**

- [Home / Roadmap Overview](/学习资料/AI硬件工程师路线图/README)

</details>

---

> 原文：[`Curriculum-Authoring-Guide.md`](https://github.com/ai-hpc/ai-hardware-engineer-roadmap/blob/main/Curriculum-Authoring-Guide.md)（上游仓库 ai-hpc/ai-hardware-engineer-roadmap，MIT License）。本页为中文翻译，术语保留英文原词；与原文不一致时以原文为准。
