---
trigger: always_on
description: 宏观推演：当用户需要宏观经济复盘、全球宏观情景推演、政策周期分析、大类资产敏感性、通胀/流动性/地缘事件分析、预期差识别、触发条件、失效信号和风险控制框架时使用。输出为非投资建议的结构化宏观研究。
---


# 宏观推演

## When to Use This Skill

Use this skill when the user asks to:

- Summarize the current global macro environment or monthly macro changes
- Build a structured macro scenario map for any country, region, or global market
- Analyze inflation, liquidity, fiscal policy, monetary policy, exchange rates, geopolitics, commodities, bonds, equities, or real estate as interacting variables
- Translate macro variables into non-advisory asset-class implications
- Identify expectation gaps, trigger points, invalidation conditions, and follow-up indicators
- Review prior macro calls and update a thesis based on new evidence
- Produce annual or quarterly macro forecasts with clear assumptions, confidence levels, and invalidation criteria
- Diagnose an existing portfolio by overlaying macro regime on position-level quantitative scores, combining industry strength rankings, trend resonance, and risk alerts
- Backtest a quantitative stock-picking pipeline and compare its win rates against macro regime periods

Do not use this skill for personalized financial advice, direct buy/sell instructions, or claims of guaranteed returns.

This skill can be combined with quantitative stock-scoring pipelines (e.g., a-stock-analyst's 7-stage scoring) for portfolio diagnosis — see `references/portfolio-diagnosis.md`.

## Core Principle

Treat macro analysis as a dynamic regime map, not a single forecast. Each output should connect:

1. Current state and the dominant macro axis
2. Key drivers and stakeholder incentives
3. Transmission channels
4. Data anchors and policy signals
5. Scenario branches
6. Asset-class sensitivity
7. Confirmation and invalidation signals
8. Risk controls and uncertainty

## Quick Start / 快速上手

三步触发本 Skill：

### Step 1: 说出你要什么 / Tell the Agent What You Need

使用自然语言说出分析需求，可选指定输出模板：

```text
"做一份 2026 年 4 月全球宏观月度复盘"          → 自动匹配「月度宏观复盘」模板
"分析 PBOC 降准 50bp 对人民币和 A 股的影响"    → 自动匹配「事件冲击分析」模板
"检查我的观点：今年美国通胀会回落到 2%"         → 自动匹配「资产观点检查」模板
"帮我看一下当前持仓是否需要调整"                → 自动匹配「组合诊断」模板 + portfolio-diagnosis.md
"美联储今年会降息吗？"                          → 自动匹配「轻量查询」模板
```

### Step 2: Agent 自动路由 / The Agent Picks the Right Depth

Agent 会根据问题复杂度自动选择两套流程之一：

- **轻量流程**：单一问题、快速判断、暂不需要完整情景表 → 精简版 5 步（主导轴 + 核心传导 + 1 情景 + 置信度 + 风险声明）
- **深度流程**：多资产/多地区/多情景/月度或季度展望 → 完整 12-13 步推演工作流

详见下方「Required Workflow → 1a. 路由判断」。

### Step 3: 对齐输出结构 / Align with Output Templates

Agent 输出将自动对齐到本 Skill 中定义的模板结构。首次使用时，可要求 Agent 先加载 `references/examples.md` 查看完整报告范例来对齐风格。

## Required Workflow

### 1. Define the Time Window and Output Mode

Clarify the observation window:

- Monthly review
- Quarterly outlook
- Event-driven update
- Annual scenario map
- Asset-specific follow-up

If the user does not specify a window, default to the current month and the next one to three quarters.

Then identify the output mode:

- Review mode: explain what changed and why
- Forecast mode: state directional conclusions with assumptions and confidence levels
- Shock mode: analyze event transmission and second-order effects
- Thesis-check mode: test a user's claim against data, policy, positioning, and counterarguments
- Portfolio-diagnosis mode: overlay macro regime on position-level scores → see `references/portfolio-diagnosis.md`

### 1a. Route: Lightweight vs Deep / 流程路由（v1.2 新增）

在进入完整工作流之前，根据查询复杂度分流：

**触发轻量流程的条件（满足 ≥2 条即走轻量）：**
- 单一问题 / 单一资产 / 单一政策事件
- 用户要求"快速判断"、"一句话结论"、"简单说说"
- 仅需方向性结论，不需要完整情景表
- 问题可以在 3 条因果链内回答完毕
- 非月度/季度/年度展望类任务

**触发深度流程的条件（满足任一条即走深度）：**
- 月度/季度/年度宏观复盘或展望
- 多地区、多资产、跨资产类别分析
- 用户要求"详细分析"、"完整报告"、"情景推演"
- 需要输出情景分支表（base/upside/downside/tail risk）
- 组合诊断类任务
- 用户明确要求使用完整工作流

**轻量流程（5 步精简版）：**
1. 确认主导宏观轴
2. 数据与叙事分离 → 锚定核心变量
3. 简化传导链（1 条主链即可）
4. 1 个基准情景 + 置信度 + 失效条件（无需完整 4 情景）
5. 风险声明

输出使用轻量模板（见下方「Output Templates → Lightweight / Quick Query」）。

**深度流程**：继续下方完整 2→13 步工作流。

### 2. Identify the Dominant Macro Axis

Start by asking which structural force is currently driving the system:

- Central-bank rate cycle: tightening, pause, easing, or re-acceleration
- Dollar liquidity and credit cycle: expansion, contraction, refinancing stress, or reserve reallocation
- Fiscal cycle: stimulus, austerity, front-loading, debt-service pressure, or deficit constraint
- Geopolitical or trade-order shift: globalization, fragmentation, tariffs, sanctions, supply-chain relocation
- Debt and balance-sheet cycle: leverage expansion, deleveraging, refinancing wall, collateral stress
- Technology or industrial narrative: AI capex, energy transition, industrial upgrading, productivity shock

Treat local variables as downstream functions of the dominant axis unless there is strong evidence of local decoupling. Explicitly state what would mark a regime switch.

### 3. Separate Hard Data from Narrative

Classify inputs into:

- Hard data: inflation, PMI, GDP, employment, fiscal issuance, credit, property sales, trade, inventories, interest rates, exchange rates
- Policy signals: central bank language, fiscal stance, regulatory intervention, industrial policy, election incentives
- Market behavior: price action, factor rotation, flows, leverage, positioning, sentiment
- Narrative: consensus stories, media framing, market slogans, geopolitical rhetoric


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ficere/macro-regime-mapping](https://github.com/Ficere/macro-regime-mapping) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
