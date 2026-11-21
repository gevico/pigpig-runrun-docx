# 嵌入式 C++ 硬件构造层（HCL）路线研究报告：基于 Kathryn C++ 复用的 gem5 与 RTL 双后端统一建模方案

> **报告定位**：本文作为 `report.md` 的核心技术专题报告，系统阐述 **嵌入式 C++ 硬件构造层（Hardware Construction Layer, HCL）** 的技术架构、开源底座选型（以 **Kathryn C++** 为首要复用核心）、前期原型全量可靠性实证成果、在 GW100 项目中的落地分工以及后续演进路线。全文面向团队评审与对外技术汇报，聚焦**最终收敛的复用路线与客观实测结论**。

---

## 1. 背景与核心问题：为什么需要一条新的 HCL 路线？

在传统的芯片架构探索流程中，体系结构团队先使用 gem5 构建高速 C++ 性能模型，完成系统级评估后再由硬件团队重新编写 RTL。这种双模型割裂带来了长期的一致性维护成本。在探索“从 gem5 走向 RTL”的过程中，前期对 GW100 Compute Tile 最小有状态单元 **`CtSync`**（32×32 位硬件同步计数器）及主动搬运单元 **`CtNidma`** 的实证评估揭示了一个本质规律：

- **直接综合或宏切换改造既有 gem5 代码在工程上不可行**：既有 gem5 模块深度依赖离散事件时间戳（`Tick` / `syncsFree`）、虚函数多态、裸指针与动态堆缓冲（如 `CtNidma` 中的 `std::vector` 与 Lambda 回调）。在 `CtSync` 上强行做同文件条件编译，会导致高达 **84.4%** 的代码落入互斥分支，且暴露出状态提交可见性、端口并行度、反压协议与同拍并发仲裁 4 类底层语义冲突。
- **传统 C++ HLS 无法凭空补全缺失的硬件架构约束**：虽然严格受限的算术内核可以干净降级为无堆、无闭包的纯 SSA LLVM IR，但行为级控制/搬运代码缺失队列深度、每拍吞吐、反压与提交周期等微架构约束，编译器无法替设计者自动决策。

由此，我们的研究视角发生了一次关键跃迁——不再试图“把任意软件语义的 gem5 代码硬译为 RTL”，而是提出了一个更具长期工程价值的命题：

> **能否采用以硬件语义为中心的嵌入式 C++ 硬件构造层（HCL），让开发者用受限的 C++ 原语编写单一硬件描述，在执行期动态构建统一硬件中间图，同时驱动“gem5 原生高速周期仿真”与“可综合 Verilog/SystemVerilog 自动生成”？**

---

## 2. 开源底座选型：为什么主选复用 Kathryn C++？

前期我们通过独立自研轻量 HCL 原型，已经验证了“C++ 执行期动态构图 → 双后端输出”这一范式在 GW100 上的早期闭环可行性。但若从零自主演进一套完整的 HCL 框架，后续将不得不承担大量语法前端、复杂控制流降级与编译优化的长期维护成本。因此，下一阶段的核心工程策略是：**以成熟开源项目为底座进行复用搭建，避免重复造轮子**。

在横向调研与评估五个代表性开源体系后，我们明确选定 **[Kathryn C++](https://www.kathryn-tools.org/cppbook/getting-started/introduction/)** 作为 A 阶段**首要复用的核心底座**，同时吸收 **[Intel ICSC + SingleSource](https://easychair.org/publications/preprint/91Rs/open)** 的空拍跳过理念与 **[NVIDIA MatchLib](https://github.com/NVlabs/matchlib/blob/master/README.md)** 的通道反压验证范式：

| 候选体系 | 宿主与构建机制 | 与 gem5 对接成本 | 开源 RTL 生成闭环 | 定位与工程取舍 |
|---|---|---|---|---|
| **[Kathryn C++](https://www.kathryn-tools.org/cppbook/getting-started/introduction/)** | 纯 C++ 嵌入式框架（Framework-Assisted HDL），Elaboration 期动态构建统一内存图（`src/model/`） | **极低**：同源 C++，原生编译输出 `.so` 可直接挂入 `SimObject` | **完整**：内置 `src/gen/` 直接输出可综合 Verilog | **首选主复用底座**：范式同构、无跨语言壁垒、无 SystemC 协程拖累、无商业许可证绑定 |
| **[Intel ICSC + SingleSource](https://github.com/intel/systemc-compiler)** | Clang/LLVM 18 编译器前端，解析 SystemC 3.0.1（[SingleSource](https://easychair.org/publications/preprint/91Rs/open) Fast 模式提速 4×–8×） | 中等：强绑定 Accellera SystemC 协程内核，接入 gem5 需额外桥接 | 强：工业级 SystemVerilog 输出 | **借鉴思想（不作主底座）**：吸收其双精度/空拍跳过思路，避免 SystemC 协程开销与 Linux/WSL 强绑定 |
| **[NVIDIA MatchLib](https://github.com/NVlabs/matchlib/blob/master/README.md)** | SystemC/C++ `Connections` 延迟不敏感通道库，支持 `SIM_MODE=1/2` 与 `RAND_STALL` | 中等：仍依赖 SystemC 仿真环境 | 弱（开源侧）：综合强绑定商业 Siemens Catapult HLS | **借鉴验证与通道范式**：吸收其 `Valid/Ready` 通道语义与万周期 `RAND_STALL` 随机反压压测方法 |
| **[PyMTL3](https://www.csl.cornell.edu/~cbatten/pdfs/jiang-pymtl3-ieeemicro2020.pdf)** | Python 多层硬件建模（FL/CL/RTL）+ IMIR 转换 Pass | 高：Python 与 gem5 C++ 之间存在跨语言 FFI 调用开销 | 成熟：支持自动翻译输出 Verilog | **重要跨语言参照**：验证了上层统一 IR 驱动多后端的通用性，但跨语言墙不适合高频 `CtWindowHandler` |
| **[PandA Bambu HLS](https://github.com/ferrandi/PandA-bambu/blob/main/README.md)** | Apache-2.0 开源 C/C++ HLS（Clang/LLVM 16–19，`ac_types`、`hls::stream`、AXI4-Master） | 需通过 Verilator 联仿或外部封装接入 | 强：面向算法级数据流综合 | **互补工具（非控制面底座）**：适合 `CtMatrix`/`CtVector` 等计算数据通路，不适合精确周期级 FSM 控制 |

### 主选复用 Kathryn C++ 的四大硬核依据

1. **与 gem5 同源 C++，避开跨语言、协程与商业授权三大硬伤**
   Kathryn C++ 直接以标准 C++ 为宿主语言（与 PyMTL3 相比无 Python-C++ FFI 墙，与 ICSC/MatchLib 相比无 SystemC 协程内核调度开销，也不依赖 Catapult 等商业 HLS 工具），天然契合 gem5 的 C++ 编译与链接生态。
2. **编译型 Hybrid Simulator（`.so`）直接击中我们实测的吞吐瓶颈**
   在前期自研原型的 50,000 周期基准测试中，纯 C++ IR 解释器吞吐为 **2,530 kcycles/s**，而 Verilator 编译态模型达到 **16,324 kcycles/s**（快约 6.45×）。根据 [Kathryn 官方架构文档](https://www.kathryn-tools.org/devbook/architecture/overview/)，其 `src/sim/`（Hybrid Simulator）并非直接解释执行图，而是**从内存模型生成 C++ 代码并编译为原生 `.so` 共享库动态加载运行**（同时自带 VCD 波形与 ZEP 零侵入周期分析器），直接替我们省去了自研 JIT/C++ 代码生成器的庞大编译器工程量。
3. **三大硬件原语精准覆盖 GW100 控制与 DMA 需求，且有超标量 CPU 级背书**
   - **Hybrid Design Flow (HDF)**：提供 `seq`、`par`、`cwhile`、`pip`/`zync`、`ztate` 等控制流抽象，用 `<<=`（边沿赋值）与 `=`（组合赋值）严格分离时序与组合语义，不靠 HLS 猜测微架构。
   - **Decentralized Update**：支持多逻辑块向同一资源发起更新，由目标端按显式优先级池自动仲裁（复位天然高于普通写），避免手写庞大的中心化 FSM。
   - **Hardware Aggregator**：提供 `SlotMeta`、`RegSlot`/`WireSlot` 与 `Table` 抽象，直接对应 `CtSync` 的 32 组同步寄存器阵列和 `CtNidma` 的乱序 Tag Table。
   - **复杂度背书**：据 [Kathryn 官网与 IEEE TCAD 2026 论文](https://www.kathryn-tools.org/)（DOI: `10.1109/TCAD.2026.3738973`），C++ 版 Kathryn 已完成 **Kride（6 级乱序超标量 RISC-V CPU）** 的设计与实现，与 RIDECORE 达成周期级对齐并在 **Kria KV260** FPGA 上完成综合验证，复杂度远超当前 GW100 控制与 DMA 模块。
4. **明确复用“C++ 原版”而非新版 Rust+Python（Kathryn2）的客观依据**
   [Kathryn 官方安装指南](https://www.kathryn-tools.org/userbook/getting-started/installation/) 与 [架构文档](https://www.kathryn-tools.org/devbook/architecture/overview/) 均明确说明：基于 Rust + Python 的重写版本（PyPI `kathryn` v0.1.0）仍在活跃开发中，原 C++ 版的多个子系统尚未移植完毕，也尚未通过论文同等级别的验证；**当前唯一经过完整论文级验证的稳定版本仍然是 C++ 版**。

---

## 3. 基于 Kathryn C++ 复用的 GW100 融合架构与设计规范

融合路线的核心原则是：**复用的 Kathryn C++ 负责“C++ 动态构图 → 编译态 `.so` 仿真 + 可综合 Verilog 生成”的底层重引擎；GW100 侧仅维护薄的 gem5 适配外壳与独立的外部四道质量门禁。**

```text
               Kathryn C++ 硬件建模层 (src/model/)
       HDF 控制流 (seq/par/pip) · Decentralized Update · Table
                                |
                                v  (Elaboration 期构建统一内存硬件图)
             =========================================
             ||     Kathryn 统一内存硬件中间模型      ||
             ||  - 显式时序 (<<=) 与组合 (=) 分离     ||
             ||  - 去中心化写优先级池与复位仲裁        ||
             ||  - 静态位宽检查与资源表管理            ||
             =========================================
                    /                          \
                   v                            v
       src/sim/ Hybrid Simulator          src/gen/ Verilog 后端
  - 自动生成 C++ 并编译为原生 .so         - 自动输出可综合 Verilog
  - 接近 Verilator 的原生执行速度         - 严格对齐外部冻结契约端口
  - 内置 VCD 波形与 ZEP 周期分析
                   |                            |
                   v                            v
         GW100 gem5 薄适配层               独立外部验证与综合链
    (CtWindowHandler / TimingPort       (G1 契约与 Lint 门禁 /
     Retry 反压 / 空拍跳过门控)          G2 双后端锁步 / G3 独立记分板 /
                   |                     G4 Yosys 真实逻辑综合)
                   v                            v
          gem5 周期级仿真闭环                物理可综合 RTL 闭环
```

### 在 Kathryn C++ 之上坚持的四项 GW100 工程规范

1. **统一中间模型与去中心化写优先级仲裁**
   - 严禁 C++ 仿真侧与 Verilog 生成侧各自重新解释用户代码；所有多源写冲突（例如 `CtSync` 中 MMIO 写 > `local_sig` > `local_wait`）必须通过显式优先级声明统一收敛，从源头杜绝双后端语义漂移。
2. **双阶段周期语义与 gem5 侧空拍跳过（`canClockGate()`）**
   - 硬件核心内部严禁出现 gem5 的 `Tick`、`curTick()` 或 `wakeAt()`，所有状态跃迁由显式时钟沿统一提交。
   - 针对 gem5 全系统稀疏访存场景，在外层适配层引入可形式化校验的空闲门控机制（仅当模块内部无活跃事务且输入无效时跳过空拍），兼顾事件驱动效率与周期级精度。
3. **构造期与运行期严格分离 + 大容量 SRAM 黑盒化**
   - 小型寄存器堆（如 `CtSync` 32×32b、Tag Table）使用 `RegArray`/`Table` 综合为触发器；片上大容量存储器（如 GW100 2 MiB 16-Bank `CtSram`）必须通过显式端口数、读写延迟、字节写掩码与冲突策略的黑盒 `SramMacro` 封装，严禁被综合器展开为触发器阵列。
4. **独立四道外置门禁（杜绝“自证清白”的共模错误）**
   - 即使仿真 `.so` 与 Verilog 来自同一份内存模型，若建模本身违背硬件规格，双后端仍会“同时出错且彼此一致”。所有模块产出必须通过独立外部四道门禁：
     - **G1 静态契约与 Lint 门禁**：核验端口、位宽、禁用语法及 `Verilator --lint-only -Wall`；
     - **G2 双后端逐拍锁步对拍**：验证原生 C++ 仿真与 Verilator 编译生成 RTL 的引脚级完全一致；
     - **G3 独立规格裁判**：对照独立五层硬件契约、冻结 gem5 参考模型与独立黄金记分板（如 `std::deque`、乱序 Tag 记分板）排除共模 Bug；
     - **G4 真实综合门禁**：接入 [Yosys](https://yosyshq.readthedocs.io/projects/yosys/en/latest/using_yosys/synthesis/synth.html) 执行逻辑综合，与独立手写 RTL 对比面积与资源开销。

---

## 4. 前期独立原型的全量可靠性闭环实证

在正式切入 Kathryn C++ 复用之前，我们通过自研轻量 HCL 原型完成了覆盖 **静态 IR 冲突校验、`CtSync`、`Fifo4`、`SramMacro` 与 `CtNidma` 乱序切片** 的完整可靠性闭环。这组实测不仅证明了“C++ 动态构图双后端”在 GW100 典型模块上的表达力与正确性，也为转向复用 Kathryn C++ 提供了关键的量化依据：

| 验证维度 | 测试对象与规模 | 实测结果 | 工程含义 |
|---|---|---|---|
| **静态 IR 冲突校验** | 注入同寄存器重复写优先级反例 | 在构造期即抛出 `IR_VERIFY_ERROR` 拦截非法多写冲突 | 验证了去中心化优先级池可在构建期阻断隐蔽写竞争 |
| **生成 RTL 紧凑度与 Lint 质量** | `CtSync`、`Fifo4`、`CtSram` 切片、`CtNidma` 切片 | `CtSync` 仅 **4,756 字节**（比手写 `ct_sync.sv` 5,519 字节更紧凑）；`Fifo4` 1,770 字节；`CtSram` 切片 1,758 字节；`CtNidma` 切片 **5,811 字节**（接近独立手写 5,040 字节）；全部通过 `Verilator --lint-only -Wall` **0 警告** | 证明动态图 + 优先级池生成策略在复杂主动模块上同样紧凑可读，无代码膨胀失控 |
| **同步寄存器堆与空拍跳过** | `CtSync`（857 周期锁步 + 2,000 周期稀疏负载） | **857 周期 100% 引脚锁步一致**；2,000 周期下 `canClockGate()` 跳过 **1,444 拍（72.2%）** 且 0 差异；原生 C++（零 Verilator）直接跑通全 9 模块 Compute Tile（`[121, 256]` + 8 轮串联） | 证明被动控制单元可在 gem5 中实现零 Verilator 集成与安全空拍跳过 |
| **通道反压与三方独立校验** | `Fifo4`（10,000 周期 50% `RAND_STALL` + 107 次飞行中复位） | 完成 **4,410 次入队、4,202 次出队、1,807 次同拍同时读写**，**HCL C++ == Verilator SV == 独立 `std::deque` 黄金记分板 100% 一致**（0 丢包/0 重复/0 乱序） | 攻克流控与反压门槛，并用独立裁判排除共模错误 |
| **同步字节掩码 SRAM 宏** | `CtSram` 256×64 位单端口切片（8 位字节掩码、`ReadOld`、2,000 周期） | 生成独立 `sram_sp` 宏封装（未误展开为触发器），**767 次写 + 741 次读 + 5 次复位**，**HCL C++ == Verilator SV == 独立字节数组黄金模型 100% 精确一致** | 证明片上大容量存储可通过黑盒宏抽象干净落地 |
| **多拍乱序 DMA 压力闭环** | `CtNidma` 切片（64B 包上限、256B Bank 边界切分、4 槽乱序 Tag Table、50% `RAND_STALL`、粘性错误） | **5 组定向边界 + 60 组随机传输**（共 **246 个 64B 包、55 次乱序 Tag 回收**），**HCL C++ == Verilator SV == 独立乱序记分板 100% 精确一致** | 证明嵌入式 C++ 硬件抽象能够稳定表达带多拍 FSM、乱序 Tag 与边界切分的主动搬运模块 |
| **真实墙钟吞吐基准（推动复用 Kathryn 的关键依据）** | `CtNidma` 切片连续 50,000 周期（`-O2` 编译） | 自研 IR 解释器 **2,530 kcycles/s**（19.76 ms）vs. Verilator 编译态 SV **16,324 kcycles/s**（3.06 ms，快 **~6.45×**） | **直接证明了复用 Kathryn C++ 的必要性**：解释器在重负载下存在明显性能短板，复用 Kathryn 内置的 C++→`.so` 编译态 Hybrid Simulator 是最经济的高性能解法 |

---

## 5. 在 GW100 中的落地分工、工程边界与后续三步推进

### 5.1 务实的混合架构分工
HCL 路线不是为了强行替换整个 GW100 仓库，而是与 **契约驱动独立 RTL（BC 路线）** 和 **专用数据通路流程** 形成优势互补，共同锚定在统一的五层 Hardware Contract 之下：

- **新设计的控制、同步、队列、仲裁与轻量 FSM/DMA 子模块**（如 `CtSync`、FIFO、Tag Table、仲裁器、寄存器管理器）→ **优先采用 Kathryn C++ 复用路线**，以单一 C++ 硬件资产同时服务 gem5 仿真与 Verilog 生成。
- **既有已成型的复杂行为级 gem5 模块**（如现有 `CtNidma`、`CtCtrl`）→ **保留 BC 路线**，避免推倒重写已有 gem5 资产，通过冻结 Spec、独立 RTL 与差分验证保障一致性。
- **高吞吐张量与向量计算引擎**（`CtMatrix`、`CtVector`）→ **采用专业 HLS（如 Bambu / Catapult）或手写 RTL**，再经在线适配层接入 gem5。

### 5.2 双版本源码实测对比与 Kathryn C++ Fork 改造清单

为给出精确到文件级的 Fork 改造方案，我们同时检出并实读了 Kathryn 的两个官方分支——**C++ 版（`master`，commit `4d703532`）**与**最新 Rust + Python 版（`origin/kathryn-rust-py`，commit `6915de71`）**，并在本地 macOS（Apple Clang 21.0.0，无需额外安装 CMake/Cargo）完成了 Kathryn C++ 核心库（149 个核心 `.cpp` 文件）的实测编译与最小端到端烟雾闭环（`SmokeCnt` → 5,274 B `smoke_top.v` + 5.6 KB `smokeProxyGen.cpp` → 88 KB `smokeProxyGen.so`）。

#### 双版本实测结论：C++ 做主干 + 反向移植 Rust+Python 版 5 项关键修复
- **为何当前仍以 C++ 版为 Fork 主干**：`Kathryn-rust-py` 的 `CLAUDE.md §4.2` 明确标注其原生编译型仿真引擎 `ModuleSimEngine` **尚未实现**，目前仿真仅依赖 Python + `cocotb` + Verilator/Icarus；而 C++ 版核心库在本地 **148/149 个文件一次编译通过**，且能端到端同时产出可综合 Verilog 与原生 `.so` 仿真库。
- **从 `Kathryn-rust-py` 反向移植（Backport）的 5 项高价值设计**：
  1. **父模块子输出声明为 `wire`（非 `reg`）与 1-bit 标量规则**：消除层次化设计和 Verilator `-Wall` 下的 `[0:0]` 向量及驱动类型报错。
  2. **按递增 Global ID 确定性排序 IO**：保证同一设计两次生成的 `.v`/`.cc` 字节一致，支撑 SCons/CMake 增量缓存。
  3. **`sim_manifest.json` 符号映射表**：在代码生成期同步导出“模型属性名 → C++ 成员名 / Verilog 信号名”映射，直接驱动 5 层契约检查与 Verilator 锁步。
  4. **`wire.default()` 优先级下移至用户写之下 + `Table`/`Karray` 整表复位**：补齐 `CtSync` 32 槽清零与 `CtNidma` Tag Table 复位能力。
  5. **`pip(auto_restart)` 中 `SET` 优先级高于 `RST`**：防止流水线同拍 flush + 重入时进入自锁死零态。

#### C++ 版实测定位的源码级 Fork 改造矩阵

| 优先级 | 实测定位文件 / 模块 | 实测暴露的具体问题与改造点 | 改动侵入度 |
|---|---|---|---|
| **P0（阻塞落地）** | `src/sim/` 生成代码（实测 `smokeProxyGen.cpp`） | **实测确认生成寄存器/线网均为 `namespace kathryn` 下的全局静态变量**（如 `uint8_t REG10_USER_cnt = 0;`），无法多 Tile 实例化 → 必须封装进独立状态类，并生成强类型端口 setter/getter 替代 `unordered_map` 字符串查表 | 中 |
| **P0（阻塞落地）** | `src/sim/controller/simController.cpp`、`simInterface.h`、`modelCompile/startGen.sh` | 绕过全局单例 `SimController` 主循环与 `conThread` 线程，暴露同步单拍 `stepCycle()` 给 gem5 `EventQueue`；将运行期 `startGen.sh`（缺 `-std=c++17`、`set -e`、含 `sleep 1` 且 macOS 缺 `-undefined dynamic_lookup`）改为构建期 AOT 静态编译；修复 `proxyBuildMng.cpp:724` 在 `SPB_GEN` 析构时 `unloadProxy` 断言崩溃 | 中 |
| **P0（阻塞落地）** | `src/sim/event/userEvent.h:17`、`CMakeLists.txt`、`src/gen/` | 清理无前缀全局宏 `#define sim`（与 gem5 严重冲突）；将单体 `CMakeLists.txt` 拆出 `libkathryn_core` 静态库；修复 `startGenKathryn()` 反向修改内存模型导致同进程无法连续 Gen+Sim 的问题；支持顶层原端口名保留与低有效 `rst_n` 配置（当前默认输出 `WIRE8_en` 和高有效 `rst`） | 低-中 |
| **P1（工程刚需）** | `src/gen/proxyHwComp/memBlock/memGen.cpp`、`src/sim/` | 实测 `MemGen::decVariable()` 仅吐 `reg [W-1:0] MEM [D-1:0]`，需扩展字节写掩码（`wstrb`）与 ASIC 黑盒 `sram_sp` 例化模式以支撑 `CtSram`；在 `ProxySimEventBase` 补齐 `isIdle()` / `canClockGate()` 空拍跳过接口并默认关闭高开销 VCD/ZEP | 中 |
| **P2（质量与兼容）** | `src/lib/instr/instrEle.cpp`、`valRep.h:287`、`src/gen/` | 修复 `instrEle.cpp:361` 的 GCC 专属 VLA 与非 `const` 比较符（唯一未过 Apple Clang 的文件）；修复 `valRep.h:287` 移位未赋值 Bug（`>>` 误写，应为 `>>=`）；消除生成 Verilog 中的 2 处内部 `UNUSEDSIGNAL` 告警以达成 `Verilator -Wall` 0 警告 | 低 |

#### Fork 维护红线（严禁触碰的边界）
- **不魔改核心降级与仲裁引擎**：`HDF` 控制流（`seq`/`par`/`pip`/`zync`）到状态节点图的转换，以及 `Decentralized Update` 写优先级仲裁内核已经过 6 级乱序 CPU `Kride` 验证，应原封不动复用。
- **不向 Kathryn 内部引入 gem5 依赖**：所有 `SimObject`、`TimingPort` Retry 队列、地址译码与窗口回调逻辑留在 GW100 侧的薄适配层，保持 Fork 仓库独立可测。

### 5.3 后续三步工程化推进计划

1. **完成 P0 最小 Fork 补丁与 `CtSync` 迁移闭环**：在内部 fork 分支落地“AOT 状态类对象化 + 移除 `#define sim` + 顶层端口/复位契约化 + `libkathryn_core` 拆分”，将 `CtSync` 迁移至 Kathryn C++，跑通 gem5 适配层、5 层契约门禁与 Verilator 锁步，并实测 50,000 周期吞吐加速比。
2. **补齐 P1 扩展与复杂模块联调（`CtNidma` + `SramMacro`）**：在 `CtSync` 闭环稳定后，补齐 `canClockGate()` 空拍门控与 `SramMacro` 黑盒原语，推进 `CtNidma` 乱序切片迁移及全 Compute Tile 稀疏负载开销评估。
3. **接入 Yosys 真实逻辑综合门禁并持续跟踪 `Kathryn-rust-py`**：引入开源 [Yosys](https://yosyshq.readthedocs.io/projects/yosys/en/latest/using_yosys/synthesis/synth.html) 对 Kathryn `src/gen/` 自动生成的 Verilog 与独立手写 SV 执行 `synth; stat` 横向对比；中长期持续关注 `Kathryn-rust-py` 的 `ModuleSimEngine` 移植进度，待其具备原生 AOT C++ 仿真导出能力后评估平滑切换。
