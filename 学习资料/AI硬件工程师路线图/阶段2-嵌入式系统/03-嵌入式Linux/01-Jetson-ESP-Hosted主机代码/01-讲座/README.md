---
title: Jetson ESP-Hosted 主机代码 — 讲义
description: Jetson ESP-Hosted 主机代码 — 讲义
published: true
date: 2026-09-27T12:29:59.000Z
tags: 学习资料
editor: markdown
dateCreated: 2026-09-27T12:29:59.000Z
---

# Jetson ESP-Hosted 主机代码 — 讲义

按顺序逐篇学习。每个文件是一次会话，均围绕真实的 `esp_hosted_ng/host/` Linux 主机代码构建。

**Hub:** [Jetson ESP-Hosted 主机代码指南](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/01-Jetson-ESP-Hosted主机代码/Guide)

| 讲义 | 主题 | 主要 kernel 接口 | 主要概念 |
|---------|-------|------------------------|---------------|
| [Lecture-01](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/01-Jetson-ESP-Hosted主机代码/01-讲座/Lecture-01) | 该主机栈的 Linux 心智模型 | `cfg80211`、HCI、netdev、SPI 传输拆分 | 子系统边界 |
| [Lecture-02](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/01-Jetson-ESP-Hosted主机代码/01-讲座/Lecture-02) | Jetson 上的构建、加载与板级策略 | kernel 模块、`module_param`、`spidev` 归属 | 板级策略 vs 驱动代码 |
| [Lecture-03](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/01-Jetson-ESP-Hosted主机代码/01-讲座/Lecture-03) | SPI 传输、GPIO 与 IRQ 驱动的 bring-up（上电点亮/调通） | `gpio_to_irq`、`request_irq`、SPI 设备复用 | 中断驱动的传输 |
| [Lecture-04](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/01-Jetson-ESP-Hosted主机代码/01-讲座/Lecture-04) | Wi-Fi 如何变成 `wlan0` | `wiphy_new`、`wiphy_register`、`register_netdevice` | Linux 无线子系统集成 |
| [Lecture-05](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/01-Jetson-ESP-Hosted主机代码/01-讲座/Lecture-05) | 低功耗蓝牙（BLE）如何变成 `hci0`，以及如何验证整条路径 | `hci_alloc_dev`、`hci_register_dev`、`hci_recv_frame` | HCI 与 BLE 主机/控制器拆分 |


<details>
<summary>English original</summary>

**Jetson ESP-Hosted Host Code — lectures**

Work through these in order. Each file is one session built around the real `esp_hosted_ng/host/` Linux host code.

**Hub:** [Jetson ESP-Hosted Host Code guide](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/01-Jetson-ESP-Hosted主机代码/Guide)

| Lecture | Topic | Main kernel interfaces | Main concepts |
|---------|-------|------------------------|---------------|
| [Lecture-01](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/01-Jetson-ESP-Hosted主机代码/01-讲座/Lecture-01) | Linux mental model for this host stack | `cfg80211`, HCI, netdev, SPI transport split | subsystem boundaries |
| [Lecture-02](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/01-Jetson-ESP-Hosted主机代码/01-讲座/Lecture-02) | Build, load, and board policy on Jetson | kernel modules, `module_param`, `spidev` ownership | board policy vs driver code |
| [Lecture-03](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/01-Jetson-ESP-Hosted主机代码/01-讲座/Lecture-03) | SPI transport, GPIOs, and IRQ-driven bring-up | `gpio_to_irq`, `request_irq`, SPI device reuse | interrupt-driven transport |
| [Lecture-04](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/01-Jetson-ESP-Hosted主机代码/01-讲座/Lecture-04) | How Wi-Fi becomes `wlan0` | `wiphy_new`, `wiphy_register`, `register_netdevice` | Linux wireless subsystem integration |
| [Lecture-05](/学习资料/AI硬件工程师路线图/阶段2-嵌入式系统/03-嵌入式Linux/01-Jetson-ESP-Hosted主机代码/01-讲座/Lecture-05) | How BLE becomes `hci0` and how to validate the full path | `hci_alloc_dev`, `hci_register_dev`, `hci_recv_frame` | HCI and BLE host/controller split |

</details>

---

> 原文：[`Phase 2 - Embedded Systems/3. Embedded Linux/Jetson ESP-Hosted Host Code/Lecture/README.md`](https://github.com/ai-hpc/ai-hardware-engineer-roadmap/blob/main/Phase%202%20-%20Embedded%20Systems/3.%20Embedded%20Linux/Jetson%20ESP-Hosted%20Host%20Code/Lecture/README.md)（上游仓库 ai-hpc/ai-hardware-engineer-roadmap，MIT License）。本页为中文翻译，术语保留英文原词；与原文不一致时以原文为准。
