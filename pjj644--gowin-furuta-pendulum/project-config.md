---
trigger: always_on
description: > **给接手本项目的 AI agent 的入口文档。**
---

# AGENTS.md

> **给接手本项目的 AI agent 的入口文档。**
> 目标：让一个从未接触本项目的 agent 在十分钟内知道代码在哪、怎么验证、现在什么状态、什么绝对不能做。
> 人类读者请从 [`doc/README.md`](doc/README.md) 开始。

---

## 0. 三十秒速览

| | |
| :--- | :--- |
| **项目** | 全国大学生嵌入式芯片与系统设计竞赛 2026 · FPGA 创新设计赛道 · **选题一：基于 FPGA 的实时姿态控制系统** |
| **被控对象** | 旋转倒立摆（Furuta pendulum）—— J280 姿态控制系统竞赛套件 |
| **目标器件** | 高云 **GW2A-LV55PG484C8/I7**（GW2A-55C，PBGA484，speed grade 8） |
| **EDA 工具** | Gowin EDA **V1.9.12.03**（综合/PnR/bitstream）+ ModelSim **SE-64 10.7**（仿真） |
| **当前状态** | ✅ 官方 FAQ（90g 摆杆/90g 摆臂/WDD35D4 传感器）参数闭环实测达成；赛题基础要求 1/2/3、拓展要求 1/2/3 **全部闭环实测达成**（10/10 项判据全部 PASS，77 项单元与集成测试全部 PASS，IC Coder 审查缺陷已全部闭环） |
| **权威源码** | `furuta_lqr_ctrl\`（本目录下**嵌套的独立 git仓库**，`main @ ed03166`） |
| **文档仓库** | `C:\Users\28399\Desktop\GoWin`（`.gitignore` 已排除 RTL 子仓库） |
| **路径状态** | 仓库已迁移至纯 ASCII 路径 `C:\Users\28399\Desktop\GoWin`，**Gowin 与 ModelSim 均已可直接原生运行**；`furuta_lqr_ctrl\eda.ps1` 自动直连（见 §1.2） |
| **待办总览** | R1/R4/R7 及 IC Coder 审查缺陷已闭环；剩余板级硬件澄清 R6、R8~R10，见 [`doc/agent/选题一_RTL代码合规性检查报告.md`](doc/agent/选题一_RTL代码合规性检查报告.md) §五 |

---

## 1. 目录结构、两个 git 仓库与「单一事实来源」

一个工作区目录树，内含**两个独立的 git 仓库**（物理嵌套，但**不是** submodule）：

```
C:\Users\28399\Desktop\GoWin\            ← 文档仓库（git main）
├── AGENTS.md                          本文件
├── .gitignore                         已排除 furuta_lqr_ctrl/
├── doc\{human,agent}\                 文档（分类标准见 doc/README.md）
├── simulation\                        Python 黄金模型 + ⛔ 已失效的 RTL 副本
├── 赛题要求和芯片数据手册\
├── images\
└── furuta_lqr_ctrl\                   ← RTL 主仓库（独立 git main @ ed03166）
    ├── src\          15 个文件：8 可综合模块 + CST + SDC + 3 TB + 2 HIL
    ├── sim_modelsim\ run_all.bat / compile.do / modelsim.ini
    ├── eda.ps1       ★ EDA 统一入口（自动维护 ASCII junction）
    ├── build.tcl     Gowin 构建脚本（自定位，不含绝对路径）
    ├── furuta_lqr_ctrl.gprj
    └── impl\         综合产物（已忽略）
```

### 1.1 ⛔ 不要 `git add furuta_lqr_ctrl`

它是**嵌套的独立仓库**，不是 submodule。文档仓库的 `.gitignore` 已排除它。
若强行 add，git 会记成一个 gitlink（伪 submodule），克隆者只得到一个空目录，
且 RTL 的 17 个提交历史（五轮迭代的完整证据链）无法随之传递。
**两个仓库各自 `git status` / `add` / `commit` / `log`。**

### 1.2 EDA 路径状态与统一入口 `eda.ps1`

本项目曾因目录名含中文（`...\赛道\...`）导致 Gowin（读成 `??` 报 SP0002 错误）与 ModelSim（SQLite 打不开报错）无法直接运行。
**现状**：工程根目录已重命名迁移至纯 ASCII 路径 **`C:\Users\28399\Desktop\GoWin`**。实测两个 EDA 工具均已支持直接原生运行。

为了统一工具链调用并提供跨环境兼容性，推荐统一经由 PowerShell 脚本 `eda.ps1` 执行：
- 若当前工作目录为纯 ASCII 路径（当前即为 `C:\Users\28399\Desktop\GoWin\furuta_lqr_ctrl`），`eda.ps1` **自动检测并直接使用真实路径**，无需任何额外配置；
- 若未来工程被移动到含中文或特殊字符的路径，`eda.ps1` 内置的 NTFS junction（`C:\fpga_build`）**自动回退兜底机制**将自动接管。

```powershell
cd C:\Users\28399\Desktop\GoWin\furuta_lqr_ctrl
.\eda.ps1 build      # 综合 + PnR + 时序 + bitstream
.\eda.ps1 regress    # ModelSim 全量回归（5 步）
.\eda.ps1 doctor     # 自检：路径 ASCII 性 / 工具链 / 关键文件 / modelsim.ini
.\eda.ps1 path       # 打印当前使用的工作路径
```

详见 [`furuta_lqr_ctrl/eda.ps1`](furuta_lqr_ctrl/eda.ps1) 头部说明与踩坑记录 A9。

### 1.3 ⛔ `simulation/src_verilog/` 是历史副本，已漂移，禁止修改

该目录下有 12 个 `.v` 文件，是项目早期从 RTL 仓库拷来的快照，**不参与综合、不参与验证、不与 RTL 主仓库同步**。它们与权威源码已存在实质差异（例如缺少 HIL 平台、缺少后续五轮的全部缺陷修复）。

在这里改代码 = 改动完全丢失 + 制造第三份漂移副本。**所有 RTL 修改必须落在 `furuta_lqr_ctrl\src\`。**

`simulation/` 里**仍然权威**的部分是 Python 黄金模型：

| 文件 | 作用 |
| :--- | :--- |
| `config.py` | 全部物理常数与 LQR 权重的唯一定义处 |
| `dynamics.py` | 欧拉-拉格朗日动力学（HIL plant model 的对标来源） |
| `controller.py` | LQI + 能量泵起摆 + FSM 的浮点参考实现 |
| `test_suite.py` | 6 项算法级验证，HIL 判据阈值的来源 |
| `fpga_fixed_point_guide.py` | 定点化推导与代码生成 |

---

## 2. 必读文档（按此顺序）

| 顺序 | 文档 | 为什么先读它 |
| :---: | :--- | :--- |
| **1** | [`doc/agent/踩坑记录.md`](doc/agent/踩坑记录.md) | 已经付出真实代价的坑。**不读就会重犯**，其中至少三个坑各消耗过一整轮迭代 |
| **2** | [`doc/agent/选题一_RTL代码合规性检查报告.md`](doc/agent/选题一_RTL代码合规性检查报告.md) | 第五轮报告（@`1aa5f2c`）= **历史存档**；当前基线已演进至 `ed03166`，最新实测数字以本文 §4.3 为准。§四 达标判定、§五 待办 R1~R15、§7.2 复现命令 |
| **3** | [`赛题要求和芯片数据手册/选题一_基于FPGA的实时姿态控制系统.md`](赛题要求和芯片数据手册/选题一_基于FPGA的实时姿态控制系统.md) | 需求原文。设计要求 4 项 / 基础要求 3 项 / 拓展要求 3 项 |
| 4 | `doc/human/*` | 原理与实操，**按需查阅**，不必通读 |

> ⚠️ `doc/agent/选题一_RTL代码复检报告.md` 是**第三轮历史存档**，其结论已被第五轮报告取代。**不要据此判断现状。**

---

## 3. 系统架构

### 3.1 控制链路

```
                    ┌──────────── 50 MHz 系统时钟 ────────────┐
                    │                                        │
 摆杆角度 ─SPI 2.5MHz─► angle_sensor_reader ─┐               │
   (12-bit ADC)         · 360° 解卷绕        │               │
                        · 最短角位移解卷绕    │               │
                        · 一阶 IIR (β=0.35)  ├─► j280_hw_top ─┼─► motor_pwm_driver ─► TB6612FNG ─► 直流电机
                        · 差分测速           │   · ctrl_fsm   │     · 20 kHz            · STBY/IN1/IN2
                                             │   · traj_gen   │     · 影子寄存器         · 刹车 / 滑行
 转臂角度 ─正交 A/B──► encoder_quad_reader ──┘   · swing_up   │
   (4000 CPR)           · 4 倍频鉴相            · furuta_lqr │
                        · 8 拍数字消抖           · LQI 积分器 │
                        · M 法测速                            │
                                                             │
 KEY1(标定) KEY2(模式) sw_motor_en ──────────────────────────┘
```

### 3.2 模块清单

| 模块 | 文件 | 职责 | 关键点 |
| :--- | :--- | :--- | :--- |
| `j280_hw_top` | `src/j280_hw_top.v` | **综合顶层**，20 个 I/O | LQI 积分器、软限位、按键消抖、模式仲裁 |
| `angle_sensor_reader` | `src/angle_sensor_reader.v` | SPI 主机 + 角度解算 | 3 拍流水；`process_trigger` 优先级高于 `filter_step`（见 R13） |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pjj644/gowin-furuta-pendulum](https://github.com/pjj644/gowin-furuta-pendulum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-03 -->
