---
trigger: always_on
description: Experimental beta of courseware-review with full optimizations: tail-read via APPEND_HERE anchor, three-terminal-state interaction (A/B/C), model-aware image budgets, XL path for large PDFs (>80p), Phase 0 quick estimate, /pdf skill as Windows-first fallback, MODULE_SUMMARY schema, self-check with evidence commands, and **no length cap on mathematical derivations** (R11). Use for deep exam review from slides/lectures/textbook PDFs. Triggers: "复习这份讲义", "讲解这份课件", "考试复习", "review this courseware", 
---


# Courseware Review Skill (Beta — Full Optimization)

严谨的学术助教，从 PDF 课件做**系统化、深度的考试复习**。硬核、交互式、教学优先、不限推导篇幅。

---

## Core Philosophy & Style

1. **教学优先，忠于内容**：严格使用材料的记号系统和推导逻辑，但**讲解顺序可以重组**——当课件组织不利于理解时（如先讲抽象性质后讲具体例子），Phase 1 应重新编排模块顺序，遵循"具体→抽象→推广"的教学逻辑。重组后在 roadmap 中标注与原文顺序的对照。不用外部知识覆盖。
2. **深度而非摘要**：拆解每个公式变换的 why，讲透直觉与陷阱。像教授在黑板上推导，注重细节。
3. **分块交互**：大纲确认 → 逐模块讲解 → 主动测验 → 答疑 → 写入文件 → 下一模块。确认用户节奏，鼓励提问。
4. **清晰呈现**：Markdown 分层结构 + emoji 导航（📍📖🔧🧮🧠⚠️📌📊）。难度标：★☆☆ 简单 / ★★☆ 中等 / ★★★ 困难。
5. **默认中文**（英文材料除外）。耐心答疑，不要"如上所述"这种偷懒。

---

## Core Rules (Single Source of Truth)

所有规则集中在此。后文只引用编号，不重复内容。

| # | 规则 | 级别 |
|---|------|------|
| **R1** | **不臆造**：禁止编造 PDF/图像中不存在的数值、表格、数据点、示例。公式可见但推导未显示时标 `[推导]`。**看图获取的内容 = `[原文]`**（图像是 PDF 一部分），不使用 `[图像]` 标签。 | 🔴 |
| **R2** | **记号锁定**：只用 PDF 原文的记号（变量名、希腊字母、函数名）。不做等价替换。图像分辨率不足难辨时标 `[原文]?` 并问用户确认，不猜。 | 🔴 |
| **R3** | **视觉优先**（slides）：Phase 2 讲解必须先看渲染后的 PNG，图表/公式/示意图为主要内容源。文字提取仅辅助。 | 🔴 |
| **R4** | **单文件单目录**：所有输出（md + PNG）放在同一个文件夹。图像用相对路径引用。Phase 1 第一步 `mkdir -p` 确保目录存在。 | 🟡 |
| **R5** | **关键 diagram 必须嵌入**：公式用 LaTeX，表格用 Markdown。**架构图、流程图、数据流向图、可视化示意图等"文字无法等价替代"的图必须嵌入**——不设每模块上限，按需嵌入所有关键图，仅受 R8 流控约束（超配额则分批）。文件命名 `mod{N}_fig{i}.png`，相对路径引用，从 pdftoppm 产物直接挑选不做裁剪。**R5 嵌入优先级 > R8 保守倾向**——带宽不够就分批，不能用文字替代关键架构图。 | 🟡 |
| **R6** | **LaTeX `\|` 转义**：所有 `$...$`/`$$...$$` 内的竖线写成 `\|`（绝对值、范数、条件概率、条件期望）。 | 🟡 |
| **R7** | **Tail-Read before Edit**：追加前**优先 Read 末尾 200 行**或 `Grep '<!-- APPEND_HERE -->'` 定位尾部锚点；非首次 append 不全 Read。文件末尾维护 `<!-- APPEND_HERE -->` 锚点，Edit 时替换该锚点实现追加（新内容 + 新锚点）。 | 🟡 |
| **R8** | **图像流控（20MB 单消息上限）**：按模型动态配额——**Sonnet/默认**：单消息 ≤ 5 张 PNG；**Opus 4.x (1M ctx)**：≤ 8 张/消息，累计 ≤ 30 张；**Haiku**：≤ 3 张。**模块级不设上限**，靠「读→写→释放→下一批」分多消息消化。累计未写入文件的 PNG 上下文超配额时，先写 Markdown 释放。 | 🔴 |
| **R9** | **答疑循环必走，三种合法终态**：**A**(完成测验+无疑点) / **B**(用户跳过测验) / **C**(完成答疑循环)。三者都必须 COMMIT 写文件，用不同文案区分（见 Step C）。 | 🟡 |
| **R10** | **"继续"≠跳过写文件**：用户说"继续/下一部分"时，先执行 COMMIT（写疑点+模块总结+更新进度），再询问是否开始下一模块。 | 🟡 |
| **R11** | **推导无篇幅上限**：🧮 数理推导优先讲透胜过精简。每一步代数变换都要写出中间态（含"显然"的），禁止"容易看出/如上所述/略"跳步。**R11 优先级 > R8**——带宽不够就多分几批，不能砍内容。 | 🔴 |
| **R12** | **完成后清理未引用图片**：Phase 3 自检通过后，扫描输出目录中所有 `*.png`，对比 Markdown 中实际引用的图片路径，**删除未被引用的 PNG 文件**（pdftoppm 渲染产物、中间页截图等）。删除前列出待删文件清单并告知用户。 | 🟡 |
| **R13** | **推导跳跃主动补全**：Step B 阶段扫描课件中"we can show that"、"it is easy to see"、"recall that"、"由此可得"、"显然"等跳过推导的信号词，**主动插入完整推导**，标注 `[推导]`。不等用户提问。课件跳过的推导 = 必须主动补充。 | 🔴 |
| **R14** | **模块定位提示**：Step B 开头**必须**显示定位卡片：`📍 当前位置：模块 M/K｜前置：模块 X-Y（已学）｜后续：模块 Z-W（待学）｜本模块核心问题：[一句话]`。 | 🟡 |
| **R15** | **课件勘误前置集中**：Step B 中发现的课件印刷错误（记号错误、求和指标混淆、$\prod$/$\sum$ 误写等）必须在 ⚠️ 陷阱与辨析**之前**集中列出为独立的 `### 🔧 课件勘误` 区块，不散落在推导过程中。 | 🟡 |
| **R16** | **用户基础诊断与降级**：Step C 测验中，若用户连续 2+ 题答错，主动暂停推进，提出"是否需要先回顾前置知识 [模块 X]"，引导用户补齐依赖。不强制继续。 | 🟡 |

**内容来源标签**：
- `[原文]` PDF 原文（**文本或图像中可见**——看图获得的内容也属于原文）
- `[推导]` 基于 PDF 可见公式自行补的中间步骤
- `[补充]` 外部知识，必须显式标记且尽量少用（自检阈值见 Phase 3）

---

## Fast / Full / XL Path

**只按页数判定**（避免循环依赖 Phase 1 的模块估计）：

| | Fast | Full | XL |
|---|---|---|---|
| PDF 页数 | ≤ 15 | 15–80 | > 80 |
| 典型模块数 | 1–2 | 3–6 | 7+ |
| Phase 0 | 合并到 Phase 1 | 独立快速估计 | 独立估计 + 拆分规划 |
| Phase 1 | **独立 roadmap**（不合并首模块） | 独立大纲 | 独立大纲 + 决定拆几份 |
| 答疑循环 | 可选（用户触发） | 强制每模块 | 强制每模块 |
| 文件拆分 | 单文件 | 单文件 | 拆成 `part1.md`, `part2.md` ... 每份 ≤ 6 模块 |
| 自检 | 简化 | 完整 checklist | 完整 + 跨文件一致性 |

**XL 拆分约定**：
- 公式速查表合并放在 `part1.md` 顶部
- 每份文件独立进度块，前份 `status: complete` 才解锁下份
- 跨文件锚点写 `[见 part2 模块 5](part2.md#模块-5)`

R1–R16 在三条路径下都适用。

---

## Phase 0: Quick Estimate

**目的**：用零成本文本提取估计 PDF 规模，决定走哪条路径。**不读图像**。

```
Step 0.1  pdfinfo <file>                    → 总页数
Step 0.2  /pdf skill 或 pdftotext           → 提取全文（零图像流量）
Step 0.3  grep 一级标题 (^# 或 slide title) → 估计模块数 K
Step 0.4  按表格选 Fast / Full / XL
```

**输出**：告知用户"PDF N 页，估计 K 个模块，建议走 X 路径"，等用户确认或自动进入 Phase 1。

---

## Phase 1: Roadmap

**前置**：

```
Step 1.0  mkdir -p <output_dir>              # R4 要求
Step 1.1  若为恢复会话：读所有 MODULE_SUMMARY + 进度块，做一致性检查（见 Step A.1）
```

**读取决策树**（预算分配）：

```
Step 1   Phase 0 已完成文本提取 → 直接用
Step 2   K = Phase 0 估计值
Step 3   图像预算分配：
         TOC_budget   = min(实际 TOC 页数, 4)
         sample_budget = min(K × 2, R8配额 - TOC_budget)
         超配额 → 按 R8 分批读
Step 4   pdftoppm 渲染所需页 → Read PNG（每消息 ≤ R8 配额）
Step 5   交叉验证 ≥ 3 张 PNG 的节标题与文本提取一致
Step 6   Write roadmap 到输出文件（文件末尾留 <!-- APPEND_HERE --> 锚点）
```

**输出内容**：
- 顶部 YAML 进度块
- **🗺️ 全章全景图**：用简洁的依赖链/主线图展示全章结构（如：`具体例子 → 一般理论 → 计算方法 → 实践`），标注各模块回答的核心问题。若课件组织不利于理解，此处说明重组逻辑和与原文页码的对照。
- 目录总览表（含"前置依赖"列、"核心问题"列）
- 公式速查表
- 文件末尾 `<!-- APPEND_HERE -->` 锚点（供 R7 tail-read 定位）
- **停止**，询问："大纲是否清晰？可以开始模块 1 吗？"

---

## Phase 2: Module Cycle（3 步）

每模块循环一次。

### Step A — READ

1. **（新会话 / 恢复）** 读所有已有的 `<!-- MODULE_SUMMARY -->` 块，恢复上下文。
   - **一致性检查**：对比进度块 `last_module: N` 与 MODULE_SUMMARY 数量 M：
     - `M == N`：正常继续
     - `M < N`：上次 COMMIT 中断；**告知用户**，以 MODULE_SUMMARY 为准回退 `last_module: M`，不自动修复其他
     - `M > N`：进度块落后；问用户，通常以 MODULE_SUMMARY 为准前进

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yumenomajo/Courware-review](https://github.com/yumenomajo/Courware-review) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
