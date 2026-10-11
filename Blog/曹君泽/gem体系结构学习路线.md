---
title: Gem5体系结构学习路线
description: Gem5体系结构学习路线
published: true
date: 2026-10-11T11:45:16.926Z
tags: 
editor: markdown
dateCreated: 2026-10-11T09:57:17.662Z
---

# Gem5体系结构学习路线

本次学习路线将采用循序渐进的方式，stage演进如下

| Stage | 主题 | 产出/目标 |
| --- | --- | --- |
| 1 | 数字逻辑、ISA 与单周期 CPU | EduCPU v1：单周期 RISC-V CPU |
| 2 | 多周期 CPU 与流水线 | EduCPU v2：五级流水线 CPU |
| 3 | 超标量与顺序多发射 | EduCPU v3：In-Order Superscalar CPU |
| 4 | 乱序执行与动态调度 | EduCPU v4：简化 Out-of-Order CPU |
| 5 | 存储体系结构与多核系统 | EduMemory：Cache、DRAM、Coherence、NoC |
| 6 | SIMD 与向量体系结构 | EduVector：向量计算单元 |
| 7 | GPU 架构与 SIMT | EduSIMT：简化 GPU Compute Unit |
| 8 | 数据流架构与空间计算 | EduDataflow：PE 阵列和数据流映射 |
| 9 | AI 加速器系统架构 | EduAccelerator：Tile、SRAM、DMA、NoC |
| 10 | 量化架构设计与 DSE | 面向 LLM 推理的架构评估平台 |

本次学习路线主要参考《一生一芯计算机体系结构量化研究方法》，并结合我们的项目目标，做数据流架构中面向大模型推理场景所需要的部分知识。

在这个过程中，我也一直在思考：AI时代到底需要什么样的学习路线？需要什么样的教程，才能更好地帮助大家学习？我最后想采取的方案是：给大家列出一个学习路线，并提供基础工程，规划每一个Stage要做的事情，包括完成这个Stage后应该了解哪些概念、回答哪些问题。除此之外，大家尽可能和大模型做讨论吧。

同时也不要忘记阅读一些权威材料，比如“一生一芯”的讲义，以及《计算机体系结构量化研究方法》《计算机组成与设计：硬件/软件接口》等经典书籍。

阅读材料:

- “一生一芯”讲义：https://ysyx.oscc.cc/docs/
- 《计算机体系结构：量化研究方法》（第 6 版）：https://book.douban.com/subject/36108789/
- 《计算机组成与设计：硬件/软件接口》（RISC-V 版，原书第 2 版）：https://book.douban.com/subject/36490912/

这些书籍在网上都有对应的免费PDF，大家可自行寻找。

## stage 0：gem5前置
本阶段的目标是为了了解 gem5 的基本运行机制，可以上手开始使用gem5，下一阶段会正式开始使用gem5 进行建模。

请结合源码阅读这个文档:
https://www.gem5.org/documentation/learning_gem5/introduction/

阅读完成后请回答以下问题：
1. gem5 是什么？它的主要用途是什么？
2. 为什么说 gem5 是一个 Modular Simulator Platform？
3. 什么是 Discrete Event Simulation（离散事件仿真）？
4. gem5 支持模拟哪些计算机系统和组件？
5. gem5 的 SE Mode 和 FS Mode 有什么区别？分别适用于什么情况？
6. ISA 和 CPU Microarchitecture 有什么区别？gem5 如何支持研究不同的 CPU 微架构？
7. gem5 为什么需要不同的 CPU Model？例如 In-Order 和 Out-of-Order？
8. gem5 如何支持不同存储系统的架构研究？
9. gem5 与 RTL 仿真器有什么区别？我们为什么选择 gem5 进行前期架构探索？

## Stage 1：数字逻辑、指令集与单周期 CPU

本阶段的目标是理解组合逻辑、时序逻辑、指令集架构及 CPU 的基本工作原理，并基于 Stage 0 的 gem5 基础工程，实现一个能够执行简单 RISC-V 程序的单周期 CPU，同时建立初步的性能量化分析能力。

### 1. 理论与源码学习

请结合以下文档和 gem5 源码进行学习：

**《一生一芯》：**
1. [F3：数字逻辑电路基础](https://ysyx.oscc.cc/docs/2607/f/3.html)
2. [F4：计算机系统的状态机模型](https://ysyx.oscc.cc/docs/2607/f/4.html)
3. [F5：支持数列求和的简单处理器](https://ysyx.oscc.cc/docs/2607/f/5.html)
4. [E5：功能完备的迷你 RISC-V 处理器](https://ysyx.oscc.cc/docs/2607/e/5.html)

**gem5 文档与源码：**
1. [Creating a Simple SimObject](https://www.gem5.org/documentation/learning_gem5/part2/helloobject/)
2. [Event-driven Programming](https://www.gem5.org/documentation/learning_gem5/part2/events/)
3. [Adding Parameters to SimObjects](https://www.gem5.org/documentation/learning_gem5/part2/parameters/)
4. [gem5 AtomicSimpleCPU 源码](https://gem5.googlesource.com/public/gem5/+/refs/heads/develop/src/cpu/simple/atomic.cc)
5. [gem5 BaseSimpleCPU 源码](https://gem5.googlesource.com/public/gem5/+/refs/heads/develop/src/cpu/simple/base.cc)

源码阅读重点放在 CPU 状态维护、指令执行、PC 更新、Tick Event 和周期调度，不要求理解完整的 gem5 CPU 框架。

### 2. 实践任务：EduCPU v1

基于 Stage 0 的 gem5 基础工程，实现一个简化的 **RV32I 单周期 CPU**。

![alt text](image-1.png) 这是一个简单的单周期CPU框图，可以作为参考。


要求至少包含：
- PC Register、Register File。
- Instruction Memory、Data Memory（理想单周期访问）。
- Instruction Decoder、Immediate Generator。
- ALU、Control Unit、Next-PC Logic。
- 基本算术、逻辑、Load/Store 和 Branch 指令。
- 基于 gem5 Event Queue 的周期执行机制。
- 指令执行 Trace、周期与指令计数器。
- 正确性测试及 Benchmark。

至少能够运行整数累加、数组求和、条件分支等简单程序。

### 3. 量化分析实验

完成以下实验：

1. **指令执行分析**：统计不同类型指令的执行数量、周期数和比例。
2. **CPI / IPC 分析**：计算 Benchmark 的 CPI、IPC 和执行时间。
3. **关键路径分析**：为 CPU 各组合模块设置假设延迟，估算不同指令的数据通路延迟及理论时钟周期。
4. **架构参数分析**：改变假设模块延迟和时钟频率，分析单周期 CPU 的性能限制。


### 4. 完成后请回答以下问题

**数字逻辑与时序系统**

1. 什么是 Combinational Logic（组合逻辑）和 Sequential Logic（时序逻辑）？两者有什么区别？
2. Register、MUX、Decoder、ALU 分别是什么？在 CPU 中承担什么职责？
3. 什么是 Clock、Clock Edge、Clock Period？为什么 CPU 需要时钟？
4. 什么是有限状态机（FSM）？如何用状态机描述一个 CPU 的执行过程？
5. 什么是当前状态（Current State）和下一状态（Next State）？为什么状态更新与组合逻辑计算需要区分？
6. 什么是 Critical Path（关键路径）？它如何限制 CPU 的时钟频率？

**指令集与 CPU 工作原理**

7. 什么是 ISA？为什么需要区分 ISA 和 Microarchitecture？
8. RISC-V RV32I 的基本指令格式有哪些？R-type、I-type、S-type、B-type、U-type、J-type 分别用于什么？
9. PC（Program Counter）是什么？它什么时候加 4，什么时候不加 4？
10. CPU 执行一条指令需要经过哪些逻辑步骤？Instruction Fetch、Decode、Execute、Memory Access、Writeback 分别完成什么？
11. Register File 是什么？RISC-V 的 `rs1`、`rs2`、`rd` 和 `x0` 有什么作用？
12. 一条 `ADD`、`LW`、`SW`、`BEQ` 指令分别会经过哪些数据通路？会修改哪些体系结构状态？
13. Control Unit 如何根据 Opcode、funct3、funct7 等字段生成控制信号？
14. 为什么一个单周期 CPU 可以让每条受支持的指令在一个时钟周期内完成？这种设计有哪些限制？

**gem5 实现机制**

15. gem5 中的 SimObject 和 ClockedObject 如何用于描述一个具有状态的 CPU？
16. gem5 的 Event Queue 如何驱动 CPU 执行？如何利用 `tick()` 和事件调度模拟时钟周期？
17. 如何在 gem5 中分别描述 CPU 的组合逻辑和时序逻辑？
18. 阅读 AtomicSimpleCPU 的源码，分析它如何进行指令执行和事件调度。为什么它不能直接等同于教材中的单周期 CPU？
19. gem5 的周期级行为模型与真实 RTL 中的组合逻辑、寄存器和时钟时序有什么区别？

**性能量化分析**

20. 什么是 Instruction Count、CPI、IPC、Clock Frequency 和 Execution Time？它们有什么关系？
21. 在理想单周期 CPU 中，CPI 和 IPC 分别是多少？成立的前提条件是什么？
22. 为什么单周期 CPU 的执行时间不仅由指令数量决定，还受到时钟周期的影响？
23. 如果 ALU、Register File、Instruction Memory 和 Data Memory 具有不同的组合延迟，如何估算 CPU 的关键路径和理论最高时钟频率？
24. 为什么减少指令执行周期数，不一定意味着程序执行时间变短？

## Stage 2：多周期 CPU 与流水线

本阶段的目标是理解单周期 CPU 的性能限制，掌握多周期处理器、经典五级流水线、流水线冒险及其处理机制，并基于 Stage 1 的 EduCPU v1，将其逐步改造为一个支持流水线执行的 RISC-V CPU。同时学习如何利用 CPI、IPC、Stall Cycle 和 Branch Prediction 等指标进行性能量化分析。

### 1. 理论与源码学习

请结合以下文档、教材和 gem5 源码进行学习：

**《一生一芯》：**

1. [B2：时序分析和优化](https://ysyx.oscc.cc/docs/2607/b/2.html)
2. [B3：性能优化和简易缓存](https://ysyx.oscc.cc/docs/2607/b/3.html)
3. [B4：流水线处理器](https://ysyx.oscc.cc/docs/2607/b/4.html)

**《计算机体系结构：量化研究方法》：**

1. Appendix C：Pipelining — Basic and Intermediate Concepts。
2. Chapter 1：Fundamentals of Quantitative Design and Analysis。
3. Chapter 3：Instruction-Level Parallelism and Its Exploitation（流水线依赖与分支预测相关部分）。

**gem5 文档与源码：**

1. [SimpleCPU Models](https://www.gem5.org/documentation/general_docs/cpu_models/SimpleCPU)
2. [MinorCPU Model](https://www.gem5.org/documentation/general_docs/cpu_models/minor_cpu)
3. [gem5 TimingSimpleCPU 源码](https://gem5.googlesource.com/public/gem5/+/refs/heads/develop/src/cpu/simple/timing.cc)
4. [gem5 MinorCPU Pipeline 源码](https://github.com/gem5/gem5/blob/stable/src/cpu/minor/pipeline.cc)
5. [gem5 MinorCPU Decode 源码](https://github.com/gem5/gem5/blob/stable/src/cpu/minor/decode.cc)
6. [gem5 MinorCPU Execute 源码](https://github.com/gem5/gem5/blob/stable/src/cpu/minor/execute.cc)

源码阅读重点：

- TimingSimpleCPU 如何处理指令执行、访存等待及事件调度。
- MinorCPU 如何组织 Fetch、Decode 和 Execute 等流水线阶段。
- MinorCPU 的 Pipeline 如何协调各阶段的执行。
- 流水线阶段之间如何传递指令与状态。
- 如何使用 Buffer、Latch 和 Stall 处理流水线执行。
- 如何处理分支跳转、流水线清空及执行状态更新。

注意：MinorCPU 并非教材中的标准五级流水线 CPU。其内部使用 Fetch1、Fetch2、Decode、Execute 等模块组织流水线，阅读源码的目的是理解 gem5 的流水线建模方法，而不是直接照搬它的结构。

### 2. 实践任务：EduCPU v2

基于 Stage 1 的 EduCPU v1，将单周期 CPU 逐步扩展为 **多周期 CPU 和经典五级流水线 CPU**。

**任务 1：实现多周期 CPU**

将单周期 CPU 的执行过程拆分为多个状态。

至少包含：

- Instruction Fetch。
- Instruction Decode。
- Execute。
- Memory Access。
- Writeback。
- CPU 控制状态机（FSM）。
- 可配置的执行阶段延迟。
- 可配置的访存等待周期。

要求能够根据不同指令类型执行不同数量的周期，并正确维护 PC、寄存器以及存储器状态。

**任务 2：实现经典五级流水线 CPU**

将多周期 CPU 改造成能够同时处理多条指令的流水线 CPU。

实现以下五个流水线阶段：

- IF：Instruction Fetch。
- ID：Instruction Decode / Register Read。
- EX：Execute / Address Calculation。
- MEM：Memory Access。
- WB：Writeback。

新增流水线寄存器：

- IF/ID。
- ID/EX。
- EX/MEM。
- MEM/WB。

要求各阶段能够在同一个模拟周期内执行不同指令，并在时钟边沿统一更新流水线状态。

实现每个阶段的有效标志（Valid）、停顿控制（Stall）和气泡（Bubble）机制。

**任务 3：实现流水线冒险检测与处理**

至少实现：

- Structural Hazard Detection。
- Data Hazard Detection。
- RAW Dependency Detection。
- Forwarding / Bypassing。
- Load-Use Stall。
- Pipeline Bubble。
- Pipeline Stall。
- Pipeline Flush。

对于基础五级顺序流水线，重点研究 RAW 相关。WAR 和 WAW 需要理解其原理，但在典型的按序读写寄存器组织下，不会像 RAW 一样成为主要的数据冒险。

**任务 4：实现简单分支预测**

至少实现：

- Always Not Taken。
- 1-bit Branch Predictor。
- 2-bit Saturating Counter Predictor。
- Branch Target Calculation。
- Branch Misprediction Detection。
- Pipeline Flush / Redirect。

支持配置不同的分支预测策略，并能够记录分支预测结果和错误预测惩罚。

不要求在本阶段实现复杂的全局历史预测器或现代商业处理器级别的分支预测系统。

**任务 5：完善流水线 Trace 与性能计数器**

新增：

- Pipeline Stage Trace。
- 每周期各流水线阶段的指令信息。
- Pipeline Valid / Stall / Bubble 状态。
- Retired Instruction Count。
- Pipeline Stall Cycles。
- Data Hazard Stall Cycles。
- Structural Hazard Stall Cycles。
- Branch Misprediction Count。
- Branch Flush Cycles。

要求能够输出一段指令在流水线中逐周期推进的记录，并据此检查流水线行为是否正确。

**任务 6：Benchmark 与正确性验证**

继续使用 Stage 1 的 Benchmark，并新增：

- Dependency Chain：连续 RAW 依赖。
- Independent Instructions：相互独立的指令。
- Load-Use：Load 后立即使用结果。
- Branch Loop：循环分支。
- Branch Pattern：不同分支模式。
- Mixed Instructions：算术、访存和分支混合。

要求单周期、多周期和流水线 CPU 在相同程序下产生一致的体系结构执行结果。

可以使用独立 RISC-V 参考模型进行差分验证。

### 3. 量化分析实验

完成以下实验：

1. **单周期、多周期与流水线性能比较**：在相同 ISA 和 Benchmark 下，比较 Instruction Count、Cycle、CPI、IPC、Clock Period 和 Execution Time，并明确不同设计的时钟周期假设。

2. **流水线吞吐率分析**：研究流水线填充、排空和程序长度对 IPC 的影响，比较理论吞吐率与实际吞吐率。

3. **数据冒险与 Forwarding 分析**：分别关闭和开启 Forwarding，统计 RAW、Load-Use 等依赖造成的 Stall，分析 Forwarding 对 CPI 的改善。

4. **分支预测性能分析**：比较不同预测策略的 Prediction Accuracy、Misprediction Count、Flush Cycles、CPI 和 Execution Time。

5. **流水线深度与关键路径分析**：通过假设组合逻辑延迟及流水线寄存器开销，研究不同流水线划分方式对时钟周期、指令延迟和吞吐率的影响。

6. **CPI Breakdown 分析**：建立流水线 CPI 分解模型，分析数据相关、结构冲突和控制相关对整体性能的影响。统计类别必须避免重复计数，并与总周期数进行一致性校验。

7. **访存延迟敏感性分析**：调整教学模型中的访存等待周期，观察流水线停顿和性能变化，分析为什么提高执行流水线吞吐率不一定能够提高整个系统的性能。

### 4. 完成后请回答以下问题

**多周期处理器与流水线基础**

1. 单周期 CPU 和多周期 CPU 有什么区别？为什么要将一条指令拆分为多个周期执行？
2. 多周期 CPU 中的 FSM 如何控制不同阶段的执行？
3. Pipeline（流水线）的基本原理是什么？它为什么能够提升处理器的吞吐率？
4. Instruction Latency 和 Instruction Throughput 有什么区别？
5. 五级流水线中的 IF、ID、EX、MEM、WB 分别承担什么职责？
6. Pipeline Register 的作用是什么？为什么不同流水线阶段之间必须保存状态？
7. 什么是 Pipeline Fill 和 Pipeline Drain？它们如何影响短程序的性能？
8. 为什么流水线 CPU 的理想 CPI 接近 1，而不是等于流水线级数？

**流水线冒险与执行控制**

9. 什么是 Structural Hazard、Data Hazard 和 Control Hazard？分别在什么情况下发生？
10. RAW、WAR、WAW 分别是什么？为什么经典五级顺序流水线主要关注 RAW？
11. 什么是 Data Forwarding？它如何减少数据依赖造成的流水线停顿？
12. 为什么 Load-Use Hazard 通常不能只依靠普通 ALU Forwarding 完全消除？
13. Stall、Bubble 和 Flush 有什么区别？分别在什么情况下使用？
14. 当流水线某个阶段发生 Stall 时，其他阶段应该如何处理，才能避免指令丢失或重复执行？
15. 如何处理分支跳转对流水线取指的影响？
16. 1-bit 和 2-bit Branch Predictor 有什么区别？为什么 2-bit 预测器通常能够避免单次反常结果立即翻转强预测方向？

**gem5 实现机制**

17. 如何利用 gem5 的 ClockedObject 和 Event Queue 实现多个流水线阶段的同步执行？
18. 为什么不能直接按照 C++ 函数的执行顺序依次修改各流水线阶段的状态？
19. 阅读 MinorCPU 源码，说明 Pipeline、Fetch、Decode 和 Execute 之间如何交互。
20. gem5 中如何模拟流水线的 Stall、Bubble、Flush 和阶段间延迟？
21. TimingSimpleCPU 与 MinorCPU 的执行模型有什么区别？它们分别适合研究什么问题？
22. 如何利用逐周期 Pipeline Trace 验证指令流动、数据依赖及分支冲刷的正确性？

**性能量化分析**

23. 为什么流水线处理器的 IPC 不能简单等同于理论峰值？
24. 如何建立 CPI Breakdown？怎样判断主要性能损失来自数据冒险还是控制冒险？
25. Forwarding、Branch Prediction 和 Pipeline Depth 分别通过什么机制影响性能？
26. 如果将流水线从 5 级增加到 10 级，CPU 的性能一定会提高吗？为什么？
27. 为什么减少 Stall Cycles 不一定意味着程序的实际执行时间按相同比例下降？
28. 如何根据 gem5 的统计结果判断当前流水线 CPU 的主要性能瓶颈？

---

## Stage 3：超标量与顺序多发射

本阶段的目标是理解指令级并行（Instruction-Level Parallelism，ILP）的基本原理，掌握超标量处理器（Superscalar Processor）的微架构设计方法，并基于 Stage 2 的 EduCPU v2，将单发射流水线扩展为能够在一个时钟周期内发射多条指令的顺序多发射 CPU。同时学习通过发射宽度、指令依赖、执行单元数量和资源利用率等指标进行量化性能分析。

### 1. 理论与源码学习

请结合以下文档、教材和 gem5 源码进行学习：

**《一生一芯》：**

1. [B4：流水线处理器](https://ysyx.oscc.cc/docs/2607/b/4.html)
2. [B3：性能优化和简易缓存](https://ysyx.oscc.cc/docs/2607/b/3.html)

《一生一芯》在这一阶段主要用于复习流水线吞吐、结构冲突、指令供给和性能优化思想。顺序多发射的系统理论主要结合下面的教材和 gem5 MinorCPU 学习。

**《计算机体系结构：量化研究方法》：**

1. Chapter 3：Instruction-Level Parallelism and Its Exploitation。
2. Chapter 1：Fundamentals of Quantitative Design and Analysis。
3. Appendix H：Hardware and Software for VLIW and EPIC（相关部分）。

重点学习 ILP、Multiple Issue、Static Scheduling、Pipeline Hazards、Superscalar Execution 和 VLIW。

**gem5 文档与源码：**

1. [MinorCPU Model](https://www.gem5.org/documentation/general_docs/cpu_models/minor_cpu)
2. [MinorCPU Pipeline 源码](https://github.com/gem5/gem5/blob/stable/src/cpu/minor/pipeline.cc)
3. [MinorCPU Execute 源码](https://github.com/gem5/gem5/blob/stable/src/cpu/minor/execute.cc)
4. [MinorCPU Scoreboard 源码](https://github.com/gem5/gem5/blob/stable/src/cpu/minor/scoreboard.cc)
5. [MinorCPU Functional Unit 源码](https://github.com/gem5/gem5/blob/stable/src/cpu/minor/func_unit.cc)
6. [MinorCPU 参数定义源码](https://github.com/gem5/gem5/blob/stable/src/cpu/minor/BaseMinorCPU.py)

源码阅读重点：

- MinorCPU 如何配置 Fetch、Decode、Issue 和 Commit 的宽度。
- Execute 阶段如何在一个周期内尝试发射多条指令。
- `executeIssueLimit`、`executeCommitLimit` 等参数的作用。
- Scoreboard 如何判断指令源操作数是否就绪。
- Functional Unit 如何描述支持的操作类型、执行延迟和发射间隔。
- Issue Logic 如何检查指令依赖、功能单元可用性和资源限制。
- 为什么 MinorCPU 可以支持多个执行单元和多指令发射，但仍遵循顺序发射约束。

### 2. 实践任务：EduCPU v3

基于 Stage 2 的 EduCPU v2，将单发射流水线 CPU 扩展为 **In-Order Superscalar CPU（顺序超标量处理器）**。

**任务 1：实现多指令 Fetch 与 Decode**

将原本每周期最多处理一条指令的前端，扩展为支持多条指令。

至少实现：

- 参数化 Fetch Width。
- 参数化 Decode Width。
- Instruction Fetch Buffer。
- 多指令 Decoder。
- Instruction Order Tracking。
- 基础 Fetch / Decode Stall 机制。

首先支持每周期最多处理两条指令，并将宽度作为可配置参数，为后续实验保留扩展能力。

需要保证同一周期获取的多条指令仍然具有明确的程序顺序。

**任务 2：实现多发射流水线**

将单发射 Execute 阶段扩展为支持多个执行单元。

至少实现：

- 参数化 Issue Width。
- 多个 Integer ALU。
- 独立的 Multiply Functional Unit（可使用 RV32M 子集作为扩展）。
- Load / Store Unit。
- Functional Unit Selection。
- Structural Hazard Detection。
- In-Order Issue Control。
- In-Order Commit Control。

要求至少能够在同一个周期发射两条满足依赖和资源约束的指令。

对于多周期执行单元，需要明确操作延迟、启动间隔和完成顺序，并确保结果按照设计规定的提交语义更新体系结构状态。

**任务 3：实现 Scoreboard 指令依赖管理**

新增 Scoreboard 模块，跟踪指令执行与寄存器依赖关系。

至少实现：

- Register Dependency Tracking。
- RAW Hazard Detection。
- Destination Register Busy Tracking。
- Operand Readiness Check。
- Functional Unit Availability Check。
- Issue Stall。
- 结果就绪与依赖解除。

要求正确处理同周期候选指令之间的数据依赖，以及较早指令执行尚未完成时的依赖。

对于 WAW、WAR 等相关，需要根据所选择的操作数读取、写回和按序提交机制明确约束，不能因不同执行单元的完成延迟不同而破坏指令语义。

**任务 4：实现多发射寄存器堆与写回机制**

扩展 Stage 2 的 Register File 和 Writeback。

至少实现：

- 支持多指令读取操作数。
- 多个执行单元结果的写回选择。
- Register File 读写端口约束。
- Writeback Port Arbitration。
- Commit Bandwidth 限制。
- 结果冲突与资源等待。

要求能够配置寄存器读写端口数量，并正确处理端口不足时的结构冲突。

注意：在周期级模型中，可以通过参数化端口数量和仲裁规则模拟硬件资源约束，不要求实现真正的多端口寄存器堆 RTL。

**任务 5：支持参数化 Functional Unit**

为不同类型的执行单元设置参数：

- Functional Unit Count。
- Operation Type。
- Execution Latency。
- Initiation Interval。
- Pipeline Depth。
- Issue / Result Bandwidth。

允许配置不同数量的 ALU 和 MUL 单元，研究执行单元数量及其延迟对处理器性能的影响。

**任务 6：完善多发射性能计数器**

在 EduCPU v2 的性能统计基础上新增：

- Fetch / Decode / Issue / Commit Instructions per Cycle。
- Issue Slot Utilization。
- Functional Unit Utilization。
- Dependency Stall Cycles。
- Structural Stall Cycles。
- Register Port Stall Cycles。
- Functional Unit Busy Cycles。
- Issue Width 与 Achieved IPC。
- 各类指令的执行单元占用情况。

要求能够区分指令没有被发射的原因，而不仅是统计最终 IPC。

**任务 7：Benchmark 与正确性验证**

继续使用前两个 Stage 的 Benchmark，并新增：

- Independent ALU：大量相互独立的算术指令。
- Dependency Chain：连续数据依赖。
- Mixed ALU / MUL：不同功能单元混合计算。
- Register Dependency：相同源/目标寄存器的依赖测试。
- Structural Conflict：功能单元和寄存器端口资源冲突。
- Load / Compute Mix：访存与独立计算混合。
- Instruction Scheduling：不同指令排列顺序。
- Branch-heavy：多发射情况下的分支控制。

要求不同发射宽度、不同功能单元配置下，程序最终执行结果保持一致。

同时使用 gem5 MinorCPU 进行对应的性能对照实验，不要求教学模型与 MinorCPU 具有完全相同的内部流水线结构和周期统计。

### 3. 量化分析实验

完成以下实验：

1. **发射宽度与 IPC 分析**：将 Issue Width 分别设置为 1、2、4，在相同 Benchmark 下比较 IPC、执行周期和实际加速比，分析为什么 IPC 不一定随发射宽度线性增长。

2. **指令依赖与 ILP 分析**：分别运行 Independent Instructions 和 Dependency Chain，研究数据相关对多发射吞吐率的限制，分析程序中可利用的指令级并行性。

3. **Scoreboard 与 Stall 分析**：统计寄存器依赖造成的 Issue Stall，观察不同依赖距离、指令执行延迟对 IPC 的影响。

4. **Functional Unit 资源分析**：改变 ALU、MUL、LSU 的数量、执行延迟和启动间隔，分析 Functional Unit Utilization、资源冲突和吞吐率的变化。

5. **Fetch / Decode / Issue / Commit 宽度分析**：分别改变前端宽度、发射宽度和提交宽度，寻找限制多发射处理器实际吞吐率的阶段。

6. **寄存器堆端口与写回带宽分析**：限制 Register File Read/Write Ports，研究端口数量和 Writeback Bandwidth 对多发射性能的影响。

7. **指令调度顺序分析**：在不改变程序计算结果的前提下，调整独立指令与依赖指令的排列顺序，比较顺序发射处理器的执行周期和 IPC，分析静态调度的作用。

8. **MinorCPU 对照实验**：在 gem5 原生 MinorCPU 上配置不同发射宽度及功能单元参数，利用 MinorTrace 和性能统计分析模型行为，并比较 EduCPU v3 与 MinorCPU 的设计差异。

9. **Issue Slot Utilization 分析**：计算不同 Benchmark 下的发射槽利用率，结合依赖、功能单元忙碌、前端供给不足等原因分析闲置槽位的来源。

10. **多发射扩展收益分析**：综合比较 1-wide、2-wide、4-wide 处理器在不同工作负载下的加速比，并结合执行单元数量、寄存器端口和调度复杂度，评价继续增加发射宽度是否合理。

### 4. 完成后请回答以下问题

**超标量与指令级并行**

1. 什么是 Instruction-Level Parallelism（ILP）？它与流水线并行有什么关系？
2. 什么是 Superscalar Processor？它与普通单发射流水线 CPU 有什么区别？
3. Fetch Width、Decode Width、Issue Width、Execute Width 和 Commit Width 分别代表什么？
4. 什么是 In-Order Issue？它与 In-Order Commit 有什么区别？
5. 为什么 Superscalar CPU 可以在一个周期内发射多条指令？
6. 为什么 Issue Width=4 并不意味着 IPC 一定能够达到 4？
7. 什么是静态指令调度（Static Scheduling）？为什么指令排列顺序可能影响执行性能？
8. VLIW 与 Superscalar 在指令调度责任、硬件复杂度和编译器要求方面有什么区别？

**指令依赖与 Scoreboard**

9. 什么是 Instruction Dependency Graph？如何根据一段 RISC-V 汇编分析指令之间的依赖关系？
10. 什么是 Scoreboard？它在顺序多发射处理器中承担什么职责？
11. Scoreboard 如何判断一条指令是否可以发射？
12. 当同一周期准备发射的两条指令存在 RAW 依赖时，处理器应该如何处理？
13. 为什么多发射处理器需要检查同一个发射组内的指令依赖？
14. 在不同功能单元执行延迟不同的情况下，为什么需要考虑 WAW、WAR 和写回顺序问题？
15. 为什么一条等待操作数的较早指令可能阻止后面相互独立的指令发射？
16. 顺序多发射处理器有哪些无法通过增加 Functional Unit 数量解决的性能瓶颈？

**执行单元与微架构设计**

17. Functional Unit Latency 和 Initiation Interval 有什么区别？
18. 为什么一个执行单元能够每周期接受新指令，不代表它只需要一个周期就能完成指令？
19. 为什么多发射 CPU 需要更多的寄存器读取端口和写回带宽？
20. Register File Port 数量如何限制处理器的发射能力？
21. 为什么增加 ALU 数量不一定提高 IPC？
22. 为什么 Fetch / Decode / Issue / Commit 中任何一个阶段都可能成为性能瓶颈？
23. 当处理器包含不同延迟的执行单元时，如何保证指令提交结果符合 ISA 语义？

**gem5 实现机制**

24. 阅读 MinorCPU 源码，分析其如何实现指令的顺序发射。
25. MinorCPU 中的 Scoreboard 如何跟踪寄存器依赖？
26. MinorCPU 的 Functional Unit 如何描述支持的操作类型、执行延迟和启动间隔？
27. `executeIssueLimit` 与 `executeCommitLimit` 分别限制什么？修改它们为什么不一定能够提升 IPC？
28. 如何利用 gem5 的 MinorTrace 判断一条指令为什么没有成功发射？
29. EduCPU v3 与 gem5 MinorCPU 在流水线组织、资源建模和性能精度方面有什么差异？

**性能量化分析**

30. 如何计算 Issue Slot Utilization？它与 IPC 有什么区别？
31. 如何通过性能统计区分 Data Dependency Stall、Structural Stall 和 Frontend Bottleneck？
32. 为什么 Independent ALU Benchmark 的 IPC 通常高于 Dependency Chain Benchmark？
33. 如何判断当前处理器应该优先增加 Issue Width、Functional Unit，还是改善指令供给？
34. 为什么 2-wide 处理器可能接近两倍加速，而 4-wide 处理器的提升却非常有限？
35. 如果要设计一个高性能顺序多发射 CPU，应该如何通过 gem5 实验确定合理的发射宽度与执行单元配置？
36. 顺序多发射处理器的根本性能局限是什么？为什么现代高性能 CPU 需要进一步引入乱序执行（Out-of-Order Execution）？
## Stage 4：乱序执行与动态调度

本阶段的目标是理解顺序多发射处理器的性能限制，掌握动态指令调度、寄存器重命名、乱序执行及按序提交等核心微架构机制，并基于 Stage 3 的 EduCPU v3，实现一个简化的乱序执行 CPU。同时结合 gem5 O3CPU 的源码与性能实验，理解现代高性能处理器如何挖掘指令级并行（ILP）和内存级并行（MLP）。

### 1. 理论与源码学习

请结合以下文档、教材和 gem5 源码进行学习：

**《计算机体系结构：量化研究方法》：**

1. Chapter 3：Instruction-Level Parallelism and Its Exploitation。
2. Chapter 5：Thread-Level Parallelism（Hardware Multithreading 和 SMT 相关内容）。
3. Chapter 1：Fundamentals of Quantitative Design and Analysis。

**gem5 文档与源码：**

1. [gem5 O3CPU Model](https://www.gem5.org/documentation/general_docs/cpu_models/O3CPU)
2. [O3CPU Pipeline](https://github.com/gem5/gem5/tree/stable/src/cpu/o3)
3. [Register Rename](https://github.com/gem5/gem5/blob/stable/src/cpu/o3/rename.cc)
4. [Rename Map](https://github.com/gem5/gem5/blob/stable/src/cpu/o3/rename_map.cc)
5. [Reorder Buffer](https://github.com/gem5/gem5/blob/stable/src/cpu/o3/rob.cc)
6. [Instruction Queue](https://github.com/gem5/gem5/blob/stable/src/cpu/o3/inst_queue.cc)
7. [Issue / Execute / Writeback](https://github.com/gem5/gem5/blob/stable/src/cpu/o3/iew.cc)
8. [Load / Store Queue](https://github.com/gem5/gem5/blob/stable/src/cpu/o3/lsq.cc)
9. [Commit](https://github.com/gem5/gem5/blob/stable/src/cpu/o3/commit.cc)
10. [O3CPU 参数定义](https://github.com/gem5/gem5/blob/stable/src/cpu/o3/BaseO3CPU.py)

源码阅读重点：

- O3CPU 中 Fetch、Decode、Rename、IEW、Commit 的整体组织。
- Register Renaming 如何消除 WAR/WAW 假相关。
- ROB、IQ、RS、LSQ 分别承担什么职责。
- Instruction Queue 如何判断指令就绪并选择指令发射。
- 执行完成与指令提交之间的区别。
- 分支预测失败、异常和推测执行恢复机制。
- ROB、IQ、LSQ 和物理寄存器容量如何限制处理器性能。

### 2. 实践任务：EduCPU v4

基于 Stage 3 的 EduCPU v3，将顺序多发射 CPU 扩展为 **Out-of-Order Superscalar CPU**。

**任务 1：实现动态调度与寄存器重命名**

至少实现：

- Register Alias Table。
- Physical Register File。
- Free List。
- Reservation Station / Instruction Queue。
- Operand Readiness Tracking。
- Wakeup / Select。
- Out-of-Order Issue。

要求能够让后续已经就绪的独立指令越过等待中的指令，提前进入执行单元。

**任务 2：实现 ROB 与乱序执行控制**

至少实现：

- Reorder Buffer。
- 指令顺序分配与按序提交。
- 多执行单元乱序完成。
- 简化 Load / Store Queue。
- Branch Misprediction Recovery。
- 精确体系结构状态更新。
- 基本资源占用与 Stall 统计。

要求指令允许乱序执行，但最终体系结构状态必须与顺序执行参考模型保持一致。

对异常和推测恢复可以采用简化模型，但必须明确其语义和覆盖范围。

**任务 3：使用 gem5 O3CPU 进行对照研究**

使用 gem5 原生 O3CPU 运行 Stage 3 的 Benchmark，并增加 OoO 专项测试。

要求能够配置：

- Fetch / Decode / Issue / Commit Width。
- ROB Entries。
- IQ Entries。
- Physical Register Count。
- LQ / SQ Entries。
- Functional Unit Latency。

同时增加 ROB Occupancy、IQ Occupancy、Rename Stall、Issue Stall、Load Stall、Commit Stall 等性能统计。

不要求 EduCPU v4 完整复刻 O3CPU；重点是实现动态调度和按序提交的核心原理。

### 3. 量化分析实验

1. **In-Order 与 OoO 性能比较**：在相同 Benchmark 和硬件资源约束下比较 IPC、执行周期和加速比。
2. **ROB 容量分析**：改变 ROB Entries，研究 ROB 容量对 ILP、IPC 和提交停顿的影响。
3. **Instruction Queue 分析**：改变 IQ/RS 容量，研究指令就绪率、发射效率和资源竞争。
4. **Register Renaming 分析**：改变物理寄存器数量，研究寄存器资源不足对 Rename Stall 和 IPC 的影响。
5. **Load Latency 与 MLP 分析**：使用相互独立的 Load 指令，研究乱序执行隐藏访存延迟的能力。
6. **Issue Width 与 ROB 联合分析**：分析发射宽度和指令窗口容量之间的关系。
7. **Branch Recovery 分析**：研究分支预测错误、错误路径指令和流水线恢复对性能的影响。
8. **OoO 瓶颈分析**：根据性能统计，判断限制 IPC 的主要因素是前端、调度窗口、执行单元、访存还是提交资源。

### 4. 完成后请回答以下问题

**动态调度与乱序执行**

1. 什么是 Dynamic Scheduling？它与静态调度有什么区别？
2. 为什么顺序多发射处理器需要进一步引入乱序执行？
3. Tomasulo Algorithm 的基本原理是什么？
4. Scoreboard 与 Tomasulo 的主要区别是什么？
5. 什么是 Register Renaming？为什么它能够消除 WAR/WAW 假相关？
6. Architectural Register 与 Physical Register 有什么区别？
7. Reservation Station 和 Instruction Queue 分别承担什么职责？
8. Wakeup / Select 是什么？如何判断一条指令已经可以发射？

**ROB、LSQ 与推测执行**

9. 什么是 Reorder Buffer？为什么需要 ROB？
10. 为什么指令可以乱序执行，但通常仍然需要按序提交？
11. 什么是 Precise Exception？ROB 如何帮助实现精确异常？
12. LSQ 的主要职责是什么？
13. 为什么 Load/Store 之间的内存依赖比普通寄存器依赖更复杂？
14. 什么是 Speculative Execution？分支预测失败后需要恢复哪些状态？
15. ROB、IQ、LSQ 和 Physical Register 数量之间有什么资源约束关系？

**gem5 实现机制**

16. gem5 O3CPU 的 Rename、IEW、Commit 分别执行哪些操作？
17. O3CPU 如何跟踪指令依赖并进行动态发射？
18. O3CPU 如何处理不同执行单元的执行延迟？
19. gem5 如何保证乱序执行后的体系结构状态正确？
20. 如何在 O3CPU 中配置 ROB、IQ、LSQ 和 Issue Width？
21. 如何利用 gem5 统计信息定位 Rename Stall、Issue Stall 和 Commit Stall？

**性能量化分析**

22. 为什么 OoO CPU 能够比 In-Order CPU 更好地隐藏部分访存延迟？
23. 为什么增加 ROB 容量可能提高 IPC？为什么收益会逐渐下降？
24. 为什么增加 Issue Width 不一定带来相应的 IPC 提升？
25. 如何区分 ILP 不足、MLP 不足与执行资源不足？
26. 如果一个 Benchmark 的 IPC 很低，应该如何设计实验判断具体瓶颈？
27. OoO 机制引入了哪些硬件复杂度和资源代价？在什么情况下这些代价可能不值得？

---

## Stage 5：存储体系结构与多核系统

本阶段的目标是理解计算机存储层次的组织方式，掌握 Cache、DRAM、虚拟内存、多核共享存储、一致性协议及片上互联的基本原理，并基于已有 EduArch-gem5 工程，建立能够开展存储延迟、带宽、并行访存和资源竞争研究的存储系统实验平台。

### 1. 理论与源码学习

请结合以下文档、教材和 gem5 源码进行学习：

**《一生一芯》：**

1. [B3：性能优化和简易缓存](https://ysyx.oscc.cc/docs/2607/b/3.html)
2. A 阶段中关于 Cache、虚拟内存及高级性能优化的相关内容。

**《计算机体系结构：量化研究方法》：**

1. Chapter 2：Memory Hierarchy Design。
2. Chapter 5：Thread-Level Parallelism。
3. Appendix B：Review of Memory Hierarchy。
4. Appendix F：Interconnection Networks。
5. Appendix L：Virtual Memory and Address Translation。

**gem5 文档与源码：**

1. [gem5 Memory System](https://www.gem5.org/documentation/general_docs/memory_system/)
2. [Classic Caches](https://www.gem5.org/documentation/general_docs/memory_system/classic_caches/)
3. [Creating SimObjects in the Memory System](https://www.gem5.org/documentation/learning_gem5/part2/memoryobject/)
4. [Creating a Simple Cache](https://www.gem5.org/documentation/learning_gem5/part2/simplecache/)
5. [Ruby Memory System](https://www.gem5.org/documentation/general_docs/ruby/)
6. [Introduction to Ruby](https://www.gem5.org/documentation/learning_gem5/part3/MSIintro/)
7. [Garnet 2.0](https://www.gem5.org/documentation/general_docs/ruby/garnet-2/)
8. [Classic Cache 源码](https://github.com/gem5/gem5/tree/stable/src/mem/cache)
9. [Memory Controller 源码](https://github.com/gem5/gem5/tree/stable/src/mem)
10. [Ruby / Garnet 源码](https://github.com/gem5/gem5/tree/stable/src/mem/ruby)

源码阅读重点：

- gem5 的 Request、Packet、RequestPort、ResponsePort。
- Timing Request / Response 的执行流程。
- Cache Hit / Miss、MSHR、Write Buffer 的实现。
- Memory Controller 如何处理访存请求和队列。
- Ruby 如何通过控制器和状态机描述一致性协议。
- Garnet 中 Router、Link、Virtual Channel 和 Flow Control 的组织。

### 2. 实践任务：EduMemory

基于已有 EduCPU 和 gem5 原生处理器模型，建设 **EduMemory：Parameterized Memory System**。

**任务 1：实现参数化 Cache / DRAM 系统**

至少支持：

- L1 Instruction / Data Cache。
- L2 Cache。
- Cache Size / Associativity / Line Size。
- Cache Replacement Policy。
- MSHR / Write Buffer。
- DRAM Channel / Bank / Latency / Bandwidth。
- Memory Request / Response 与 Backpressure。

优先复用 gem5 Classic Memory System，而不是从零重新实现复杂 Cache。

**任务 2：构建多核与一致性实验系统**

至少完成：

- 双核或四核系统配置。
- Private L1 / Shared L2。
- 共享内存访问。
- Cache Coherence。
- False Sharing。
- Ruby 一致性协议实验。
- 多核心共享带宽竞争。

该任务可以使用 gem5 原生 CPU 模型，不要求 EduCPU v4 自己实现完整的虚拟内存和多核一致性支持。

**任务 3：构建片上互联实验环境**

基于 gem5 Ruby / Garnet，完成：

- 基础 Mesh Network 配置。
- Router / Link。
- 可配置网络拓扑及链路带宽。
- Traffic Generator。
- Network Latency / Throughput Statistics。
- 基础拥塞与流控分析。

注意 Classic Memory System 与 Ruby/Garnet 需要分别建立兼容的模拟配置，不能直接将任意 Classic Cache 和 Ruby Cache 混合使用。

### 3. 量化分析实验

1. **Cache 容量与性能分析**：改变 L1/L2 Cache 容量，研究 Miss Rate、MPKI、CPI 和执行时间。
2. **Cache 组织分析**：改变 Associativity、Line Size 和 Replacement Policy，分析不同访问模式下的性能。
3. **MSHR 与 MLP 分析**：改变 MSHR 数量，研究并发访存请求与处理器实际性能的关系。
4. **DRAM Latency / Bandwidth 分析**：分别改变延迟和带宽，分析两者对不同 Benchmark 的影响。
5. **访存模式分析**：比较 Sequential、Random、Stride 和 Pointer Chasing。
6. **TLB 分析**：在支持地址翻译的模型中，研究 TLB 容量和 Miss 对访存性能的影响。
7. **多核资源竞争分析**：改变核心数量及共享存储带宽，分析系统吞吐与扩展性。
8. **Cache Coherence 分析**：研究 False Sharing 和共享数据访问引起的一致性流量。
9. **NoC 性能分析**：改变网络规模、链路带宽、流量模式和注入率，分析网络延迟、吞吐与拥塞。

### 4. 完成后请回答以下问题

**存储层次与 Cache**

1. 为什么计算机需要 Memory Hierarchy？
2. 什么是 Temporal Locality 和 Spatial Locality？
3. Cache 的 Direct-Mapped、Set-Associative 和 Fully-Associative 有什么区别？
4. 什么是 Cache Hit、Miss、Miss Rate 和 MPKI？
5. Write-Through 和 Write-Back 有什么区别？
6. 什么是 MSHR？为什么 Non-Blocking Cache 需要 MSHR？
7. 为什么增加 Cache 容量不一定显著改善性能？
8. Cache Miss Rate 与执行时间之间是什么关系？

**DRAM 与内存级并行**

9. DRAM 中 Channel、Rank、Bank、Row Buffer 分别是什么？
10. Latency 与 Bandwidth 有什么区别？
11. Memory-Level Parallelism（MLP）是什么？
12. 为什么有较高理论带宽的系统，实际有效带宽可能很低？
13. 如何利用 Little's Law 估算隐藏存储延迟所需要的并发请求数量？
14. Memory Controller 的队列与调度策略如何影响性能？

**虚拟内存、多核与一致性**

15. Virtual Address 与 Physical Address 有什么区别？
16. TLB 的主要作用是什么？TLB Miss 会带来什么开销？
17. Cache Coherence 与 Memory Consistency 有什么区别？
18. MESI 协议中的 M、E、S、I 分别代表什么？
19. 什么是 False Sharing？它为什么影响多核程序性能？
20. 为什么多个 CPU Core 同时访问内存时可能出现性能下降？

**gem5 实现机制**

21. gem5 的 Timing Request / Response 是如何完成的？
22. `sendTimingReq()`、`recvTimingResp()` 和 Retry 机制分别有什么作用？
23. gem5 Classic Memory 与 Ruby Memory 有什么区别？
24. Ruby 为什么需要使用一致性状态机？
25. Garnet 如何描述 Router、Link、Virtual Channel 和 Flow Control？

**性能量化分析**

26. 如何判断一个工作负载主要受到访存延迟还是内存带宽限制？
27. 如何分析 Cache Miss、MSHR 和 DRAM Queue 对 CPI 的贡献？
28. 如何判断增加 Cache、内存通道或并发访存能力哪个更有效？
29. 如何量化多核共享资源竞争带来的性能损失？
30. 为什么一个拥有更多计算单元的系统可能因为存储瓶颈而性能下降？

---

## Stage 6：SIMD 与向量体系结构

本阶段的目标是理解数据级并行（Data-Level Parallelism，DLP）的基本原理，掌握 SIMD、Vector ISA、向量执行单元以及向量访存等核心机制，并基于 EduArch-gem5 工程实现一个简化向量计算引擎，为后续 GPU SIMT 和数据流 PE 阵列的学习奠定基础。

### 1. 理论与源码学习

请结合以下文档、教材和相关源码进行学习：

**《计算机体系结构：量化研究方法》：**

1. Chapter 4：Data-Level Parallelism in Vector, SIMD, and GPU Architectures。
2. Appendix G：Vector Processors。
3. Appendix J：Computer Arithmetic。
4. Chapter 7：Domain-Specific Architectures（相关部分）。

**RISC-V Vector 文档：**

1. [RISC-V Vector Extension Specification v1.0](https://docs.riscv.org/reference/isa/unpriv/v-st-ext)
2. [RISC-V Vector C Intrinsics Specification](https://docs.riscv.org/reference/vector-c-intrinsics/v1.0/index.html)

**gem5 文档与源码：**

1. [gem5 ISA Implementation](https://www.gem5.org/documentation/general_docs/architecture_support/isa_parser/)
2. [gem5 RISC-V 架构源码](https://github.com/gem5/gem5/tree/stable/src/arch/riscv)
3. [gem5 GPU Vector Register File](https://github.com/gem5/gem5/blob/stable/src/gpu-compute/vector_register_file.cc)
4. [gem5 GPU Compute Unit](https://github.com/gem5/gem5/blob/stable/src/gpu-compute/compute_unit.cc)

需要注意，GPU 中的 Vector Register File 可以作为研究并行寄存器组织的参考，但并不等价于完整的 RISC-V Vector Processor。

本阶段的 EduVector 使用教学用向量指令或任务接口，不要求直接实现完整的 RVV ISA。

源码阅读重点：

- 向量寄存器与普通标量寄存器的组织区别。
- 向量执行单元如何处理多个数据元素。
- Vector Lane、Mask 和 Reduction 的基本实现方式。
- 向量执行单元与存储系统之间的数据传输。
- 向量执行宽度和计算吞吐率之间的关系。

### 2. 实践任务：EduVector

基于已有 EduArch-gem5 工程，实现 **EduVector：Parameterized Vector Engine**。

**任务 1：实现向量计算单元**

至少实现：

- Vector Register File。
- Vector Instruction / Command Decoder。
- 参数化 SIMD Lane。
- Vector ALU。
- Vector MAC。
- Vector Length。
- Masked Execution。
- 向量指令执行控制与状态更新。

要求能够执行 Vector Add、Vector Multiply 和 Dot Product。

**任务 2：实现向量访存与归约**

至少实现：

- Vector Load / Store。
- Consecutive / Strided Access。
- Gather / Scatter（简化实现）。
- Reduction。
- Vector Memory Request。
- 数据尾部处理（Tail Handling）。

要求能够运行向量加法、点积、归约和简单 GEMV。

**任务 3：完善向量性能模型**

新增：

- Vector Instruction Count。
- Active Lane Count。
- Lane Utilization。
- Vector Execution Cycles。
- Memory Transfer Bytes。
- Achieved Operations per Cycle。
- Arithmetic Intensity。
- Effective Bandwidth。

支持配置不同 Vector Length、Lane Count、执行延迟和访存带宽。

### 3. 量化分析实验

1. **Scalar 与 Vector 性能比较**：比较相同数学计算在标量和向量架构中的指令数、周期数和吞吐。
2. **SIMD Lane 数量分析**：改变 Lane Count，研究实际加速比与硬件利用率。
3. **Vector Length 分析**：改变 Vector Length，研究短向量和长向量的执行效率。
4. **Mask 与 Tail 分析**：研究数据规模不能整除向量宽度时的计算资源浪费。
5. **Vector Memory 分析**：比较连续、跨步和不规则访存。
6. **Reduction 分析**：研究不同归约组织对延迟和吞吐的影响。
7. **Roofline 分析**：建立不同数据规模下的理论性能上限，并与 gem5 模拟结果对照。
8. **数据精度分析**：比较不同数据宽度对访存流量和理论计算吞吐的影响。

### 4. 完成后请回答以下问题

**SIMD 与向量体系结构**

1. 什么是 Data-Level Parallelism（DLP）？
2. SIMD、Vector 和 Superscalar 有什么区别？
3. Vector ISA 与普通标量 ISA 的主要区别是什么？
4. Vector Register、Vector Length、Vector Lane 分别是什么？
5. RISC-V Vector 中 VLEN、SEW、LMUL、VL 分别是什么？
6. 为什么同一条向量指令能够对多个数据元素进行计算？
7. 什么是 Strip Mining？为什么向量程序需要这种技术？
8. Masked Execution 的作用是什么？

**向量微架构**

9. 为什么增加 SIMD Lane 不一定带来线性加速？
10. Vector Functional Unit 的吞吐率由哪些因素决定？
11. 为什么 Reduction 通常需要特殊的硬件组织？
12. Vector Chaining 的作用是什么？
13. Vector Register File 的带宽如何限制执行单元吞吐？
14. 为什么 Vector Load / Store 对存储带宽有较高要求？
15. Gather / Scatter 与连续访存有什么区别？

**gem5 实现机制**

16. 如何在 gem5 中描述一个参数化的 SIMD Lane 阵列？
17. 如何模拟不同 Vector Instruction 的执行周期？
18. 如何处理一条向量指令的多个数据元素及其完成条件？
19. 如何模拟向量执行单元与存储系统之间的请求、等待和背压？

**性能量化分析**

20. 如何计算向量执行单元的理论峰值计算吞吐？
21. 如何计算 Lane Utilization？
22. 什么是 Arithmetic Intensity？如何计算？
23. Roofline Model 如何判断计算瓶颈与带宽瓶颈？
24. 为什么向量宽度增加后，性能可能逐渐受到访存限制？
25. 为什么 GEMV 与 GEMM 的 Arithmetic Intensity 通常不同？
26. 如果设计一个高吞吐向量计算单元，应该如何确定 Lane Count 与 Memory Bandwidth？

---

## Stage 7：GPU 架构与 SIMT

本阶段的目标是理解现代 GPU 的程序执行模型和微架构设计，掌握 SIMT、Warp/Wavefront、线程调度、分支分歧、延迟隐藏及 GPU 存储层次等核心机制，并基于已有工程实现一个简化的 GPU Compute Unit。

### 1. 理论与源码学习

请结合以下文档、教材和 gem5 源码进行学习：

**《计算机体系结构：量化研究方法》：**

1. Chapter 4：Data-Level Parallelism in Vector, SIMD, and GPU Architectures。
2. Chapter 5：Thread-Level Parallelism。
3. Chapter 2：Memory Hierarchy Design（GPU 存储相关内容）。

**GPU 编程与体系结构文档：**

1. [CUDA C++ Programming Guide](https://docs.nvidia.com/cuda/cuda-c-programming-guide/)
2. [AMD ROCm Documentation](https://rocm.docs.amd.com/)
3. [HIP Programming Model](https://rocm.docs.amd.com/projects/HIP/en/latest/understand/programming_model.html)

CUDA 资料主要用于理解 Grid、Block、Thread、Warp 等编程模型，gem5 GPU Full System 实验则使用 AMD GPU 模型和兼容的 HIP/ROCm 环境。

**gem5 文档与源码：**

1. [gem5 AMD GPU Full System](https://www.gem5.org/documentation/general_docs/gpu_models/gpufs)
2. [GPU Compute Unit](https://github.com/gem5/gem5/blob/stable/src/gpu-compute/compute_unit.cc)
3. [Wavefront](https://github.com/gem5/gem5/blob/stable/src/gpu-compute/wavefront.cc)
4. [GPU Scheduler](https://github.com/gem5/gem5/blob/stable/src/gpu-compute/scheduler.cc)
5. [GPU Schedule Stage](https://github.com/gem5/gem5/blob/stable/src/gpu-compute/schedule_stage.cc)
6. [GPU Scoreboard](https://github.com/gem5/gem5/blob/stable/src/gpu-compute/scoreboard_check_stage.cc)
7. [GPU Global Memory Pipeline](https://github.com/gem5/gem5/blob/stable/src/gpu-compute/global_memory_pipeline.cc)
8. [GPU Local Memory Pipeline](https://github.com/gem5/gem5/blob/stable/src/gpu-compute/local_memory_pipeline.cc)
9. [GPU 模型参数](https://github.com/gem5/gem5/blob/stable/src/gpu-compute/GPU.py)

源码阅读重点：

- GPU Compute Unit 的基本组织。
- Wavefront 如何维护线程状态及执行状态。
- GPU Scheduler 如何选择就绪 Wavefront。
- Scoreboard 如何处理线程指令依赖。
- SIMD 执行单元与 Wavefront 的关系。
- Global / Local Memory Pipeline 的运行机制。
- GPU 中如何实现访存等待与线程切换。

### 2. 实践任务：EduSIMT

基于 Stage 6 的向量执行思想，实现 **EduSIMT：Educational GPU Compute Unit**。

**任务 1：实现 SIMT Core**

至少实现：

- Compute Unit。
- Warp / Wavefront。
- Thread Context。
- SIMD Execution Lanes。
- Active Mask。
- SIMT Instruction Execution。
- Branch Divergence。
- Re-convergence。

要求能够执行包含多个线程的简单 Kernel，并正确处理线程分支及同步执行语义。

**任务 2：实现 Warp 调度与资源管理**

至少实现：

- Warp Scheduler。
- Ready / Waiting Warp Queue。
- Scoreboard。
- Round-Robin Scheduling。
- 访存等待和 Warp Switching。
- Register Resource Allocation。
- Maximum Resident Warps。
- 基础 Barrier Synchronization。

要求在一个 Warp 等待访存时，调度器能够选择其他就绪 Warp 执行。

**任务 3：实现 GPU 存储访问模型**

至少实现：

- Global Memory。
- Shared / Local Memory。
- Memory Coalescing。
- Bank Conflict。
- Memory Request Queue。
- 基础访存背压。
- Memory Transaction Statistics。

支持 Vector Add、Reduction、简单 GEMM 及不同访问模式的 Kernel。

**任务 4：使用 gem5 GPUFS 进行真实 Kernel 实验**

使用 gem5 官方支持的 AMD GPUFS 配置运行 HIP Kernel。

至少完成：

- GPUFS 基础环境配置。
- 简单 Kernel 运行。
- GPU Debug Trace。
- GPU Performance Statistics。
- 与 EduSIMT 教学模型进行机制层面的对照。

GPUFS 需要对应的 x86 系统配置、GPU 模型及 ROCm 软件栈。该实验作为课程工程中的独立配置，不要求直接使用 EduCPU 代替 GPUFS 的 Host CPU。

### 3. 量化分析实验

1. **Warp 数量与吞吐分析**：改变 Resident Warp 数量，分析执行吞吐率和延迟隐藏效果。
2. **Warp Scheduling 分析**：比较调度策略对计算单元利用率和访存等待的影响。
3. **Branch Divergence 分析**：改变不同线程的分支行为，分析 Active Lane Efficiency。
4. **Occupancy 分析**：研究寄存器资源、Shared Memory 和 Warp 数量对 Occupancy 的影响。
5. **Memory Coalescing 分析**：比较连续、跨步和不规则访存的 Memory Transaction 数量。
6. **Shared Memory Bank Conflict 分析**：研究数据访问模式与 Bank Conflict 的关系。
7. **GPU Kernel 性能分析**：运行 Vector Add、Reduction、GEMM 等 Kernel，统计计算吞吐、带宽和执行时间。
8. **CPU OoO 与 GPU SIMT 对比**：分析动态指令调度和 Warp Scheduling 在隐藏执行及访存延迟方面的不同机制。

### 4. 完成后请回答以下问题

**GPU 执行模型**

1. GPU Kernel 从 Host 提交到 GPU 执行完成经历哪些步骤？
2. Grid、Block、Thread、Warp/Wavefront 分别是什么？
3. GPU Compute Unit 与 CPU Core 有什么区别？
4. SIMD 与 SIMT 的区别是什么？两者之间有什么联系？
5. 为什么 GPU 能够支持大量并行线程？
6. GPU 为什么通常更重视吞吐率，而不是单条指令的最低延迟？

**SIMT 与 Warp Scheduling**

7. 什么是 Warp Scheduler？它的职责是什么？
8. GPU 如何判断一个 Warp 是否可以继续执行？
9. 什么是 Warp-Level Parallelism？
10. 为什么 GPU 能够通过 Warp Switching 隐藏访存延迟？
11. 什么是 Branch Divergence？它如何影响执行效率？
12. 什么是 Active Mask 和 Re-convergence？
13. 什么是 Occupancy？它受到哪些资源限制？
14. 为什么提高 Occupancy 不一定提高性能？
15. GPU Warp Scheduler 与 CPU OoO Scheduler 有什么区别？

**GPU 存储系统**

16. Global Memory、Shared Memory、Register File 分别承担什么职责？
17. 什么是 Memory Coalescing？
18. 为什么连续访存通常比不规则访存更容易高效利用带宽？
19. 什么是 Shared Memory Bank Conflict？
20. GPU 为什么需要较高的存储带宽？
21. GPU 如何通过大量未完成访存请求利用内存级并行？

**gem5 实现机制**

22. gem5 的 Wavefront 如何保存线程执行状态？
23. GPU Scheduler 如何选择可以发射的 Wavefront？
24. Compute Unit 如何管理 SIMD Lane 和寄存器资源？
25. gem5 GPUFS 如何与 Host CPU 和 ROCm 软件栈交互？
26. EduSIMT 与 gem5 GPUFS 的建模范围和精度有什么不同？

**性能量化分析**

27. 如何计算 Warp Scheduler Utilization 和 Active Lane Efficiency？
28. 如何量化 Branch Divergence 造成的执行效率损失？
29. 如何判断 GPU Kernel 是计算受限、访存受限还是调度受限？
30. 为什么增加 Compute Unit 数量不一定带来线性加速？
31. 为什么 GEMM 与 GEMV 在 GPU 上的性能特征存在明显差异？
32. GPU 的哪些设计机制值得应用到未来的数据流 AI 加速器中？

---

## Stage 8：数据流架构与空间计算

本阶段的目标是理解 Control-Flow Architecture 与 Dataflow Architecture 的区别，掌握数据依赖驱动、空间映射、Processing Element、Systolic Array 及不同 Dataflow Mapping 的基本原理，并基于现有 gem5 工程实现一个可参数化的空间数据流计算引擎。

### 1. 理论与源码学习

请结合以下教材、论文和 gem5 源码进行学习：

**《计算机体系结构：量化研究方法》：**

1. Chapter 7：Domain-Specific Architectures。
2. Chapter 4：Data-Level Parallelism。
3. Appendix F：Interconnection Networks。

**数据流架构相关论文与资料：**

1. [Eyeriss: An Energy-Efficient Reconfigurable Accelerator for Deep Convolutional Neural Networks](https://doi.org/10.1109/JSSC.2016.2616357)
2. [Eyeriss v2: A Flexible Accelerator for Emerging Deep Neural Networks](https://arxiv.org/abs/1807.07928)
3. [Timeloop：DNN Accelerator Mapping and Evaluation Framework](https://github.com/NVlabs/timeloop)
4. [MAESTRO：Data-Centric Approach to DNN Accelerator Performance Analysis](https://github.com/maestro-project/maestro)

这些资料主要用于理解数据流映射、计算资源利用率和数据搬运成本，不要求在本阶段完整学习所有论文中的硬件实现。

**gem5 文档与源码：**

1. [ClockedObject 与事件机制](https://www.gem5.org/documentation/learning_gem5/part2/events/)
2. [Creating SimObjects in the Memory System](https://www.gem5.org/documentation/learning_gem5/part2/memoryobject/)
3. [gem5 Memory System](https://www.gem5.org/documentation/general_docs/memory_system/)
4. [gem5 Garnet 2.0](https://www.gem5.org/documentation/general_docs/ruby/garnet-2/)

本阶段主要通过自定义 SimObject 实现数据流计算模型。gem5 自带的 CPU/GPU 模型不直接提供完整的通用数据流加速器，因此需要自行设计 PE、数据传递和调度语义。

### 2. 实践任务：EduDataflow

基于 EduArch-gem5，实现 **EduDataflow：Parameterized Spatial Dataflow Engine**。

**任务 1：实现 PE 与规则计算阵列**

至少实现：

- Processing Element。
- MAC / ALU。
- PE Local Register。
- PE Array。
- PE-to-PE Data Transfer。
- Pipeline Execution。
- Data Input / Output。
- 基础状态和周期统计。

从单 PE 开始扩展为二维阵列，实现一个能够运行矩阵乘法的简化 Systolic Array。

**任务 2：实现不同 Dataflow Mapping**

至少支持：

- Weight Stationary。
- Output Stationary。
- Input Stationary（可使用简化映射）。
- Loop Tiling。
- Data Reuse。
- 数据注入与结果收集。

要求同一数学算子可以采用不同映射方式执行，并保证结果一致。

**任务 3：实现数据流执行与性能统计**

至少包含：

- Compute Task / Dataflow Graph。
- 数据依赖跟踪或静态调度。
- PE Busy / Idle。
- Pipeline Fill / Drain。
- Data Transfer Count。
- Data Reuse Count。
- PE Utilization。
- Array Execution Cycles。

首先支持 GEMM 和 GEMV，再扩展到简单 Reduction 或 Elementwise 运算。

### 3. 量化分析实验

1. **PE Array Size 分析**：改变二维阵列规模，研究吞吐率、执行时间与利用率。
2. **阵列形状分析**：比较不同矩阵尺寸及长宽比对 PE 利用率的影响。
3. **Pipeline Fill / Drain 分析**：研究启动和排空开销对小规模计算的影响。
4. **Dataflow Mapping 分析**：比较 Weight Stationary、Output Stationary 等策略下的数据传输量和执行周期。
5. **Loop Tiling 分析**：研究不同 Tiling Size 对数据重用和计算效率的影响。
6. **GEMM / GEMV 对比分析**：比较两类算子的 Arithmetic Intensity、计算资源利用率和带宽需求。
7. **规则与不规则负载分析**：研究矩阵形状变化、稀疏数据或非规则访存对阵列效率的影响。
8. **理论模型与周期模型对照**：使用解析模型预测计算周期，与 gem5 EduDataflow 模拟结果进行对照。

### 4. 完成后请回答以下问题

**数据流体系结构**

1. 什么是 Dataflow Architecture？它与 Control-Flow Architecture 有什么区别？
2. 什么是 Dataflow Graph？节点和边分别表示什么？
3. 什么是 Static Dataflow 和 Dynamic Dataflow？
4. Data-Driven Execution 与传统指令执行有什么区别？
5. 什么是 Spatial Architecture？
6. Systolic Array 为什么适合规则矩阵运算？
7. Processing Element 的基本职责是什么？
8. CGRA 与传统 Systolic Array 有什么区别？

**数据复用与映射**

9. 什么是 Temporal Reuse 和 Spatial Reuse？
10. Weight Stationary、Output Stationary、Input Stationary 有什么区别？
11. 什么是 Loop Tiling？它为什么能够提高数据重用？
12. 为什么数据搬运可能比计算本身消耗更多能量？
13. 如何为一个 GEMM 算子选择合适的 Dataflow Mapping？
14. 为什么不同矩阵形状可能需要不同的映射策略？

**PE 阵列与执行机制**

15. 为什么 PE 数量增加不一定带来线性加速？
16. 什么是 Pipeline Fill / Drain？
17. 如何计算 PE Array Utilization？
18. 为什么规则 PE 阵列可能难以高效执行 GEMV？
19. 固定数据通路与可配置互联分别有哪些优缺点？
20. 如何在 gem5 中模拟 PE 之间的数据传输及依赖？

**性能量化分析**

21. 如何计算一个 GEMM 的 FLOPs 和数据搬运量？
22. 如何建立 Systolic Array 的理论执行周期模型？
23. 如何判断计算瓶颈来自 PE 数量、阵列形状还是数据供给？
24. 如何量化不同 Dataflow Mapping 的数据复用收益？
25. 为什么减少外部数据传输可能比增加 MAC 数量更加有效？
26. 如何根据 Benchmark 判断一种 Dataflow Architecture 是否适合目标工作负载？

---

## Stage 9：AI 加速器系统架构

本阶段的目标是从 Stage 8 的 PE Array 出发，构建具备多级计算组织、片上存储、DMA、NoC、控制器及 Host 接口的 AI 加速器系统模型，并掌握计算、存储和互联资源之间的设计权衡，为后续真实数据流 ASIC 的微架构探索奠定基础。

### 1. 理论与源码学习

请结合以下教材、文档和源码进行学习：

**《计算机体系结构：量化研究方法》：**

1. Chapter 7：Domain-Specific Architectures。
2. Chapter 2：Memory Hierarchy Design。
3. Appendix F：Interconnection Networks。
4. Chapter 1：Fundamentals of Quantitative Design and Analysis。

**相关架构设计参考：**

1. [Eyeriss Architecture](https://doi.org/10.1109/JSSC.2016.2616357)
2. [Eyeriss v2](https://arxiv.org/abs/1807.07928)
3. [Timeloop](https://github.com/NVlabs/timeloop)

**gem5 文档与源码：**

1. [gem5 Memory System](https://www.gem5.org/documentation/general_docs/memory_system/)
2. [Creating SimObjects in the Memory System](https://www.gem5.org/documentation/learning_gem5/part2/memoryobject/)
3. [gem5 Garnet 2.0](https://www.gem5.org/documentation/general_docs/ruby/garnet-2/)
4. [gem5 Full-System Devices](https://www.gem5.org/documentation/general_docs/fullsystem/devices)
5. [gem5 DMA Device 源码](https://github.com/gem5/gem5/blob/stable/src/dev/dma_device.cc)
6. [gem5 Port 源码](https://github.com/gem5/gem5/blob/stable/src/mem/port.cc)
7. [Garnet Router 源码](https://github.com/gem5/gem5/blob/stable/src/mem/ruby/network/garnet/Router.cc)

源码阅读重点：

- gem5 如何实现设备和内存之间的通信。
- DMA Device 如何提交读写请求并处理完成事件。
- RequestPort / ResponsePort 如何处理时序与背压。
- Garnet 如何组织 Router、Link 和 Network。
- 如何将独立加速器建模为 Host CPU 可控制的设备。
- 如何描述计算、访存与互联并行执行及资源竞争。

### 2. 实践任务：EduAccelerator

基于 Stage 8 的 EduDataflow，构建 **EduAccelerator：Multi-Tile AI Accelerator**。

**任务 1：实现多级计算与存储组织**

至少实现：

- PE / Compute Array。
- Tile。
- Cluster（根据架构设计选择是否引入）。
- Tile Local SRAM。
- Shared SRAM。
- SRAM Bank。
- Register File。
- 参数化计算和存储资源。

要求支持多个 Tile 并行执行不同任务，并能分析计算资源与片上存储容量之间的关系。

**任务 2：实现 DMA 与片上互联**

至少实现：

- DMA Engine。
- DMA Request / Completion。
- Burst Transfer。
- Double Buffering。
- Tile-to-Tile Communication。
- Shared Memory Access。
- NoC / Interconnect。
- Backpressure / Arbitration。

要求能够描述计算、DMA、SRAM 和互联之间的并行活动及资源竞争。

**任务 3：实现 Accelerator Controller 与命令接口**

至少实现：

- Accelerator Command Queue。
- Compute Command。
- DMA Command。
- Synchronization / Barrier。
- Tile Scheduling。
- Task Completion。
- 简化 ISA / Command Format。

能够通过命令或任务描述启动 GEMM、GEMV 等运算。

本阶段主要定义执行语义与硬件接口，不要求实现完整 AI 编译器。

**任务 4：实现 Host–Accelerator 交互**

至少实现：

- Host CPU。
- Accelerator Device。
- MMIO 或简化命令提交接口。
- 数据地址与任务参数传递。
- Completion / Interrupt。
- Host–Accelerator Data Transfer。
- System-Level Statistics。

可以先采用 gem5 内部的抽象设备接口完成正确性交互，再增加 PCIe 传输的延迟、吞吐和事务开销模型。

不要求直接实现完整的 PCIe 协议。

### 3. 量化分析实验

1. **Tile 数量与扩展性分析**：改变 Tile Count，研究加速比、平均 Tile Utilization 和资源竞争。
2. **SRAM 容量分析**：改变 Local / Shared SRAM 容量，研究数据复用、外存访问量和性能。
3. **SRAM Banking 分析**：研究 Bank Count、端口数量及访问模式对吞吐的影响。
4. **DMA 分析**：改变 Burst Size、Queue Depth、Outstanding Requests，研究有效带宽。
5. **Double Buffering 分析**：比较计算与数据传输串行执行和重叠执行的周期数。
6. **NoC 分析**：改变网络拓扑、链路带宽和 Traffic Pattern，研究延迟与拥塞。
7. **多 Tile 共享带宽分析**：研究计算资源增加与外部存储带宽不足之间的关系。
8. **Host–Accelerator 分析**：统计命令提交、数据传输、计算执行和任务完成的时间开销。
9. **GEMM / GEMV 资源配置分析**：对比不同 Tile、SRAM 和带宽配置下两类算子的性能瓶颈。

### 4. 完成后请回答以下问题

**AI 加速器计算组织**

1. 为什么需要把多个 PE 组织成 Tile？
2. Tile 与 Cluster 有什么区别？什么时候需要 Cluster？
3. 如何确定一个 Tile 应该包含多少 PE？
4. 多 Tile 的并行调度与单个 PE Array 有什么区别？
5. 为什么增加 Tile 数量不一定带来线性加速？
6. 如何量化多 Tile 系统的负载均衡情况？

**存储与数据搬运**

7. Local SRAM 和 Shared SRAM 各有哪些优缺点？
8. Scratchpad 与 Cache 有什么区别？
9. 什么是 SRAM Banking？为什么可能出现 Bank Conflict？
10. DMA Engine 的主要职责是什么？
11. Burst Transfer 为什么能够提高数据传输效率？
12. Double Buffering 如何隐藏部分数据传输延迟？
13. 为什么计算单元可能因为 SRAM 或 DMA 带宽不足而处于空闲状态？

**互联与系统架构**

14. Bus、Crossbar、Mesh 分别适合什么场景？
15. NoC 与 CGRA 的可配置互联有什么区别？
16. 什么是 Routing、Arbitration 和 Backpressure？
17. NoC 的链路带宽和 Router 延迟如何影响系统吞吐？
18. 为什么多个 Tile 访问共享存储时可能出现拥塞？
19. 如何判断需要增加 NoC 带宽还是外部 Memory Bandwidth？

**控制与接口**

20. Accelerator Controller 需要承担哪些职责？
21. Compute Command、DMA Command 和 Synchronization Command 应如何协同？
22. Accelerator ISA 与 CPU ISA 有什么不同？
23. 如何将一个 GEMM 算子拆分为多个 Tile Task？
24. Host CPU 通过什么机制控制独立 AI Accelerator？
25. MMIO、DMA、Interrupt 分别用于什么？
26. PCIe 的传输带宽和通信延迟会如何影响端侧加速器？

**性能量化分析**

27. 如何将一个算子的总执行时间分解为计算、DMA、NoC 和等待开销？
28. 如何判断系统瓶颈来自计算资源、片上 SRAM、DMA、NoC 还是外部 DRAM？
29. 如果固定总 SRAM 容量，应该如何权衡 Tile 数量和每 Tile 存储容量？
30. 如何根据性能数据确定合理的 Tile / Cluster 划分？
31. 为什么 GEMM 和 GEMV 可能需要不同的计算与存储资源分配？
32. 如何将 gem5 的性能模型结果转化为微架构设计规范？

---

## Stage 10：量化架构设计与 DSE

本阶段的目标是综合运用前面所有 Stage 的体系结构知识、gem5 建模能力和性能分析方法，针对端侧 LLM 推理工作负载，完成一次系统化的架构设计空间探索（Design Space Exploration，DSE）。

要求学员能够从 Workload 分析出发，建立理论性能模型和 gem5 微架构模型，进行参数扫描、性能瓶颈分析及设计方案比较，最终形成有量化证据支持的架构设计结论。

### 1. 理论与源码学习

请结合以下教材、文档和研究工具进行学习：

**《计算机体系结构：量化研究方法》：**

1. Chapter 1：Fundamentals of Quantitative Design and Analysis。
2. Chapter 2：Memory Hierarchy Design。
3. Chapter 4：Data-Level Parallelism。
4. Chapter 7：Domain-Specific Architectures。

**Workload 与架构分析相关资料：**

1. [Roofline: An Insightful Visual Performance Model](https://doi.org/10.1145/1498765.1498785)
2. [Timeloop：Accelerator Mapping and Evaluation](https://github.com/NVlabs/timeloop)
3. [Accelergy：Architecture-Level Energy Estimation](https://github.com/Accelergy-Project/accelergy)
4. [MAESTRO：Data-Centric Accelerator Performance Analysis](https://github.com/maestro-project/maestro)

**gem5 文档与源码：**

1. [gem5 Statistics](https://www.gem5.org/documentation/learning_gem5/part1/gem5_stats)
2. [gem5 Standard Library](https://www.gem5.org/documentation/gem5-stdlib/overview)
3. [gem5 SimObject Documentation](https://www.gem5.org/documentation/learning_gem5/part2/)
4. [gem5 Memory System](https://www.gem5.org/documentation/general_docs/memory_system/)

源码与工具学习重点：

- gem5 Statistics 的定义、注册和读取。
- 如何控制 Benchmark 的统计区间。
- 如何建立参数化架构配置。
- 如何使用自动化脚本完成 Design Space Exploration。
- 如何通过理论性能模型检查周期模拟结果。
- 如何进行性能、面积和能耗估计，以及如何声明模型精度。

### 2. 实践任务：Architecture DSE Platform

基于 Stage 9 的 EduAccelerator，建设完整的 **AI Accelerator Architecture Exploration Platform**。

**任务 1：建立 LLM Workload Characterization**

选择一个代表性的端侧 LLM 模型，分析：

- Model Parameters。
- Batch Size。
- Context Length。
- Prefill / Decode。
- GEMM。
- GEMV。
- Attention。
- Softmax。
- RMSNorm。
- KV Cache。
- FLOPs / Bytes。
- Arithmetic Intensity。
- Operator Dependency。

首先通过真实张量形状建立算子级 Benchmark，不要求在 gem5 中运行完整大模型。

要求针对不同 Batch Size、Sequence Length、矩阵形状生成相应的工作负载。

**任务 2：构建理论性能模型和参数化 gem5 模型**

至少实现：

- Analytical Performance Model。
- Roofline Model。
- Compute Latency Model。
- Memory Transfer Model。
- NoC Communication Model。
- Tile Scheduling Model。
- 参数化 Hardware Configuration。
- Performance Counter Collection。
- Bottleneck Analysis。

要求理论模型和 gem5 模型使用明确一致的工作负载定义，并对计算量、数据搬运量、周期数及资源利用率进行交叉检查。

**任务 3：实现自动化 DSE 与架构评估**

至少支持：

- Parameter Sweep。
- Experiment Configuration。
- Benchmark Automation。
- Statistics Parsing。
- Scaling Analysis。
- Sensitivity Analysis。
- Bottleneck Classification。
- Performance Visualization。
- Pareto Analysis。
- Architecture Comparison。

主要扫描参数包括：

- PE Count。
- Tile Count。
- Compute Frequency。
- SRAM Capacity / Banking。
- DMA Bandwidth。
- NoC Bandwidth。
- External Memory Bandwidth。
- Dataflow Mapping。

增加经过标定的面积/能耗估算时，必须注明其模型来源与误差范围，不能直接将 gem5 周期统计当作真实芯片功耗。

### 3. 量化分析实验

1. **Workload Characterization**：分析 Prefill 和 Decode 的算子类型、计算量、访存量与 Arithmetic Intensity。
2. **Roofline 分析**：建立理论峰值算力和带宽限制，判断不同算子的性能上限。
3. **PE / Tile 数量扫描**：研究计算资源增加对实际吞吐、延迟和利用率的影响。
4. **SRAM 容量扫描**：研究 SRAM 容量与外部数据搬运量、执行周期之间的关系。
5. **Memory Bandwidth 分析**：改变外部内存带宽，研究 GEMM、GEMV 和 Attention 的性能敏感性。
6. **NoC 分析**：研究多 Tile 数据传输、拥塞和链路利用率。
7. **Dataflow Mapping 分析**：比较不同数据流策略下的数据复用、执行时间和资源利用率。
8. **Prefill / Decode 对比分析**：研究两类推理阶段对计算能力、存储带宽和存储容量的不同需求。
9. **架构参数敏感性分析**：找出对性能影响最大的架构参数，分析参数变化的边际收益。
10. **多目标 DSE**：在给定面积、功耗及带宽预算下比较多个设计点，寻找 Pareto-optimal Architecture。
11. **性能模型验证**：对照解析模型与 gem5 结果，分析模型偏差、假设和未建模机制。
12. **Architecture Trade-off**：综合性能、成本、扩展性和实现复杂度，为一个候选架构提出定量设计依据。

### 4. 完成后请回答以下问题

**LLM Workload Characterization**

1. 如何从 LLM 模型结构推导主要算子的计算量和访存量？
2. Prefill 与 Decode 在计算负载和访存模式上有什么区别？
3. Batch Size 如何影响 GEMM/GEMV 的计算特征？
4. Context Length 如何影响 Attention 和 KV Cache 的存储需求？
5. 为什么低 Batch Decode 通常对外部内存带宽更加敏感？
6. 如何计算 GEMM、GEMV、Attention 的 Arithmetic Intensity？
7. 为什么不同算子对计算资源和存储系统的要求不同？

**理论性能建模**

8. 什么是 Roofline Model？它能够解释哪些性能问题？
9. 如何计算加速器的 Peak FLOPS 和 Achievable FLOPS？
10. 如何建立一个算子的理想执行周期模型？
11. 如何估算 SRAM / DRAM 数据搬运时间？
12. 什么情况下计算与数据传输可以重叠？重叠后如何估算总执行时间？
13. 如何利用 Amdahl's Law 判断局部优化对整体性能的贡献？
14. 如何利用 Little's Law 分析访存延迟和并发请求之间的关系？

**架构设计空间探索**

15. 什么是 Design Space Exploration（DSE）？
16. 如何确定需要扫描的架构参数和参数范围？
17. 为什么增加 PE Count 不一定显著提高性能？
18. 如何确定合理的 Tile Count 和 SRAM Capacity？
19. 如果只能增加一种资源，应该优先增加计算能力、SRAM 还是外部内存带宽？
20. 如何判断某个架构参数已经达到收益饱和点？
21. 什么是 Sensitivity Analysis？它如何帮助架构师确定设计优先级？
22. 什么是 Pareto Frontier？为什么架构设计往往不存在单一的最优解？

**gem5 性能模型与验证**

23. 如何将一个抽象微架构转换为 gem5 的 SimObject 和周期执行模型？
24. 如何验证 gem5 模型的功能正确性？
25. 如何检查模型中的计算量、访存量和周期统计是否自洽？
26. 为什么模型的详细程度不等于模型的准确程度？
27. 如何识别模拟结果中可能存在的错误或不合理假设？
28. 如何选择需要进一步校准的性能参数？
29. 如果解析模型与 gem5 模拟结果存在明显差异，应该如何定位原因？

**最终架构设计能力**

30. 如何从一个 Workload 出发提出初步的 Architecture Design？
31. 如何定量比较不同 Dataflow Mapping？
32. 如何判断当前加速器的主要性能瓶颈？
33. 如何将性能模型中的资源抽象映射到实际微架构模块？
34. 如何在性能、面积、功耗和研发复杂度之间进行权衡？
35. 一个微架构设计方案应向 RTL 开发人员交付哪些内容？
36. gem5 模型和 RTL 模型之间应如何开展交叉验证？
37. 如何判断一份 Architecture Evaluation Report 的结论是否具有充分的量化证据？
