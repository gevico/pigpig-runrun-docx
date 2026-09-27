---
title: jetson-llm 文档
description: jetson-llm 文档
published: true
date: 2026-09-27T11:30:55.000Z
tags: 学习资料
editor: markdown
dateCreated: 2026-09-27T11:30:55.000Z
---

# jetson-llm 文档

| 文档 | 描述 |
|----------|-------------|
| [architecture.md](/学习资料/AI硬件工程师路线图/项目/01-Jetson-LLM运行时/01-文档/architecture) | 系统架构、数据流、设计决策 |
| [memory.md](/学习资料/AI硬件工程师路线图/项目/01-Jetson-LLM运行时/01-文档/memory) | 内存优先设计：budget、KV cache、scratch pool、OOM 防护 |
| [kernels.md](/学习资料/AI硬件工程师路线图/项目/01-Jetson-LLM运行时/01-文档/kernels) | 全部 6 个 CUDA kernel：各自的功能、Orin 调优、性能说明 |
| [gguf.md](/学习资料/AI硬件工程师路线图/项目/01-Jetson-LLM运行时/01-文档/gguf) | GGUF 格式解析：config、张量、tokenizer、权重映射 |
| [engine.md](/学习资料/AI硬件工程师路线图/项目/01-Jetson-LLM运行时/01-文档/engine) | Transformer 前向传播、decode 循环、CUDA graphs、采样 |
| [jetson-hal.md](/学习资料/AI硬件工程师路线图/项目/01-Jetson-LLM运行时/01-文档/jetson-hal) | 功耗模式、热管理、sysfs 接口、实时统计 |
| [server.md](/学习资料/AI硬件工程师路线图/项目/01-Jetson-LLM运行时/01-文档/server) | HTTP API：端点、请求/响应格式、部署 |
| [build.md](/学习资料/AI硬件工程师路线图/项目/01-Jetson-LLM运行时/01-文档/build) | 构建系统、依赖、交叉编译说明 |
| [testing.md](/学习资料/AI硬件工程师路线图/项目/01-Jetson-LLM运行时/01-文档/testing) | 测试计划、测试说明、预期结果、调试 |
| [performance.md](/学习资料/AI硬件工程师路线图/项目/01-Jetson-LLM运行时/01-文档/performance) | benchmark、性能剖析、优化目标 |


<details>
<summary>English original</summary>

**jetson-llm Documentation**

| Document | Description |
|----------|-------------|
| [architecture.md](/学习资料/AI硬件工程师路线图/项目/01-Jetson-LLM运行时/01-文档/architecture) | System architecture, data flow, design decisions |
| [memory.md](/学习资料/AI硬件工程师路线图/项目/01-Jetson-LLM运行时/01-文档/memory) | Memory-first design: budget, KV cache, scratch pool, OOM guard |
| [kernels.md](/学习资料/AI硬件工程师路线图/项目/01-Jetson-LLM运行时/01-文档/kernels) | All 6 CUDA kernels: what they do, Orin tuning, performance notes |
| [gguf.md](/学习资料/AI硬件工程师路线图/项目/01-Jetson-LLM运行时/01-文档/gguf) | GGUF format parsing: config, tensors, tokenizer, weight mapping |
| [engine.md](/学习资料/AI硬件工程师路线图/项目/01-Jetson-LLM运行时/01-文档/engine) | Transformer forward pass, decode loop, CUDA graphs, sampling |
| [jetson-hal.md](/学习资料/AI硬件工程师路线图/项目/01-Jetson-LLM运行时/01-文档/jetson-hal) | Power modes, thermal management, sysfs interface, live stats |
| [server.md](/学习资料/AI硬件工程师路线图/项目/01-Jetson-LLM运行时/01-文档/server) | HTTP API: endpoints, request/response format, deployment |
| [build.md](/学习资料/AI硬件工程师路线图/项目/01-Jetson-LLM运行时/01-文档/build) | Build system, dependencies, cross-compilation notes |
| [testing.md](/学习资料/AI硬件工程师路线图/项目/01-Jetson-LLM运行时/01-文档/testing) | Test plan, test descriptions, expected results, debugging |
| [performance.md](/学习资料/AI硬件工程师路线图/项目/01-Jetson-LLM运行时/01-文档/performance) | Benchmarking, profiling, optimization targets |

</details>

---

> 原文：[`Projects/jetson-llm-runtime/docs/README.md`](https://github.com/ai-hpc/ai-hardware-engineer-roadmap/blob/main/Projects/jetson-llm-runtime/docs/README.md)（上游仓库 ai-hpc/ai-hardware-engineer-roadmap，MIT License）。本页为中文翻译，术语保留英文原词；与原文不一致时以原文为准。
