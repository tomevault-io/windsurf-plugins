---
trigger: always_on
description: 当非多模态大模型（DeepSeek V4 等）需要分析硬件仿真波形文件（VCD/EVCD/FST）中的关键信号与时序行为时使用此 Skill。通过编排 Multimodal-Looker → sootty → magick 管道，将波形转换为可视化的 PNG 波形图，由多模态 Agent 读图分析并返回文本结论。适用于 RTL 调试、信号时序检查、握手协议验证、总线行为分析等场景。
---


# Waveform View Skill

使非多模态大模型（如 DeepSeek V4、Claude 3.5 Sonnet 等）能够分析硬件仿真波形文件（waveform file）—— VCD、EVCD、FST 格式——通过调用多模态 Agent 读取由 sootty 生成的可视化波形图（waveform plot），返回结构化的文本分析结论。

**核心原则（Core Principle）：** 波形分析是一条严格的管道（pipeline）：Caller Worker 提出分析需求 → Multimodal-Looker Agent 执行 sootty + magick 生成 PNG → Multimodal-Looker 读图分析 → 返回文本结论给 Caller Worker。任何试图绕过可视化步骤、直接阅读原始 VCD 文本的行为都是无效的。

---

## Non-Negotiables

- **REQUIRED COMPANION AGENT:** 必须搭配 `multimodal-looker` Agent 使用（注意正确拼写：`multimodal-looker`，非 `mulitmodal-looker`）。本 Skill 不替代 `multimodal-looker`，而是定义 Caller Worker 与 Multimodal-Looker 之间的交互协议。
- **NO OTHER SKILLS:** 当此 Skill 适用时，仅使用 `waveform-view-skill` 外加 `multimodal-looker` Agent。不要加载其他 Skill（如 `pdf`、`academic-paper-writer-pro` 等）。所有波形分析的分组、管道编排、失败处理逻辑均已在本文档中定义。
- **sootty MUST be available:** sootty 必须作为 CLI 命令可用（`sootty --help` 应返回正常退出码 0）。不依赖 sootty 的 Python API（当前环境不支持）。
- **SVG→PNG 转换工具 MUST be available:** 优先使用 `magick`（ImageMagick 7）；如果 magick 不可用，fallback 为 `rsvg-convert`。如果两者均不可用，必须报告为阻塞性错误（blocker），不得继续。
- **对于大型波形文件（>100KB）：** 每次 sootty 调用必须指定信号列表（`-w`）和时间窗口（`-s`/`-e` 或 `-l`）。禁止在大型文件上运行无限制的 `sootty [file] -o`。
- **分析结论 MUST be text:** Multimodal-Looker 返回的结论必须是纯文本（text），不得返回图片路径作为最终分析结果。
- **单次 sootty 调用信号数量上限：** `-w` 列表中的信号数量不得超过 12 个（超过将导致波形图不可读）。
- **禁止绕过可视化管道：** 不得尝试直接读取 VCD/FST 原始文本作为分析依据。非多模态模型无法从 VCD 文本中理解波形行为。

## Skill Boundary

- **Allowed companion agent:** `multimodal-looker` 仅此一个。Multimodal-Looker 承担所有图像生成和图像读取任务。
- **Allowed tools:** `sootty`（CLI），`magick`（ImageMagick 7 CLI），`rsvg-convert`（fallback），`pywellen`（可选，仅 FST 格式需要）。
- **Forbidden:** 
  - 将此 Skill 视为独立分析工具（必须搭配 Multimodal-Looker）
  - 使用 sootty 的 Python API 调用方式
  - 绕过 sootty 直接使用其他波形查看器
  - 加载其他 Skill 替代本文档中已定义的管道逻辑
- **此 Skill 管辖的范围：** Caller Worker 与 Multimodal-Looker 之间的交互协议——包括任务描述格式、管道步骤顺序、失败处理策略、效率优化规则。
- **此 Skill 不管辖的范围：** sootty 的具体实现、VCD 文件格式解析、Multimodal-Looker 的图像识别能力本身、RTL 仿真流程。

## Required Outcome

默认的成功交付物为：

- 由 Multimodal-Looker 返回的文本分析结论，内容应包含：
  - 信号名称列表及各自行为描述
  - 关键时序事件（时钟边沿、信号跳变时刻、握手完成时刻等）
  - 异常行为识别（毛刺 glitch、时序违例 timing violation、协议错误 protocol error）
  - 分析问题的直接回答
- 中间产物（SVG 文件、PNG 文件）保存于 `/tmp/` 目录，以 `wave_` 为前缀

以下情况不属于成功完成：

- 仅返回 SVG 或 PNG 路径作为"分析结果"
- 分析结论仅表述"波形看起来正常"而未引用具体信号行为
- 在未检查 `sootty` 和 `magick` 可用性的情况下声称分析完成
- 直接阅读 VCD 文本并从中推断时序行为
- 跳过预扫描（pre-scan）步骤而猜测信号名称

## When to Use

当用户提出以下需求时，触发此 Skill：

**中文触发词（Chinese triggers）：**
- "分析波形" / "查看波形" / "看波形"
- "检查这个 VCD" / "打开这个 evcd" / "分析 FST"
- "检查信号时序" / "查看时序关系"
- "仿真波形分析" / "RTL 调试波形" / "调试这个波形"
- "看一下 clk 和 data 的关系" / "抓一下波形"
- "帮我看看这个信号的波形" / "波形debug"
- "验证握手协议" / "检查 AXI 时序"
- "这个信号有没有 glitch" / "找毛刺"

**English triggers:**
- "analyze waveform" / "debug waveform" / "view waveform"
- "inspect VCD" / "check VCD file" / "open evcd"
- "check signal timing" / "verify handshake protocol"
- "look at the waveform" / "show me the waveform"
- "is there a glitch on this signal" / "find timing violation"
- "compare signals" / "check bus behavior"

**不适用此 Skill 的情况（What NOT to use for）：**
- **逻辑综合（logic synthesis）或形式验证（formal verification）** —— 这些是 EDA 工具链问题，不是波形可视化问题
- **RTL 仿真执行本身** —— 此 Skill 不运行仿真，只分析已生成的波形文件
- **纯文本日志分析** —— 如果用户提供的是仿真日志（log file）而非波形文件，不适用此 Skill
- **电路原理图（schematic）分析** —— 波形图与电路图不同
- **功率分析（power analysis）或时序报告（timing report）** —— 需专门的 EDA 工具
- **单信号简单询问** —— 如用户仅问"VCD 里有哪些信号"，无需走完整管道；仅需预扫描即可

## Portability

- 本 Skill 不假设任何特定的硬件平台或操作系统。
- sootty 命令假定为全局可用的 CLI 命令（通过 `nix`、`pip`、或系统包管理器安装）。
- `magick`（ImageMagick 7）和 `rsvg-convert` 任选其一即可；文档中的命令示例应同时覆盖两者。
- `pywellen`（FST 格式支持）为可选项：仅在需要处理 FST 文件时才需检查。
- 所有中间输出（SVG、PNG）使用 `/tmp/` 目录，不依赖任何特定用户目录结构。
- 不假设 `multimodal-looker` 以外的任何自定义 Agent 存在。
- 示例波形文件路径引用 sootty 官方仓库的 `example/` 目录（https://github.com/Ben1152000/sootty），而非任何本机路径。

## Workflow Model

本 Skill 定义三个角色（roles）：

### 角色 1: Caller Worker（调用方主 Worker）

调用方 Worker（如 Sisyphus、Claude Code 主 Agent）的职责：

- 接收用户的分析需求并识别波形分析场景
- 传递以下信息给 Multimodal-Looker：
  - 波形文件路径（waveform file path）
  - 分析问题描述（analysis question）—— 具体要查什么
  - 建议的信号列表（suggested signal list）—— 如果用户已指定
  - 建议的时间范围（suggested time range）—— 如果用户已指定
- 接收 Multimodal-Looker 返回的文本分析结论
- 将结论以人类可读的格式呈现给用户

### 角色 2: Multimodal-Looker Agent（多模态观察 Agent）

Multimodal-Looker（只读 Agent）是本 Skill 的执行核心，负责：

- 执行波形预扫描（signal pre-scan）获取信号列表
- 选择合适的信号和时间窗口
- 通过 bash 工具执行 `sootty` CLI 命令生成 SVG
- 通过 bash 工具执行 `magick`（或 `rsvg-convert`）将 SVG 转换为 PNG
- 使用其多模态视觉能力读取 PNG 波形图
- 分析波形行为并返回文本结论

Multimodal-Looker 的约束：

- 只读 Agent，不修改任何源文件
- 必须使用 sootty CLI（非 Python API）
- 生成的中间文件放在 `/tmp/` 目录
- 每次 sootty 调用必须携带明确的分析问题

### 角色 3: sootty（CLI 工具）

sootty 是一个命令行工具，位于 https://github.com/Ben1152000/sootty ，负责：

- 读取 VCD、EVCD、FST 格式的波形文件
- 根据指定的信号列表和时间窗口生成 SVG 格式的波形图
- 提供 formula 语言用于精确定义时间窗口和断点

## Dispatch Rules

Caller Worker 与 Multimodal-Looker 之间的任务派发规则：

- **单次派发原则：** Caller Worker 一次派发一个分析任务给 Multimodal-Looker。不要在单次派发中混合多个无关的分析问题。
- **任务描述必须包含：**
  1. 波形文件路径（绝对路径）
  2. 具体的分析问题（不是"看看波形"，而是"检查 clk 和 data 的建立时间是否满足"）
  3. 已知的信号名称（如果有）
  4. 已知的关键时间点（如果有）
- **Multimodal-Looker 可以自行决策：** 信号选择、时间窗口大小、是否需要多轮缩放、使用哪个转换工具。Caller Worker 不应微观管理（micromanage）每一步。
- **并行分析：** 如果分析任务涉及多个信号组之间的时序关系，单个 Multimodal-Looker 内部可以通过分批调用来完成，无需 Caller Worker 多次派发。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [srcres258/waveform-view-skill](https://github.com/srcres258/waveform-view-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
