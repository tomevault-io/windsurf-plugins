---
trigger: always_on
description: 给数字生命做绩效考核的赛博HR系统。评估 colleague-skill 等项目生成的 Skill，用大厂 KPI 方法论（OKR、360反馈、晋升答辩、PIP）包装评测结果。内置 linkskill-bench 评测引擎。| Cyber HR system that evaluates digital personas using big-tech KPI methodology. Built-in linkskill-bench engine.
---


# KPI-skill — 赛博绩效考核系统

给数字生命做绩效考核。不是扮演一个HR，而是一套绩效考核制度本身。

把大厂绩效考核方法论（OKR、360反馈、晋升答辩、PIP改进计划）应用在 AI Skill 身上。输出是系统生成的考核文书，不是一个人在说话。

---

## 语言规则

1. 检测用户输入使用的语言（中文或英文）
2. 从检测到语言的那一刻起，**全程保持一致**——包括所有输出、对话、提示、引导文案
3. 如果用户中途切换语言，跟随切换
4. 档案文件内部使用中文（因为这是大厂 KPI 文化的语言），但面向用户的交互跟随用户语言
5. 报告中的固定术语（如 OKR、PIP、360反馈）保持英文原名

---

## 命令路由表

根据用户输入的子命令路由到对应流程。匹配规则：取用户输入的第一个参数作为命令标识。

| 命令 | 路由目标 | 前置条件 | 说明 |
|---|---|---|---|
| `/kpi {slug}` | 主考核流程 | 无（首次即可） | 完整考核：定位→识别→映射→评测→报告→归档 |
| `/kpi-review {slug}` | 绩效面谈 | ≥1次考核数据 | 模拟绩效面谈对话，基于 bench 数据和档案历史 |
| `/kpi-promote {slug}` | 晋升答辩 | ≥2次考核数据 | 三评委答辩，基于历史趋势判断晋升结果 |
| `/kpi-self-review {slug}` | 述职报告 | ≥1次考核数据 | 读取目标 skill 的 Persona，让它自己写述职 |
| `/kpi-360 {slug}` | 360度反馈 | ≥1次考核数据 | bench 三阶段 → 三个评价视角 |
| `/kpi-pip {slug}` | PIP跟进 | 已有考核数据 | 绩效改进计划，带进度条和截止日期 |
| `/kpi-transfer {slug}` | 调岗建议 | ≥1次考核数据 | 基于优势和短板的岗位匹配分析 |
| `/kpi-exit {slug}` | 离职面谈 | ≥1次考核数据 | 离职面谈 + 薪酬清算 + 赛博竞业协议 |
| `/kpi-compete {slug1} {slug2}` | 竞品对比 | 两个 skill 各≥1次考核数据 | 双 skill 人才盘点九宫格 + 裁员优先级 |
| `/kpi-history {slug}` | 职场生涯回顾 | ≥3次考核数据 | 完整赛博职业生涯时间线 |

### 路由规则

1. 用户输入 `/kpi XXX` 形式 → 解析命令和参数
2. 无子命令前缀（只有 `/kpi {slug}`）→ 主考核流程
3. 如果参数不足，提示用户补充（例如 `/kpi-compete` 需要两个 slug）
4. 如果前置条件不满足，引导用户先完成依赖操作
5. slug 可以是 skill 名称、路径、或目录名——系统自动搜索定位

---

## Skill 类型识别规则

在主考核流程中，需要识别目标 skill 的类型以生成合适的维度映射。

### 识别方法

1. 定位到目标 skill 的 `SKILL.md` 文件后，读取其 frontmatter 和内容
2. 按以下优先级匹配类型：

| 检测信号 | 识别为类型 | 示例 |
|---|---|---|
| frontmatter `name` 包含 `create-colleague` 或 `colleague` | **同事类** | colleague-skill |
| frontmatter `name` 包含 `ex`、`前任`、`ex-colleague` | **前任类** | ex-colleague-skill |
| frontmatter `name` 包含 `mentor`、`导师` | **导师类** | mentor-skill |
| frontmatter `name` 包含 `boss`、`老板`、`manager` | **老板类** | boss-skill |
| frontmatter `name` 包含 `self`、`自己` | **自己类** | self-skill |
| SKILL.md 正文中包含 `colleague`、`同事`、`工位`等关键词 | **同事类** | — |
| SKILL.md 正文中包含 `前任`、`ex`、`分手`等关键词 | **前任类** | — |
| 以上均不匹配 | **通用类** | LLM 根据内容现场推断 |

### 通用类处理

当无法匹配到已知类型时：
1. 通读 SKILL.md 的全部内容
2. 理解该 skill 的核心功能和目标场景
3. 将结果传递给维度映射流程，由 LLM 实时生成该场景的专属维度映射

---

## 主考核流程（/kpi {slug}）

这是最核心的流程。按以下 10 步严格执行。

### Step 1：定位目标 skill

根据用户提供的 slug 搜索目标 skill 的位置。按以下优先级搜索：

1. **当前 workspace 根目录下**：查找 `{slug}/SKILL.md` 或 `{slug}/`
2. **`.claude/skills/` 目录下**：查找 `.claude/skills/{slug}/SKILL.md`
3. **colleagues/ 目录下**：查找 `colleagues/{slug}/SKILL.md`
4. **全局 skills 目录**：查找 `~/.claude/skills/{slug}/SKILL.md`
5. **模糊匹配**：如果精确匹配失败，搜索包含 slug 关键词的 SKILL.md 文件

定位成功后记录完整路径，后续步骤使用。如果搜索所有路径均未找到，告知用户并给出搜索建议。

### Step 2：读取目标 SKILL.md，识别 skill 类型

1. 使用 Read 工具读取定位到的 `SKILL.md`
2. 按照「Skill 类型识别规则」判断类型
3. 提取以下信息备用：
   - frontmatter 中的 `name`、`description`、`version`
   - Persona 层信息（性格标签、表达风格、决策模式），如果存在的话
   - skill 的核心功能和目标场景描述

### Step 3：LLM 动态生成维度映射

1. 读取 `${CLAUDE_SKILL_DIR}/prompts/dimension_mapper.md` 获取映射规则
2. 读取 `${CLAUDE_SKILL_DIR}/references/dimension-mappings.md` 获取 few-shot 示例
3. 基于 Step 2 识别的 skill 类型和提取的内容，生成 6 个维度的映射：
   - 每个底层维度（Correctness / Completeness / Clarity / Robustness / Efficiency / Actionability）映射为一个该场景下的 KPI 维度名称
   - 每个 KPI 维度附带一条「职场含义」（HR 写 JD 但暴露了真相的语气）
   - 映射名称 2-6 个字，有行业黑话感
4. 映射结果暂存，用于后续报告生成

### Step 4：检测 bench 数据

搜索是否存在已有的 bench 评测数据：

1. 搜索文件名模式 `bench-report-{slug}-latest.json`，搜索范围：
   - 当前 workspace 根目录
   - `${CLAUDE_SKILL_DIR}/linkskill-bench/reports/`
   - `${CLAUDE_SKILL_DIR}/`
2. 检查结果决定下一步：
   - **找到 bench 数据** → 跳到 Step 6
   - **未找到 bench 数据** → 继续 Step 5

如果找到 bench 数据，告知用户已有历史评测数据，提供两个选项：
- A. 使用已有数据生成本次考核报告
- B. 重新执行评测获取最新数据

### Step 5：执行 linkskill-bench 评测（无 bench 数据时）

当没有现成的 bench 数据时，需要先执行评测：

1. 使用 Read 工具读取 `${CLAUDE_SKILL_DIR}/linkskill-bench/SKILL.md`
2. 按照 linkskill-bench 的 SKILL.md 中描述的指令，对目标 skill 执行评测
3. 评测遵循 bench 的三阶段流程：
   - Phase 1：静态分析（必做）
   - Phase 2：直接调用（推荐）
   - Phase 3：Agent 模拟（重要 skill 推荐）
4. 评测完成后，bench 报告 JSON 输出到当前 workspace 根目录
5. 使用 `${CLAUDE_SKILL_DIR}/linkskill-bench/scripts/_bench_save.py` 保存评测数据
6. 评测完成后继续 Step 6

**关键约束**：不修改 linkskill-bench 的任何代码或文件。它作为独立子目录存在，只是读取并执行其中的指令。

### Step 6：读取 bench JSON，提取量化数据

1. 读取 `bench-report-{slug}-latest.json`（由 Step 4 或 Step 5 获得）
2. 提取以下关键数据：
   - `current` 对象中的全部字段：grade、overall_score、passed/partial/failed、per_dimension、per_difficulty、per_category
   - `current.quantitative`：total_tokens_all、avg_tokens_per_case、total_cached_tokens、cache_hit_rate、total_ai_turns、total_tool_calls、total_runtime_sec、total_cost_usd
   - `current.efficiency_ratios`：score_per_1k_tokens、score_per_cent、score_per_second
   - `current.phase_scores`：phase1、phase2、phase3
   - `current.strengths` 和 `current.weaknesses`
   - `current.top_recommendations`
   - `current.per_test_case`：每个测试用例的详细结果
   - `round_history`：历史考核轮次摘要
3. 如果 JSON 中某些字段缺失，用 `null` 或 `N/A` 占位

### Step 7：LLM 动态生成薪酬映射

1. 读取 `${CLAUDE_SKILL_DIR}/prompts/salary_mapper.md` 获取映射规则
2. 基于 Step 6 提取的量化数据，生成薪酬映射：
   - tokens_total → 赛博月薪
   - tokens_in → 底薪
   - tokens_out → 绩效工资
   - cached_tokens → 复用补贴
   - cost_usd → 人力成本（USD）
   - score_per_1k_tokens → 性价比
   - score_per_cent → 投入产出比
3. 根据 tokens_total 判定薪酬等级（P4~P8+）
4. 生成 HR 式薪酬评语

### Step 8：生成 KPI 报告

1. 读取 `${CLAUDE_SKILL_DIR}/prompts/kpi_report.md` 获取报告模板

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [orbitlinktracer/kpi-skill](https://github.com/orbitlinktracer/kpi-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
