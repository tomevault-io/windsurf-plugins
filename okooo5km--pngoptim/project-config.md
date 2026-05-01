---
trigger: always_on
description: > 依据文档：`docs/RECONSTRUCTION_MASTER_PLAN.md`、`docs/PHASE_TASKLIST.md`、`docs/EVALUATION_SPEC.md`、`docs/ACCEPTANCE_MATRIX.md`
---

# PNGOptim 项目阶段记忆（AGENTS）

> 最后更新：2026-03-06  
> 依据文档：`docs/RECONSTRUCTION_MASTER_PLAN.md`、`docs/PHASE_TASKLIST.md`、`docs/EVALUATION_SPEC.md`、`docs/ACCEPTANCE_MATRIX.md`

## 1. 项目开发目的（统一认知）

本项目目标是用 Rust 在单仓库内实现一个可发布的 PNG 量化压缩工具，并按“先复刻、后优化”的策略达到或超过对标工具（pngquant/libimagequant）的工程能力：

1. 功能与 CLI 语义高兼容（参数、退出码、I/O、元数据策略）。
2. 体积、质量、性能达到统计意义等价或更优。
3. 跨平台稳定（macOS / Linux / Windows）且可复现。
4. 全过程以可自动化评测和报告驱动，不做无数据优化。
5. 在核心量化算法上优先对齐 `pngquant` / `libimagequant` 的实现思路：允许编码实现不同，但 `quality`、palette search、remap、dither 的主链应一致。

当前明确非目标：
1. 不追求 bit-exact 输出。
2. 不做 GUI。
3. 不扩展 PNG 之外格式。
4. v1 不引入复杂 ML 压缩策略。

## 1.1 工程约束与决策锁定

1. 仓库主线、CI 编排、发布资产生成统一使用 Rust；不得把 Python 重新引入为项目运行时或主线编排依赖。
2. 临时本地分析命令不视为项目资产；若使用一次性 shell/Python 片段做实验，禁止提交进仓库，也不得让文档、workflow、CLI 依赖该运行时。
3. 算法目标不是“完全自己发明一套”，而是尽可能对齐 `pngquant` / `libimagequant` 的成熟实现思路。
4. 目前不直接复制或链接参考实现代码进入主线，不是出于教条式自研，而是因为当前项目已按 MIT 发布，直接引入参考实现代码需要先明确许可证策略与仓库治理方案。
5. 如果后续决定直接复用参考实现代码，必须先把许可证、分发方式、仓库结构和发布策略记录进文档，再执行代码接入。
6. 后续所有算法复刻工作采用 `reference-first` 纪律：先对照本地参考仓库对应模块提炼实现细节、启发式和边界条件，再编码；禁止脱离参考实现结构凭感觉连续打补丁。

## 2. 分阶段开发规划（不设周期，仅阶段）

执行顺序固定：A -> B -> C -> D -> E -> F -> G -> H

### 阶段 A：治理与基线先行
- 目标：先定规则与基线，再进入实现。
- 关键任务：
1. 冻结主方案版本与文档先行机制。
2. 建立依赖准入与许可证审查流程。
3. 建立评测数据集、参数矩阵、报告规范。
4. 产出并冻结 baseline 报告。
- 阶段出口：`Compliance Policy v1` + `Baseline Report v1` + `Acceptance Matrix v1`
- 当前状态：`Done`

### 阶段 B：最小可运行闭环
- 目标：端到端可跑（读 -> 量化 -> 写），且基础稳定。
- 关键任务：
1. 打通最小 pipeline。
2. 完成 CLI 初版（单文件、基础参数、错误码框架）。
3. 全样本 smoke 测试通过且无崩溃。
- 阶段出口：`MVP Pipeline` + `Smoke Report v1`
- 当前状态：`Done`

### 阶段 C：行为语义复刻
- 目标：工具使用语义与对标工具一致。
- 关键任务：
1. 参数语义对齐（quality/speed/dither/output/ext/strip/skip-if-larger/posterize）。
2. 退出码与错误语义对齐。
3. stdin/stdout、批处理、覆盖策略对齐。
4. 元数据策略对齐。
- 阶段出口：`Compatibility Report v1` + 行为差异清单
- 当前状态：`Done`

### 阶段 D：质量与体积复刻
- 目标：核心压缩能力达到对标水平。
- 关键任务：
1. 质量指标达标（SSIM/Butteraugli/PSNR 组合门禁）。
2. 体积指标达标（均值/中位数/P95）。
3. 专项场景修复（低色、透明边缘、UI/渐变）。
4. 失败样本闭环清理（持续下降或清零）。
- 阶段出口：`Quality & Size Report v1`
- 当前状态：`Done`

### 阶段 E：性能优化冲刺
- 目标：形成可量化性能优势。
- 关键任务：
1. 模块级耗时与内存可观测。
2. 搜索/抖动/写出热点逐项优化。
3. 引入 SIMD 与并行调度策略。
4. 每次优化必须回归质量与体积门禁。
- 阶段出口：`Perf Report v1` + 资源画像报告
- 当前状态：`Done`

### 阶段 F：稳定性与跨平台收口
- 目标：达到发布候选质量。
- 关键任务：
1. 回归 + fuzz 零崩溃。
2. 三平台一致性回归（macOS/Linux/Windows）。
3. 可复现构建与 RC 门禁落地。
- 阶段出口：`RC Candidate` + `Stability Report v1` + `Cross-platform Report v1`
- 当前状态：`Done`

### 阶段 G：开源发布与社区协作
- 目标：具备可持续开源协作能力。
- 关键任务：
1. 发布文档、评测脚本、样本说明、许可证声明。
2. 建立贡献规范、Issue/PR 模板和回归流程。
3. 建立长期性能回归与样本扩充机制。
- 阶段出口：`Public Release v1`
- 当前状态：`Done`

### 阶段 H：APNG 动图压缩优化（超越阶段）
- 目标：在保持静态 PNG 主线稳定的前提下，补齐 APNG 解析、重组、结构优化与有损量化优化，形成相对当前对标工具的能力超集。
- 关键任务：
1. 完成 APNG 格式研究与实现约束冻结（`acTL` / `fcTL` / `fdAT`、default image、dispose / blend、sequence number、全局色彩约束）。
2. 建立 APNG 解析、画布合成、帧矩形与时序模型，保证可正确 round-trip。
3. 先实现 lossless APNG 结构优化（重复帧折叠、帧矩形裁剪、filter / deflate / metadata 策略）。
4. 再把当前静态 PNG 的 palette search / remap / selective dithering 主链提升为 animation-aware 全局量化器。
5. 建立 APNG 数据集、质量/体积/性能门禁与跨平台回归。
- 阶段出口：`APNG Optimization Plan v1` + `APNG Compatibility Report v1` + `APNG Optimization Report v1`
- 当前状态：`Blocked`

## 3. 验收门禁（阶段推进依据）

1. `MVP` 门槛：DOD-01/02/03/10/12 通过。
2. `复刻` 门槛：MVP + DOD-05/06/07 通过。
3. `发布候选` 门槛：复刻 + DOD-08/09/11 通过。
4. 规则：任一 P0 失败不可合并；P1 豁免必须登记风险和关闭条件。

## 4. 进度记录（持续更新区）

### 阶段状态总览
| 阶段 | 状态 | 当前焦点 | 证据/报告 |
|---|---|---|---|
| A | Done | 阶段收口完成 | `docs/phase-a/PHASE_A_PROGRESS.md` |
| B | Done | 阶段收口完成 | `docs/phase-b/PHASE_B_PROGRESS.md` |
| C | Done | 阶段收口完成 | `docs/phase-c/PHASE_C_PROGRESS.md` |
| D | Done | 阶段收口完成 | `docs/phase-d/PHASE_D_PROGRESS.md` |
| E | Done | 阶段收口完成 | `docs/phase-e/PHASE_E_PROGRESS.md` |
| F | Done | 阶段收口完成 | `docs/phase-f/PHASE_F_PROGRESS.md` |
| G | Done | 阶段收口完成 | `docs/phase-g/PHASE_G_PROGRESS.md` |
| H | Blocked | 等待静态 PNG 量化主线收口后恢复 | `docs/phase-h/PHASE_H_PROGRESS.md` |

### 附加产品轨道
| 轨道 | 状态 | 当前焦点 | 证据/报告 |
|---|---|---|---|
| Algorithm Replication | In Progress | 静态 PNG reference-first 复查与质量主链收口 | `docs/phase-d/ALGORITHM_REPLICATION_ANALYSIS_V1.md` |

### Algorithm Replication 新规划（Reference-First）
| 子阶段 | 状态 | 参考模块 | 目标 | 当前结论 |
|---|---|---|---|---|
| RF-1 | Done | `pngquant.c` + `attr.rs` | 对齐 `quality/speed` 语义、预算和门禁标尺 | `quality <-> MSE` 已接通 |
| RF-2 | Partially Done | `quant.rs` + `mediancut.rs` + `kmeans.rs` | 对齐 feedback loop、palette search、unused color replacement | 已有骨架，但误差约束收缩仍不稳定 |
| RF-3 | Done | `nearest.rs` | 对齐 VP-tree nearest、likely-index、剪枝逻辑 | 已完成，性能回退已大幅收回 |
| RF-4 | Done | `remap.rs::remap_to_palette` | 对齐 remap 阶段 palette 统计回灌、background/importance 处理 | plain/dithered remap 均已对齐 K-Means finalize + init_int_palette 时序，剩余显式 background 分支（APNG 需要时再加） |
| RF-5 | Partially Done | `remap.rs::dither_map` + `remap_to_palette_floyd` | 对齐 dither map、selective Floyd、background-aware 分支 | core subset、透明区域 plain-fallback 已接入，剩余显式 background 图像分支 |
| RF-6 | Done | `pngquant.c` + `quant.rs` | 对齐 `skip-if-larger` 启发式和 remap 后质量决策 | same-score size-aware 与 `skip-if-larger` 质量/体积联动均已接入 |
| RF-7 | Done | 全链路 | 重跑 quality/perf/stability/release 门禁，形成新基线 | 本地与跨平台复核均已通过 |

### 当前硬阻塞与下一步
1. **18 vs 19 色差异根因已确认：ICC 颜色管理差异**（非算法 bug）
   - demo.png 含 Apple Display ICC profile；我们正确地先转 sRGB 再量化，pngquant 直接用 Display-profile 像素并丢弃 ICC
   - 剥除 ICC 后两者均产生 18 色，算法等价已验证
   - 我们的 ICC→sRGB 路径是正确行为（输出在所有标准显示器上颜色准确）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [okooo5km/pngoptim](https://github.com/okooo5km/pngoptim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
