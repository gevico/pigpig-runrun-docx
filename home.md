---
title: LLM ASIC 文档站
description: LLM ASIC 文档站
published: true
date: 2026-09-15T03:31:31.731Z
tags: 
editor: markdown
dateCreated: 2026-09-14T09:08:34.701Z
---

# LLM ASIC文档站

主要用于存放项目文档，个人blog，学习资料等


## 项目定位(后续会不断细化)
项目定位为面向 LLM 推理场景的 dataflow 架构 ASIC 芯片，通过 PCIe/CXL 与 SoC 互联。


## 当前阶段
项目处于 POC（概念验证）阶段，微架构采用 TT 小规模版本。

POC 目标：总体目标是 搭建简易完整闭环系统

- 搭建简易性能模型，tt 简略版本
- Kmd umd 简易版本
- 算子移植常见的3-5种算子
- 完成llm选型
- 推理引擎 大部分理解其工作过程

## 分工(大致按照业界岗位进行划分)
1. 项目 PM: 泽文
1. gem5 搭建性能模型: 曹君泽，冷俊豪，王嘉辰
2. kmd/umd 驱动: 曹君泽
3. 算子移植,MLIR:  rd
4. 大模型选型调研:王宇新，冷
5. 推理引擎学习调研：jensen