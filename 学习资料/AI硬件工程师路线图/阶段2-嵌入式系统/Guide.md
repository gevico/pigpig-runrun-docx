---
title: 阶段 2：嵌入式系统
description: 阶段 2：嵌入式系统
published: true
date: 2026-09-27T09:12:30.000Z
tags: 学习资料
editor: markdown
dateCreated: 2026-09-27T09:12:30.000Z
---

# 阶段 2：嵌入式系统

<div class="course-identity embedded-systems" markdown="1">
<div class="course-identity__icon">EMB</div>
<div markdown="1">
<p class="course-identity__eyebrow">阶段 2 · 嵌入式系统</p>
<p class="course-identity__title">从抽象计算转向板卡、总线、boot 流程、受限设备与产品。</p>
<p class="course-identity__meta">产物：bring-up 日志 + 嵌入式 demo · 度量：boot、延迟、功耗、可靠性</p>
</div>
</div>


> *从抽象计算模型转向真实板卡、总线、boot 流程，以及必须在实验室之外工作的受限系统。*

**layer 映射：** 主要为 **L4**（固件、RTOS、BSP、嵌入式 Linux），并直接连接到 **L3**（驱动、DMA、设备接口）和 **L1**（边缘 AI 部署）。

**岗位目标：** 嵌入式软件工程师 · RTOS 工程师 · BSP 工程师 · 嵌入式 Linux 工程师 · 边缘 AI 工程师

**前置要求：** [阶段 1 — 数字基础](/学习资料/AI硬件工程师路线图/阶段1-基础知识/Guide)

**后续内容：** [阶段 3 — 人工智能](/学习资料/AI硬件工程师路线图/阶段3-人工智能/Guide)，然后是阶段 4 的其中一条路线：[Xilinx FPGA](/学习资料/AI硬件工程师路线图/阶段4-Xilinx-FPGA/01-Xilinx-FPGA开发/Guide)、[NVIDIA Jetson](/学习资料/AI硬件工程师路线图/阶段4-路线B-Nvidia-Jetson/01-Nvidia-Jetson平台/Guide) 或 [ML 编译器](/学习资料/AI硬件工程师路线图/阶段4-路线C-深度学习推理优化/Guide)

---

## 为什么需要这个阶段

AI 硬件并非孤立存在。它装在系统中出货，而这些系统必须：

- 可靠启动
- 与传感器和外设通信
- 承受功耗和热管理限制
- 在现场安全更新
- 与 Linux、RTOS 以及板级专用软件栈集成

这个阶段教授的是介于原始硬件知识与可部署产品之间的工程层。

---

## 阶段结构

| # | 模块 | 你学到什么 | 为什么重要 |
|---|--------|----------------|----------------|
| **1** | [原理图绘制与 PCB 设计](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/01-原理图与PCB设计/Guide) | 板级设计、接口、布局布线基础、bring-up 考量 | AI 产品仍然依赖真实硬件集成 |
| **2** | [嵌入式软件](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/02-嵌入式软件/Guide) | Cortex-M、FreeRTOS、中断、DMA、SPI/I2C/UART/CAN 等总线 | 围绕传感器、外设和底层设备的控制面 |
| **3** | [嵌入式 Linux](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/Guide) | Yocto、BSP 定制、rootfs、kernel 集成、生产级 Linux | Jetson、机器人和边缘设备的软件基础 |
| **4** | [产品设计](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/04-产品设计/Guide) | 产品角色、信任线索、房间行为、安装设置和系统边界 | 良好的嵌入式系统若沦为弱产品，依然会失败 |

**推荐顺序：** `1 → 2 → 3 → 4`

如果你使用开发套件而非自制板卡，模块 2 可以与模块 1 的部分内容并行开始。当你已经具备足够的硬件和软件背景、能够权衡 tradeoff 而不只是罗列功能时，模块 4 的价值最大。

---

## 你应该产出什么

到本阶段结束时，你至少应该拥有：

- 一件面向硬件或接口的产物，例如原理图评审、接口映射或 bring-up 检查清单
- 一件 MCU（微控制器）/RTOS 产物，例如外设驱动、ISR 设计说明或 FreeRTOS 项目
- 一件嵌入式 Linux 产物，例如 Yocto image 构建说明、设备树改动、rootfs 定制或 BSP 文档
- 一件产品产物，例如 V1 设计简报、信任模型，或布局与控制方案的理由说明

这些输出之所以重要，是因为嵌入式的可信度来自能跑通的系统，而非泛泛的熟悉。

---

## 达成标准

当你能做到以下各项时，就可以继续：

- 解释外设如何通过总线、中断和驱动抵达软件
- 推理 RTOS 任务化与裸机、Linux 之间的 tradeoff
- 指出 BSP、bootloader、设备树和 rootfs 定制在部署栈中的位置
- 解释至少一个涉及控制、信任、声学、布局或安装设置的产品 tradeoff
- 产出至少一件可复现的 bring-up 或嵌入式 Linux 产物

---

## 谁应该优先考虑这个阶段

- **Jetson / 机器人 / 边缘 AI 方向：** 将此阶段视为必修
- **编译器优先的学习者：** 做到足以理解部署现实，然后进入阶段 3 和阶段 4C
- **FPGA / 加速器学习者：** 聚焦于将定制硬件连接到可用系统的 Linux 和接口部分
- **消费设备 / 家庭 AI / 语音产品方向：** 不要跳过模块 4，因为房间角色和信任决策可能让其他方面扎实的工程失效

---


<details>
<summary>English original</summary>

**Phase 2: Embedded Systems**

<div class="course-identity embedded-systems" markdown="1">
<div class="course-identity__icon">EMB</div>
<div markdown="1">
<p class="course-identity__eyebrow">Phase 2 · Embedded Systems</p>
<p class="course-identity__title">Move from abstract computing to boards, buses, boot flows, constrained devices, and products.</p>
<p class="course-identity__meta">Artifact: bring-up log + embedded demo · Measure: boot, latency, power, reliability</p>
</div>
</div>


> *Move from abstract computing models to real boards, buses, boot flows, and constrained systems that must work outside the lab.*

**Layer mapping:** Primarily **L4** (firmware, RTOS, BSP, embedded Linux), with direct connections into **L3** (drivers, DMA, device interfaces) and **L1** (edge AI deployment).

**Role targets:** Embedded Software Engineer · RTOS Engineer · BSP Engineer · Embedded Linux Engineer · Edge AI Engineer

**Prerequisites:** [Phase 1 — Digital Foundations](/学习资料/AI硬件工程师路线图/阶段1-基础知识/Guide)

**What comes after:** [Phase 3 — Artificial Intelligence](/学习资料/AI硬件工程师路线图/阶段3-人工智能/Guide), then one of the Phase 4 tracks: [Xilinx FPGA](/学习资料/AI硬件工程师路线图/阶段4-Xilinx-FPGA/01-Xilinx-FPGA开发/Guide), [NVIDIA Jetson](/学习资料/AI硬件工程师路线图/阶段4-路线B-Nvidia-Jetson/01-Nvidia-Jetson平台/Guide), or [ML Compiler](/学习资料/AI硬件工程师路线图/阶段4-路线C-深度学习推理优化/Guide)

---

**Why This Phase Exists**

AI hardware does not live in isolation. It ships inside systems that must:

- boot reliably
- talk to sensors and peripherals
- survive power and thermal limits
- update safely in the field
- integrate with Linux, RTOS, and board-specific software stacks

This phase teaches the engineering layer between raw hardware knowledge and deployable products.

---

**Phase Structure**

| # | Module | What you learn | Why it matters |
|---|--------|----------------|----------------|
| **1** | [Schematic Capture and PCB Design](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/01-原理图与PCB设计/Guide) | board-level design, interfaces, layout basics, bring-up concerns | AI products still depend on real hardware integration |
| **2** | [Embedded Software](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/02-嵌入式软件/Guide) | Cortex-M, FreeRTOS, interrupts, DMA, buses like SPI/I2C/UART/CAN | The control plane around sensors, peripherals, and low-level devices |
| **3** | [Embedded Linux](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/Guide) | Yocto, BSP customization, rootfs, kernel integration, production Linux | The software foundation for Jetson, robotics, and edge devices |
| **4** | [Product Design](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/04-产品设计/Guide) | product role, trust cues, room behavior, setup, and system boundaries | Good embedded systems still fail if they become weak products |

**Recommended order:** `1 → 2 → 3 → 4`

If you are using development kits instead of custom boards, Module 2 can start in parallel with parts of Module 1. Module 4 is most valuable once you already have enough hardware and software context to reason about tradeoffs, not just features.

---

**What You Should Produce**

By the end of this phase, you should have at least:

- one hardware or interface-oriented artifact such as a schematic review, interface map, or bring-up checklist
- one MCU/RTOS artifact such as a peripheral driver, ISR design note, or FreeRTOS project
- one embedded Linux artifact such as a Yocto image build note, device-tree change, rootfs customization, or BSP write-up
- one product artifact such as a V1 design brief, trust model, or placement-and-controls rationale

These outputs matter because embedded credibility comes from working systems, not generic familiarity.

---

**Exit Criteria**

You are ready to continue when you can:

- explain how a peripheral reaches software through buses, interrupts, and drivers
- reason about RTOS tasking vs bare-metal vs Linux tradeoffs
- identify where BSP, bootloader, device tree, and rootfs customization fit in a deployment stack
- explain at least one product tradeoff involving controls, trust, acoustics, placement, or setup
- produce at least one reproducible bring-up or embedded Linux artifact

---

**Who Should Prioritize This Phase**

- **Jetson / robotics / edge AI targets:** treat this phase as mandatory
- **Compiler-first learners:** do enough to understand deployment realities, then move to Phase 3 and Phase 4C
- **FPGA / accelerator learners:** focus on the Linux and interface pieces that connect custom hardware to usable systems
- **Consumer device / home AI / voice product targets:** do not skip Module 4, because room-role and trust decisions can invalidate otherwise solid engineering

---

</details>

## Next

→ [**阶段 3 — 人工智能**](/学习资料/AI硬件工程师路线图/阶段3-人工智能/Guide) · [**阶段 4 方向 A — Xilinx FPGA**](/学习资料/AI硬件工程师路线图/阶段4-Xilinx-FPGA/01-Xilinx-FPGA开发/Guide) · [**阶段 4 方向 B — NVIDIA Jetson**](/学习资料/AI硬件工程师路线图/阶段4-路线B-Nvidia-Jetson/01-Nvidia-Jetson平台/Guide) · [**阶段 4 方向 C — ML 编译器**](/学习资料/AI硬件工程师路线图/阶段4-路线C-深度学习推理优化/Guide)


<details>
<summary>English original</summary>

**Next**

→ [**Phase 3 — Artificial Intelligence**](/学习资料/AI硬件工程师路线图/阶段3-人工智能/Guide) · [**Phase 4 Track A — Xilinx FPGA**](/学习资料/AI硬件工程师路线图/阶段4-Xilinx-FPGA/01-Xilinx-FPGA开发/Guide) · [**Phase 4 Track B — NVIDIA Jetson**](/学习资料/AI硬件工程师路线图/阶段4-路线B-Nvidia-Jetson/01-Nvidia-Jetson平台/Guide) · [**Phase 4 Track C — ML Compiler**](/学习资料/AI硬件工程师路线图/阶段4-路线C-深度学习推理优化/Guide)

</details>

---

> 原文：[`Phase 2 - Embedded Systems/Guide.md`](https://github.com/ai-hpc/ai-hardware-engineer-roadmap/blob/main/Phase%202%20-%20Embedded%20Systems/Guide.md)（上游仓库 ai-hpc/ai-hardware-engineer-roadmap，MIT License）。本页为中文翻译，术语保留英文原词；与原文不一致时以原文为准。
