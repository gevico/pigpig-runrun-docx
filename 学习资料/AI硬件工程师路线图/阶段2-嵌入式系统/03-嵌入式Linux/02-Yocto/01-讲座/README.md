---
title: Yocto — 循序渐进的讲义
description: Yocto — 循序渐进的讲义
published: true
date: 2026-09-27T11:30:40.000Z
tags: 学习资料
editor: markdown
dateCreated: 2026-09-27T11:30:40.000Z
---

# Yocto — 循序渐进的讲义

按顺序学完这些内容。每个文件是一个 session：先读，再做文末的实验，然后继续下一节。

**Hub：** [Yocto 课程指南](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/02-Yocto/Guide)

| 讲义 | 主题 |
|---------|--------|
| [Lecture-01](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/02-Yocto/01-讲座/Lecture-01) | 如何使用本课程 |
| [Lecture-02](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/02-Yocto/01-讲座/Lecture-02) | Module 0 — 前置要求与主机环境搭建 |
| [Lecture-03](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/02-Yocto/01-讲座/Lecture-03) | Module 1 — Yocto 是什么（以及不是什么）（[Lab 1 示例](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/02-Yocto/01-讲座/Lab-01-Worked-Example-Edge-AI-Camera)） |
| [Lecture-04](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/02-Yocto/01-讲座/Lecture-04) | Module 2 — 一张图看懂架构 |
| [Lecture-05](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/02-Yocto/01-讲座/Lecture-05) | Module 3 — 第一次成功构建 |
| [Lecture-06](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/02-Yocto/01-讲座/Lecture-06) | Module 4 — recipe：系统的原子单元 |
| [Lecture-07](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/02-Yocto/01-讲座/Lecture-07) | Module 5 — layer：元数据如何保持可维护 |
| [Lecture-08](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/02-Yocto/01-讲座/Lecture-08) | Module 6 — 镜像、软件包与特性 |
| [Lecture-09](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/02-Yocto/01-讲座/Lecture-09) | Module 7 — Kernel、bootloader、设备树 |
| [Lecture-10](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/02-Yocto/01-讲座/Lecture-10) | Module 8 — SDK 与应用开发工作流 |
| [Lecture-11](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/02-Yocto/01-讲座/Lecture-11) | Module 9 — 像工程师一样调试构建 |
| [Lecture-12](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/02-Yocto/01-讲座/Lecture-12) | Module 10 — 许可证、合规与供应链 |
| [Lecture-13](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/02-Yocto/01-讲座/Lecture-13) | Module 11 — 性能、缓存与 CI |
| [Lecture-14](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/02-Yocto/01-讲座/Lecture-14) | Capstone 项目 |
| [Lecture-15](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/02-Yocto/01-讲座/Lecture-15) | 术语表与快速参考 |
| [Lecture-16](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/02-Yocto/01-讲座/Lecture-16) | 延伸阅读 |


<details>
<summary>English original</summary>

**Yocto — step-by-step lectures**

Work through these in order. Each file is one session: read it, do the lab at the end, then move on.

**Hub:** [Yocto course guide](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/02-Yocto/Guide)

| Lecture | Topic |
|---------|--------|
| [Lecture-01](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/02-Yocto/01-讲座/Lecture-01) | How to use this course |
| [Lecture-02](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/02-Yocto/01-讲座/Lecture-02) | Module 0 — Prerequisites and host setup |
| [Lecture-03](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/02-Yocto/01-讲座/Lecture-03) | Module 1 — What Yocto is (and is not) ([Lab 1 example](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/02-Yocto/01-讲座/Lab-01-Worked-Example-Edge-AI-Camera)) |
| [Lecture-04](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/02-Yocto/01-讲座/Lecture-04) | Module 2 — Architecture in one picture |
| [Lecture-05](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/02-Yocto/01-讲座/Lecture-05) | Module 3 — First successful build |
| [Lecture-06](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/02-Yocto/01-讲座/Lecture-06) | Module 4 — Recipes: the atoms of the system |
| [Lecture-07](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/02-Yocto/01-讲座/Lecture-07) | Module 5 — Layers: how metadata stays maintainable |
| [Lecture-08](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/02-Yocto/01-讲座/Lecture-08) | Module 6 — Images, packages, and features |
| [Lecture-09](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/02-Yocto/01-讲座/Lecture-09) | Module 7 — Kernel, bootloader, device tree |
| [Lecture-10](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/02-Yocto/01-讲座/Lecture-10) | Module 8 — SDK and application workflow |
| [Lecture-11](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/02-Yocto/01-讲座/Lecture-11) | Module 9 — Debugging builds like an engineer |
| [Lecture-12](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/02-Yocto/01-讲座/Lecture-12) | Module 10 — Licenses, compliance, supply-chain |
| [Lecture-13](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/02-Yocto/01-讲座/Lecture-13) | Module 11 — Performance, caching, and CI |
| [Lecture-14](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/02-Yocto/01-讲座/Lecture-14) | Capstone projects |
| [Lecture-15](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/02-Yocto/01-讲座/Lecture-15) | Glossary and quick reference |
| [Lecture-16](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/02-Yocto/01-讲座/Lecture-16) | Further reading |

</details>

---

> 原文：[`Phase 2 - Embedded Systems/3. Embedded Linux/Yocto/Lecture/README.md`](https://github.com/ai-hpc/ai-hardware-engineer-roadmap/blob/main/Phase%202%20-%20Embedded%20Systems/3.%20Embedded%20Linux/Yocto/Lecture/README.md)（上游仓库 ai-hpc/ai-hardware-engineer-roadmap，MIT License）。本页为中文翻译，术语保留英文原词；与原文不一致时以原文为准。
