---
title: 卡尔曼滤波学习系列
description: 卡尔曼滤波学习系列
published: true
date: 2026-09-27T11:30:41.000Z
tags: 学习资料
editor: markdown
dateCreated: 2026-09-27T11:30:41.000Z
---

# 卡尔曼滤波学习系列

属于 [AI Hardware Engineer Roadmap](/学习资料/AI硬件工程师路线图/README) 的一部分。

## 学习路径

| 等级 | 章节 |
|-------|----------|
| **起点** | [00 - Index](/学习资料/AI硬件工程师路线图/阶段3-人工智能/03-硬件与边缘AI/02-传感器融合/03-kalman-filter/00-index) |
| **小学** | [01](/学习资料/AI硬件工程师路线图/阶段3-人工智能/03-硬件与边缘AI/02-传感器融合/03-kalman-filter/01-what-is-estimation) · [02](/学习资料/AI硬件工程师路线图/阶段3-人工智能/03-硬件与边缘AI/02-传感器融合/03-kalman-filter/02-noisy-measurements) · [03](/学习资料/AI硬件工程师路线图/阶段3-人工智能/03-硬件与边缘AI/02-传感器融合/03-kalman-filter/03-combining-information) |
| **初中** | [04](/学习资料/AI硬件工程师路线图/阶段3-人工智能/03-硬件与边缘AI/02-传感器融合/03-kalman-filter/04-averages-and-uncertainty) · [05](/学习资料/AI硬件工程师路线图/阶段3-人工智能/03-硬件与边缘AI/02-传感器融合/03-kalman-filter/05-weighted-averages) · [06](/学习资料/AI硬件工程师路线图/阶段3-人工智能/03-硬件与边缘AI/02-传感器融合/03-kalman-filter/06-prediction) |
| **高中** | [07](/学习资料/AI硬件工程师路线图/阶段3-人工智能/03-硬件与边缘AI/02-传感器融合/03-kalman-filter/07-kalman-filter-idea) · [08](/学习资料/AI硬件工程师路线图/阶段3-人工智能/03-硬件与边缘AI/02-传感器融合/03-kalman-filter/08-1d-kalman-filter) · [09](/学习资料/AI硬件工程师路线图/阶段3-人工智能/03-硬件与边缘AI/02-传感器融合/03-kalman-filter/09-tracking-moving-object) |
| **本科** | [10](/学习资料/AI硬件工程师路线图/阶段3-人工智能/03-硬件与边缘AI/02-传感器融合/03-kalman-filter/10-matrix-form) · [11](/学习资料/AI硬件工程师路线图/阶段3-人工智能/03-硬件与边缘AI/02-传感器融合/03-kalman-filter/11-multidimensional-kf) |

## 参考

- [Cheat Sheet](/学习资料/AI硬件工程师路线图/阶段3-人工智能/03-硬件与边缘AI/02-传感器融合/03-kalman-filter/cheat-sheet)
- [传感器融合指南](/学习资料/AI硬件工程师路线图/阶段3-人工智能/03-硬件与边缘AI/02-传感器融合/Guide)

## 示例输出

![1D](/学习资料/AI硬件工程师路线图/Assets/images/kalman_1d_tracking.png)
![2D](/学习资料/AI硬件工程师路线图/Assets/images/2d_tracking.png)
![6D](/学习资料/AI硬件工程师路线图/Assets/images/kalman_6d_tracking.png)

## 运行

```bash
pip install numpy matplotlib
python kalman_1d.py   # or kalman_2d.py, kalman_6d.py
```

## 外部链接

- [Kalman Filter (Wikipedia)](https://en.wikipedia.org/wiki/Kalman_filter)
- [Roger Labbe's Book](https://github.com/rlabbe/Kalman-and-Bayesian-Filters-in-Python)


<details>
<summary>English original</summary>

**Kalman Filter Learning Series**

Part of the [AI Hardware Engineer Roadmap](/学习资料/AI硬件工程师路线图/README).

**Learning Path**

| Level | Chapters |
|-------|----------|
| **Start** | [00 - Index](/学习资料/AI硬件工程师路线图/阶段3-人工智能/03-硬件与边缘AI/02-传感器融合/03-kalman-filter/00-index) |
| **Elementary** | [01](/学习资料/AI硬件工程师路线图/阶段3-人工智能/03-硬件与边缘AI/02-传感器融合/03-kalman-filter/01-what-is-estimation) · [02](/学习资料/AI硬件工程师路线图/阶段3-人工智能/03-硬件与边缘AI/02-传感器融合/03-kalman-filter/02-noisy-measurements) · [03](/学习资料/AI硬件工程师路线图/阶段3-人工智能/03-硬件与边缘AI/02-传感器融合/03-kalman-filter/03-combining-information) |
| **Middle School** | [04](/学习资料/AI硬件工程师路线图/阶段3-人工智能/03-硬件与边缘AI/02-传感器融合/03-kalman-filter/04-averages-and-uncertainty) · [05](/学习资料/AI硬件工程师路线图/阶段3-人工智能/03-硬件与边缘AI/02-传感器融合/03-kalman-filter/05-weighted-averages) · [06](/学习资料/AI硬件工程师路线图/阶段3-人工智能/03-硬件与边缘AI/02-传感器融合/03-kalman-filter/06-prediction) |
| **High School** | [07](/学习资料/AI硬件工程师路线图/阶段3-人工智能/03-硬件与边缘AI/02-传感器融合/03-kalman-filter/07-kalman-filter-idea) · [08](/学习资料/AI硬件工程师路线图/阶段3-人工智能/03-硬件与边缘AI/02-传感器融合/03-kalman-filter/08-1d-kalman-filter) · [09](/学习资料/AI硬件工程师路线图/阶段3-人工智能/03-硬件与边缘AI/02-传感器融合/03-kalman-filter/09-tracking-moving-object) |
| **Undergraduate** | [10](/学习资料/AI硬件工程师路线图/阶段3-人工智能/03-硬件与边缘AI/02-传感器融合/03-kalman-filter/10-matrix-form) · [11](/学习资料/AI硬件工程师路线图/阶段3-人工智能/03-硬件与边缘AI/02-传感器融合/03-kalman-filter/11-multidimensional-kf) |

**Reference**

- [Cheat Sheet](/学习资料/AI硬件工程师路线图/阶段3-人工智能/03-硬件与边缘AI/02-传感器融合/03-kalman-filter/cheat-sheet)
- [Sensor Fusion Guide](/学习资料/AI硬件工程师路线图/阶段3-人工智能/03-硬件与边缘AI/02-传感器融合/Guide)

**Example Outputs**

![1D](/学习资料/AI硬件工程师路线图/Assets/images/kalman_1d_tracking.png)
![2D](/学习资料/AI硬件工程师路线图/Assets/images/2d_tracking.png)
![6D](/学习资料/AI硬件工程师路线图/Assets/images/kalman_6d_tracking.png)

**Run**

```bash
pip install numpy matplotlib
python kalman_1d.py   # or kalman_2d.py, kalman_6d.py
```

**External**

- [Kalman Filter (Wikipedia)](https://en.wikipedia.org/wiki/Kalman_filter)
- [Roger Labbe's Book](https://github.com/rlabbe/Kalman-and-Bayesian-Filters-in-Python)

</details>

---

> 原文：[`Phase 3 - Artificial Intelligence/Track A - Hardware and Edge AI/4. Sensor Fusion/kalman-filter/README.md`](https://github.com/ai-hpc/ai-hardware-engineer-roadmap/blob/main/Phase%203%20-%20Artificial%20Intelligence/Track%20A%20-%20Hardware%20and%20Edge%20AI/4.%20Sensor%20Fusion/kalman-filter/README.md)（上游仓库 ai-hpc/ai-hardware-engineer-roadmap，MIT License）。本页为中文翻译，术语保留英文原词；与原文不一致时以原文为准。
