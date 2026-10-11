---
title: Gem5体系结构学习路线
description: Gem5体系结构学习路线
published: true
date: 2026-10-11T09:57:17.662Z
tags: 
editor: markdown
dateCreated: 2026-10-11T09:57:17.662Z
---

# Gem5体系结构学习路线
本次学习路线将采用循序渐进的方式，stage演进如下

Stage 0	gem5 基础设施	建立统一的 EduArch-gem5 工程
Stage 1	数字逻辑、ISA 与单周期 CPU	EduCPU v1：单周期 RISC-V CPU
Stage 2	多周期 CPU 与流水线	EduCPU v2：五级流水线 CPU
Stage 3	超标量与顺序多发射	EduCPU v3：In-Order Superscalar CPU
Stage 4	乱序执行与动态调度	EduCPU v4：简化 Out-of-Order CPU
Stage 5	存储体系结构与多核系统	EduMemory：Cache、DRAM、Coherence、NoC
Stage 6	SIMD 与向量体系结构	EduVector：向量计算单元
Stage 7	GPU 架构与 SIMT	EduSIMT：简化 GPU Compute Unit
Stage 8	数据流架构与空间计算	EduDataflow：PE 阵列和数据流映射
Stage 9	AI 加速器系统架构	EduAccelerator：Tile、SRAM、DMA、NoC
Stage 10	量化架构设计与 DSE	面向 LLM 推理的架构评估平台