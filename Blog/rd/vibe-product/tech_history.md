---
title: tech_history
description: 
published: true
date: 2026-09-20T15:30:35.593Z
tags: 
editor: markdown
dateCreated: 2026-09-20T15:29:58.735Z
---

# 技术史 —— 学 AI Infra 的横切注释层

> 这份文档不属于任何一层：它是**注释**，回答的是"为什么长成现在这样"。
> 每条线的末尾都写「读完能回答什么」——那是验收判据，答不出来就不算读过。
> 位置在 `Blog/rd/vibe-product/`（未审区），内容按 `Blog/rd/AGENT.md` 的纪律自觉核对。

（结合zartbot的技术博客，学一遍技术史）

## 怎么用这份文档

**三条纪律：**

1. **年份和版本号只用来搭时间轴，引用前核一手来源。** 本文的年份来自我的记忆，我把握不足的都标了 ⚠️。凡是要写进别的文档里的年份，按 `AGENT.md` §三 的权威顺序回去核。
2. **一手 vs 二手要分清。** 二手（公众号、博客、架构分析）适合上手，一手（ISA 手册、规范、论文）才能当依据。二手读到的结论，必须能用一手复述一遍。
3. **「读完能回答什么」是判据。** 每条末尾的三个问句答不出来，说明只是看过了，不是学过了。

**它和 `Resources/` 七层的关系：** 七层是归档位置（东西放哪），本文是横切视图（这些位置为什么是现在这样）。同一个主题会在两边各出现一次——不是重复，是索引和注释的区别。

---

## 覆盖表

| 层 | 条 | 备注 |
|---|---|---|
| **L1 芯片与微架构** | 1 片上存储之争 · 2 GPU 微架构 · 3 加速器 · 4 存储与内存层级 · 5 数值格式与量化 | 最厚的一层，也都是 POC 的第一个候选命题所在 |
| **L2 互联与系统** | 6 片间与系统互联 · 7 集群与数据中心网络 | ⑥ 是项目定位的核心 |
| **L3 驱动与运行时** | 8 编程模型 · 9 驱动与运行时 | ⑨ 直接对应 POC 的 kmd/umd 那条 |
| **L4 编译器与算子** | 10 编译器 · 11 GEMM 与算子库 | 算子移植 + MLIR 的正面战场 |
| **L5 推理引擎与模型** | 12 模型架构史 · 13 推理引擎与调度 | 负载侧，反过来约束前面所有层 |
| **L6 分布式与服务化** | 14 分布式并行策略 · 15 容错与恢复 | 推理侧还没走到，但不能不知道 |
| **L7 应用与 Agent** | 16 从 API 到 Agent | 最上面一层，决定"值不值得做" |

## 建议的阅读顺序（四阶段）

| 阶段 | 条 | 为什么排在这个位置 |
|---|---|---|
| **A 地基** | **1** → 2 → 3 | ①②③ 决定 L1，而 L1 是 POC 的第一个候选命题。不先读，后面所有取舍都没有参照系 |
| **B 约束** | 4 → 5 | ④⑤ 给的是"物理上限"：带宽、容量、精度误差。它们是你算上界的输入 |
| **C 主战场** | 10 → 11 → 8 | 算子移植 + MLIR 这条线的正面战场（rd 的活） |
| **D 定位与负载** | 6 → 9 → 12 → 13 → 7 → 14 → 15 → 16 | ⑥ 是项目定位，⑨ 是 POC 的另一条腿，⑫⑬ 是负载，⑦⑭⑮⑯ 按需（推理侧走到再读） |

**为什么第 1 条排最前面**：scratchpad（软件管理）vs cache（硬件管理）的取舍，就是你第一个候选命题的历史版本。先读它，等于提前拿到一批先例和反例。

---

# L1 芯片与微架构

## 1. 片上存储之争（放在第一位读）

**为什么读**：scratchpad vs cache 的取舍，在 DSP → Cell BE → GPU → 加速器上一再重演。你要证的"per-core scratchpad 站得住"，历史上有人证过、也有人证伪过——**不读它，你就是在没有先例的情况下重做一遍**。

**时间线**

- 1990s：DSP/嵌入式确立 scratchpad 的基本论证（确定性、面积与能耗效率、无 tag 开销）
- 2002：⚠️ Banakar 等把"scratchpad 作为片上存储的设计替代"系统化
- 2005：Cell BE 的 SPE local store（256 KB，显式 DMA）——最极端的软件管理先例
- 2006 起：GPU 走 cache 路线（共享内存 + L1/L2 分层），把管理权部分交还硬件
- 2016 起：TPU 系加速器回到软件管理，用编译器排定片上搬运
- 现在：两边都活着，分界线是"**访问模式可不可预测**"

**读什么**

- `[一手]` Cell BE 的 SPE 文档与 DMA 模型（对 back-of-the-envelope 最友好）
- `[一手]` TPU v1 论文（ISCA 2017）里关于片上存储与编译器排布的段落
- `[二手]` `/mnt/f/Books/dl/Machine-Learning-Systems-Vol1.pdf`——从 roofline 与硬件约束推导设计的那几章

**读完能回答**：① 什么条件下 scratchpad 比 cache 好，判据是哪个量？② Cell BE 为什么最后输了，输在架构还是生态？③ 把 fast memory 容量翻倍，你的架构会变好吗——为什么？

**落到**：L1（存储层级与数据复用）——**第一个候选命题**

## 2. GPU 微架构

**为什么读**：它是所有"高性能"讨论的默认参照系，也是你们 POC 的**性能基线**。不懂它，你没法说"我的选择与 GPU 不同，而且更好"。

**时间线**

- 固定管线 → 可编程着色器（2001 上下）
- 2006：G80 统一架构；⚠️ CUDA 1.0 在 2007
- 2010 Fermi：真正的 cache 层级、ECC、统一地址空间
- 2012 Kepler → 2014 Maxwell → 2016 Pascal
- 2017 Volta：Tensor Core 与独立线程调度
- 2018 Turing → 2020 Ampere → 2022 Hopper → 2024 Blackwell：核弹化、chiplet 化

**读什么**

- `[一手]` `/mnt/f/Documents/cuda/`、`/mnt/f/Documents/nvidia/` 里的编程指南与架构白皮书
- `[本机]` `~/Source/`：`blackwell-isa`、`cutlass`、`CuAssembler`、`DocumentSASS`、`nv_isa_solver`、`sass-toolkit`、`sasskit`、`ptxNinja`、`gpuocelot`
- `[二手]` `/mnt/f/Vaults/Tech/zartbot/`（含"Nvidia Rubin 架构分析预览"）——上手快，但**必须回头核一手**

**读完能回答**：① 为什么 GPU 的 shared memory 是"半软件管理"的？② Tensor Core 出现后，寄存器压力格局怎么变了？③ 一条 GEMM kernel 从 launch 到完成，硬件上有哪些必须等待的点？

**落到**：L1（计算单元与存储层级）

## 3. 加速器（dataflow 的谱系）

**为什么读**：TT 是你们微架构的起点，但 TT 只是这条谱系里的一支。知道前后左右，才知道哪些约束是**这类架构共有的**、哪些只是 TT 的选择——这正是"与 TT 差异清单"的判据来源。

**时间线**

- 2015：TPU v1 部署（推理专用，systolic array + 软件管理存储）；2017 论文公开论证
- ⚠️ 2016 前后：Graphcore、Cerebras、Groq、Tenstorrent 相继成立，路线各不相同
- 2017 起：Transformer 成为主负载，加速器从 CNN 假设转向 attention/MoE 假设
- 2020s：chiplet、多芯粒、CXL 挂载出现在加速器方案里

**读什么**

- `[一手]` `~/Source/tt-isa-documentation`（写 kernel 的权威参考）、`~/Source/tt-metal/tech_reports/`（60 份，你们正在用的材料）
- `[本机]` `~/Source/`：`tt-metal`、`tt-lang`、`tt-forge`、`ttsim`、`ttsim-qemu`、`tt-share`
- `[二手]` TPU 系列的公开分析（用来对照 TT 的选择）

**读完能回答**：① systolic array 与 tile-based dataflow 的分界线在哪？② 哪些约束是"专用加速器都躲不掉的"？③ TT 的哪些选择是历史包袱而非物理必然？

**落到**：L1（dataflow 谱系与你们的定位）

## 4. 存储与内存层级（把"物理上限"钉住）

**为什么读**：前面三条讲"架构怎么选"，这条讲"**选了也躲不掉什么**"。你要算的那个"物理上界"，主要就是这条线给的数。

**时间线**

- 1995：memory wall 被明确命名（算力增长快于访存带宽）
- 1990s–2010s：SDR → DDR → DDR2/3/4/5，约每五到七年一代；显卡另走 GDDR
- 2015：HBM 首次商用（3D 堆叠 + TSV），把带宽与封装绑在一起
- 2016 起：HBM2/2E/3/3E；片上 SRAM 的成本随工艺下降得比逻辑更慢
- 2014（ISSCC）：Horowitz 的"计算的能量问题"给出 pJ/bit 与 pJ/op 的量级表，成为所有 back-of-the-envelope 的起点
- 现在：**算术强度**（byte/FLOP）成为一切设计的中心量

**读什么**

- `[一手]` Horowitz, *Computing's Energy Problem (and what we can do about it)*，ISSCC 2014——那张表要背下来量级
- `[本机]` `~/Source/less_slow.cpp`（性能反直觉的那批实验，正好是这条线的实战版）
- `[二手]` `/mnt/f/Books/dl/Machine-Learning-Systems-Vol1.pdf` 的 roofline 章节

**读完能回答**：① 为什么 HBM 用 3D 堆叠而不是提高频率？② 片上访存与片外访存的单位能量差几个量级？③ 你要证的那个设计选择，卡在容量还是带宽还是能量？

**落到**：L1（存储层级）、以及所有"算上界"的动作

## 5. 数值格式与量化

**为什么读**：它是"有后果的词"最密集的一块——每一个格式名都对应一个存储/带宽/误差的取舍，而且**词用错后果立刻显形**。

**时间线**

- 早期：FP32 一统
- 2017 Volta：FP16 与 FP32 混合（Tensor Core 的默认形态）
- 2018 Turing：INT8 / INT4
- 2020 Ampere：TF32、BF16
- 2022 Hopper：FP8（E4M3 / E5M2）
- 2023–2024：block-scaled 微缩放格式（MX FP8/FP6/FP4 一类）与 NVFP4
- 算法侧：PTQ vs QAT；per-tensor → per-channel → per-group；GPTQ / AWQ / SmoothQuant / GGUF 的 k-quants

**读什么**

- `[一手]` `/mnt/f/Documents/nvidia/` 里的格式白皮书与 Tensor Core 精度说明
- `[本机]` `~/Source/tt-metal`、`~/Source/cutlass`（看它们各自支持哪些格式、累加用什么精度）
- `[二手]` 量化方法的综述（注意区分"论文里的精度"与"工程里能跑的精度"）

**读完能回答**：① 为什么输入用低精度、累加却要用 FP32？② block scaling 解决的是哪一类误差，代价是什么？③ 你的 ASIC 若支持 FP8/FP4，片上容量与带宽需求会怎么变——这个变化会不会改掉你前面几条线的结论？

**落到**：L1（精度格式），并反向影响 L5（模型能不能用）

---

# L2 互联与系统

## 6. 片间与系统互联（项目定位的核心）

**为什么读**：你们项目定位就是"通过 PCIe/CXL 与 SoC 互联"。这条线的历史里有一串**死掉的方案**，它们的死法比 CXL 的特性更有信息量。

**时间线**

- PCIe 各代：带宽翻倍，但**始终不可缓存一致**
- 2016：NVLink 出现——为绕开 PCIe 在加速器间互联上的限制
- ⚠️ 2016 前后：CCIX、OpenCAPI、Gen-Z 三个竞争方案出现，最终都被 CXL 边缘化
- 2019 / 2020 / 2022：CXL 1.0 / 2.0 / 3.0——CXL.cache + CXL.mem 把"加速器与 host 共享地址空间"标准化
- 2022：UCIe 处理封装内 die 间互联

**读什么**

- `[一手]` CXL 规范（`/mnt/f/Documents/cxl/`）：先只读 Type-1/2/3 的定义与 CXL.cache / CXL.mem 的分工
- `[本机]` `~/Source/`：`cxl-lab`、`develop_for_cxlmemsim`、`linux-insides-zh`、`qemu-book`
- `[本机]` `~/Desktop/Codebase/cosim-gpu`（即 `~/Source/cosim-gpu`，QEMU + gem5 的联合仿真通路）
- `[二手]` CCIX / OpenCAPI / Gen-Z 的回顾（**重点读"为什么没成"**）

**读完能回答**：① CXL 相对 PCIe 到底解决了什么，代价是什么？② 为什么 CCIX/OpenCAPI/Gen-Z 失败了，CXL 凭什么活下来？③ 你的 ASIC 挂在 CXL 上，哪一类访问会先成为瓶颈——延迟还是一致性？

**落到**：L2（PCIe / CXL / 多芯片）

## 7. 集群与数据中心网络（scale-out）

**为什么读**：这一层现在离 POC 还远，但它是"**并行策略能不能兑现**"的物理约束。你们迟早要回答"协作规模应当多大"。

**时间线**

- 2010s：InfiniBand（FDR/EDR/HDR…）与 RoCE 成为训练集群主流
- ⚠️ 2016 起：NVLink/NVSwitch 把"超节点"从概念做成产品
- 2019 起：RDMA + 集合通信库（NCCL 一类）成为训练栈的固定件
- 2020s：800G 以太网、超节点规模继续放大；"网络是训练故障主要来源"成为共识

**读什么**

- `[一手]` InfiniBand / RoCE 的入门规范章节（只需知道延迟与带宽量级、以及拥塞与流控的位置）
- `[二手]` `/mnt/f/Books/dl/Machine-Learning-Systems-Vol2.pdf` 的通信与并行章节
- `[本机]` `~/Desktop/260914.md` 第 7 章（数据中心网络）——你们自己那份提纲

**读完能回答**：① all-reduce 在 ring 与 tree 上的差异，分别什么时候更优？② 为什么 all-to-all（MoE）比 all-reduce 更难？③ 如果协作规模翻倍，你的瓶颈会从哪一层移到哪一层？

**落到**：L2、L6（分布式与服务化）

---

# L3 驱动与运行时

## 8. 编程模型

**为什么读**：它决定"硬件向程序员暴露什么"。你们的 ASIC 最终要选一个（tile 级 DSL？显式 DMA？编译器全排？），而这个选择的历史成本可以直接读到。

**时间线**

- 图形 API 时代：程序员写"每像素干什么"
- 2006 起：CUDA 确立 host/device 分离 + SIMT 编程模型
- 2008：OpenCL 1.0 试图跨厂商统一，最后败在生态
- PTX 的意义：作为**虚拟 ISA**，把编译器和硬件解耦
- 2010s 末起：从"写 kernel"转向"写 tile 级 DSL 或交给编译器"

**读什么**

- `[一手]` CUDA 编程指南的编程模型章节（只读模型部分，API 细节按需查）
- `[本机]` `~/Source/tt-metal`（TT-Metalium 是另一种编程模型：kernel + 显式数据搬运）
- `[本机]` `~/Source/tt-lang`（更上层抽象的尝试）

**读完能回答**：① PTX 这种虚拟 ISA 替硬件挡掉了什么？② TT 的编程模型与 CUDA 最大的差别是哪一条？③ 让你给这个 ASIC 选编程模型，你会先排除哪一类——为什么？

**落到**：L3（驱动/运行时的分界面）、L4（编译器暴露什么）

## 9. 驱动与运行时（POC 的另一条腿）

**为什么读**：POC 里"kmd/umd 简易版本"是明确的一条腿，而且它是**唯一能先于性能模型跑起来的**——因为设备接入通路是现成的。这条线回答"什么必须留在内核"。

**时间线**

- 2000s：`/dev/dri`、DRM/DRI 确立"内核管资源、用户态管策略"的分工
- 2006 起：CUDA 把驱动切成 driver API 与 runtime API 两层
- 2010s：UIO → VFIO（设备直通）；DMA-BUF 处理跨设备缓冲
- 2020s：UMD + KMD 的分工定型；vfio-user 让"用户态进程假装成设备"成为标准做法（仿真里尤其有用）

**读什么**

- `[本机]` `~/Source/qemu-book`、`~/Source/linux-insides-zh`（理解内核侧要处理什么）
- `[本机]` `~/Source/tt-share`、`~/Source/ttsim-qemu`、`~/Desktop/control/`（你们已经打通的实践）
- `[一手]` VFIO 与 vfio-user 的接口文档

**读完能回答**：① 哪些操作必须在 KMD 而不能在 UMD，判据是什么？② vfio-user 这类用户态设备模拟为什么在仿真里特别有用？③ 你们的驱动接口里，哪些是硬件强制的、哪些只是习惯？

**落到**：L3（KMD/UMD）

---

# L4 编译器与算子

## 10. 编译器

**为什么读**：dataflow ASIC 的编译器要解决的核心问题是**算法与调度的分离**——这条线有二十年积累，你不必从头发明抽象。

**时间线**

- 2012：Halide 提出算法（algorithm）与调度（schedule）分离——本线最重要的思想
- 2017–2018：TVM 把这一思想推广到深度学习（Relay → TIR 的降级链）
- 2019：MLIR 提出多层 IR 与 dialect 体系，成为事实标准
- 2019：Triton 用 tile 级 DSL 换掉手写 CUDA 的收益上限
- 2020s：torch.compile 一类把编译接进框架默认路径

**读什么**

- `[一手]` Halide 的 PLDI 2012 论文（只读"算法/调度分离"那节的动机就够）
- `[本机]` `~/Source/tvm`、`~/Source/triton`、`~/Source/assignment-tirx-gemm`
- `[一手]` MLIR 官方文档里的 dialect 与 lowering 概念

**读完能回答**：① "算法与调度分离"在你们架构上对应哪两个东西？② 为什么 MLIR 用多层 IR 而不是一层？③ Triton 相比 CUDA 牺牲了什么、换来了什么？

**落到**：L4——**算子移植 + MLIR 的正面战场**

## 11. GEMM 与算子库

**为什么读**：GEMM 是唯一被优化到极致的算子，它把"数据复用"的所有手段都演过一遍。你那套 layout 代数（`layout_polynomials.md`）就长在这条线上。

**时间线**

- 2014：cuDNN 起步，靠手工调优的 kernel 集合
- ⚠️ 2017：CUTLASS 把 GEMM 拆成可组合的模板（tile → warp → thread 层级）
- 2022：CUTLASS 3.0 引入 CuTe，把 layout 变成可做代数的一等对象
- 2020s：Triton / DeepGEMM 一类继续把调优经验固化进编译器或库

**读什么**

- `[本机]` `~/Source/cutlass`、`~/Source/DeepGEMM`、`~/Source/SGEMM_CUDA`、`~/Source/less_slow.cpp`、`~/Source/RL-Kernel`
- `[一手]` CuTe 的 layout 文档（`media/docs/cute/`）
- `[二手]` `/mnt/f/Website/siboehm`（SGEMM 优化那篇经典博客）
- `[本机]` 自己那批实验：`~/Desktop/cutlass-experiments/`

**读完能回答**：① 从 naive 到 cuBLAS 级，每一步收益靠的是什么？② tile 层级的每一层各自在解决哪个资源？③ 你的 layout 判据比 CUTLASS 的静态断言多接受了哪些配置——那些配置在硬件上会发生什么？

**落到**：L4——**你自己最有差异化的资产**

---

# L5 推理引擎与模型

## 12. 模型架构史（"负载"是怎么长成现在这样的）

**为什么读**：POC 里有一条"完成 llm 选型"。不读这条线，选型只能靠别人的评测分数；读了它，你能按**硬件画像**选而不是按榜单选。

**时间线**

- 1997：LSTM——序列建模的长期默认
- 2014：seq2seq + attention（Bahdanau 一类）——attention 作为 RNN 的附件出现
- 2017：Transformer（*Attention Is All You Need*）——attention 反过来成了主体
- 2018：BERT / GPT 确立"预训练 + 微调"范式
- 2020：GPT-3 让 in-context learning 成为中心
- 2021：MoE 复兴（Switch Transformer 一类）
- 2022–2023：开源 LLM（LLaMA 系）、RoPE、FlashAttention、长上下文
- 2024 起：MoE 普及、推理模型、多模态

**读什么**

- `[一手]` *Attention Is All You Need*（只读架构与复杂度那两节）
- `[二手]` `/mnt/f/Vaults/Tech/zartbot/`（含 *Attention, Sparse or Linear*）
- `[本机]` `~/Source/tiny-llm`、`~/Source/RWKV-LM`（RWKV 是"另一条路"的典型对照）
- `[本机]` `~/Desktop/260914.md` 第 2 章（模型架构如何形成资源需求）

**读完能回答**：① attention 取代 RNN 的第一性原因是并行性还是拟合能力？② MoE 在硬件上的真实代价是什么（数据相关的访存）？③ 你们选的那个模型，它的哪一部分正好压在你们架构的强项上？

**落到**：L5（模型与负载）

## 13. 推理引擎与调度（吞吐的瓶颈怎么换了地方）

**为什么读**：这是要服务的负载形态。引擎的每一次代际，都是因为**瓶颈换了个地方**——这个规律本身就是判断力。

**时间线**

- 2016 起：TensorRT / ONNX Runtime——图优化 + kernel 选择
- 2019–2021：静态 batching 的天花板暴露（整批等最慢的）
- ⚠️ 2022：连续批处理（Orca 一类）把吞吐从"整批"里救出来
- 2023：PagedAttention / vLLM 把 KV cache 变成可管理的分页存储
- 2023–2024：前缀复用（RadixAttention 一类）
- 2024 起：分离式 prefill / decode（DistServe、Splitwise 一类）

**读什么**

- `[一手]` PagedAttention 论文（SOSP 2023）
- `[本机]` `~/Source/tiny-llm`
- `[本机]` `~/Desktop/260914.md` 第 8–9 章（推理优化、分布式推理）

**读完能回答**：① 为什么 KV cache 的**管理方式**比它的容量更影响吞吐？② prefill 与 decode 的资源画像差在哪，为什么这会逼出分离式部署？③ 你们这个 ASIC 要加速的是 prefill、decode 还是两者——这个选择会怎样反过来改架构？

**落到**：L5（推理引擎与调度）

---

# L6 分布式与服务化

## 14. 分布式并行策略

**为什么读**：推理侧现在还没走到，但你们迟早要回答"模型怎么分工、状态放哪里"。并行的每一维本质上都在**拿通信换显存和算力**，这个交换比是可算的。

**时间线**

- 2018–2019：数据并行 + 梯度累积成为默认起点
- 2019：张量并行（Megatron-LM 一类）——切算子本身
- 2019：流水并行（GPipe / PipeDream 一类）——切层
- 2020：ZeRO 把优化器状态与梯度也切开
- 2021：3D 并行与 MoE 专家并行成为大模型标配
- 2022 起：FSDP、序列并行；推理侧则走 TP/PP/EP 与 PD 分离

**读什么**

- `[二手]` `/mnt/f/Books/dl/Machine-Learning-Systems-Vol2.pdf` 的分布式章节
- `[本机]` `~/Desktop/260914.md` 第 6 章（超节点）、第 10 章（训练系统）
- `[一手]` 各家并行论文里"通信量 vs 显存节省"的推导（这是最该抄的一类推导）

**读完能回答**：① 四种并行（数据/张量/流水/专家）各自换的是什么，交换比怎么算？② 为什么张量并行要求高带宽互联，而流水并行不要求？③ 你们的 ASIC 若只做单芯，这一层里哪一条仍然必须懂？

**落到**：L6（并行策略）

## 15. 容错与恢复

**为什么读**：规模一上去，**故障从"意外"变成"常态"**。这条线决定"系统的可用形态"，也决定状态（KV cache、权重）该放在哪一级存储。

**时间线**

- 2010s：checkpoint / restart 是训练的默认手段
- 2019 起：大集群上"故障常态化"推动更细粒度的恢复（重算代替恢复）
- 2020s：训练侧出现冗余计算、弹性训练等方案
- 推理侧：⚠️ 2024 起出现"持久化 KV / 会话恢复"类做法（把 KV 或恢复日志放进比 HBM 更持久的层级）

**读什么**

- `[二手]` `/mnt/f/Books/dl/Machine-Learning-Systems-Vol2.pdf` 的可靠性与检查点部分
- `[本机]` `~/Desktop/Notes/` 与 `~/Desktop/draft/` 里你自己关于容错推理的记录
- `[一手]` 你正在跟的那类"持久化 KV / 恢复日志"论文

**读完能回答**：① 为什么"重算"常常比"恢复"更划算？② KV cache 放哪一级存储，取决于哪两个量？③ 容错需求会不会反过来要求架构支持某一级持久化——这就是 L2 的 CXL 吗？

**落到**：L6（容错与状态放置）

> 老实说：这条线的**本机材料最薄**（几份笔记 + 一篇论文），所以它的时间线可信度也最低，读的时候把它当"待建立"而非"已知"。

---

# L7 应用与 Agent

## 16. 从 API 到 Agent

**为什么读**：最上面这一层决定"值不值得做"。而且 agent 的负载形态与"问答式推理"差别很大——它会**反过来**改变对下面每一层的要求。

**时间线**

- 2022：ChatGPT 让"对话式调用"成为默认界面
- 2023：function calling / tool use 标准化；ReAct 一类把"想—做—看"固化成交互循环
- 2024：长上下文 agent、computer use 与它的评测（OSWorld 一类）
- 2024：上下文与工具的协议化（MCP 一类）
- 2025 起：多 agent 编排、评测与可观测性成为独立话题

**读什么**

- `[一手]` 各家 function calling / 工具协议的规范
- `[二手]` `/mnt/f/Vaults/Tech/` 里那篇 computer-use agent 的评测（OSWorld 2.0）
- `[本机]` `~/Source/LibreChat`、`~/Source/agent-almanac`、`~/Desktop/quizforge`（你自己的 agent 实践是最好的材料）

**读完能回答**：① agent 负载与"一问一答"在资源画像上差在哪（轮数、上下文增长、串行等待）？② prefix caching 为什么对 agent 特别值钱？③ 从 agent 的需求往回看，你的 ASIC 该优先优化哪一段？

**落到**：L7（应用与 Agent），并反向约束 L5/L6

---

## 附一：把这十六条拼成一条 NVIDIA 路径

如果你要走"以 NVIDIA 为教材、从底到上读一遍"这条路径，顺序建议是：

```
L1  2 GPU 微架构（架构白皮书 + ISA）
L1  4 存储与内存层级（带宽与能量的量级）
L1  5 数值格式与量化（Tensor Core 支持哪些精度）
L3  8 编程模型（CUDA 的编程模型章节）
L4 11 GEMM 与算子库（CUTLASS / CuTe）
L4 10 编译器（Triton，以及 MLIR 在 NV 后端的位置）
L3  9 驱动与运行时（driver API 与 runtime API 的分工）
L2  6 片间与系统互联（NVLink / NVSwitch / PCIe 各担什么）
L5 13 推理引擎（这一层 NVIDIA 自己也在用别人的生态）
```

`1 片上存储之争`、`3 加速器`、`12 模型架构史` 不在这条路径里——它们是**反例与对照**，应该与这条路径**并行读**，否则你会把 NVIDIA 的默认选择当成唯一选择。

## 附二：一二手材料的分工与核对纪律

| | 一手 | 二手 |
|---|---|---|
| 是什么 | ISA 手册、规范、论文、官方文档、源码 | 公众号、博客、架构分析、视频 |
| 适合 | 当依据、写进文档 | 上手、建立直觉、找方向 |
| 纪律 | 引用时给出版本与页码/行区间 | 结论必须能用一手复述一遍才可采信 |

本机的一手材料主要在两处：`~/Source/`（源码与手册）和 `/mnt/f/Documents/`（PDF 规范）。
二手的现成入口：`/mnt/f/Vaults/Tech/zartbot/`、`/mnt/f/Website/`。

## 附三：本文自己承认的欠账

1. **一手材料的精确坐标没填。** 每条线只写到"读哪份"，没写到"读第几章/哪几行"。按 `AGENT.md` 的纪律，写进文档的断言要附核查路径——这一项现在是欠账，也是下一步最该做的事（它顺便能验证二手结论）。
2. **年份里带 ⚠️ 的需要核**（Banakar 2002、CUDA 1.0、CUTLASS 2017、加速器与互联方案成立年份、Orca 2022、NCCL 与 NVSwitch 的位置、持久化 KV 的时间）。
3. **两处覆盖仍偏薄**：`15 容错与恢复`（本机材料最少，可信度最低）、`7 集群与网络`（离 POC 远，只有二手材料）。
4. **训练侧未展开。** POC 不做训练，但训练的并行与容错历史会渗进推理系统（尤其 L6），现在只取了"渗进来的那部分"。
5. **这份清单是注释层，不是学习计划。** 真正带顺序和验收的是阅读顺序表；如果要更细的"每天读什么"，应该另开一份路径文件，而不是把计划塞进这里。
