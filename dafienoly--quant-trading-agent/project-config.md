---
trigger: always_on
description: 本文件是本仓库的根级 Agent 指令文件，适用于 PM Agent、Architect Agent、Developer Agent、Test Engineer Agent、Reviewer Agent、Acceptance Agent 以及自动化流水线 Agent。
---

# AGENTS.md

本文件是本仓库的根级 Agent 指令文件，适用于 PM Agent、Architect Agent、Developer Agent、Test Engineer Agent、Reviewer Agent、Acceptance Agent 以及自动化流水线 Agent。

本文件必须保持通用、稳定、长期有效。不要在这里加入临时 sprint 目标、某个阶段的验收修复、一次性股票池、当前版本的细节任务或短期 workaround。此类内容应放在：

```text
docs/features/<feature-id>/
feedback/bugs/
```

当前 canonical 文档布局为“按 feature 聚合”：

```text
docs/features/<feature-id>/
  requirements.md
  architecture.md
  team-plan.md
  phase-<n>-dev-report.md
  phase-<n>-test-report.md
  opencode-lead-review.md
  codex-review-r1.md
  acceptance.md
  user-guide.md
  r3-failure.md
```

`docs/requirements/`、`docs/design/`、`docs/dev_plans/`、`docs/dev_reports/`、`docs/test_reports/`、`docs/review/`、`docs/acceptance/` 仅作为历史文档和兼容读取路径保留；新功能默认写入 `docs/features/<feature-id>/`。

如果某些工具读取的是 `AGENT.md` 而不是 `AGENTS.md`，也必须遵守同一套规则。`AGENTS.md` 是本仓库的 canonical root guide。

---

## 1. 项目定位

本项目是面向 A 股、港股以及未来多市场工作流的量化交易 Agent 系统。

目标产品不是一堆松散的策略脚本，也不是一个让 LLM 随口荐股的玩具，而是一个产品化的投资研究与交易辅助平台，长期包括以下层次：

```text
AgentOps 可观测
  -> 可信市场数据和基本面数据
  -> Provider 契约与 fallback 治理
  -> 可审计 Quant Tool Registry
  -> 受控 Model Gateway 与 Research Agent 层
  -> 决策快照与证据引擎
  -> 风险调整仓位引擎
  -> 策略验证与完整回测报告
  -> 半导体 / AI 链 Risk Sentinel
  -> 战略 Alpha 研究与股票池管理
  -> Paper Trading 与人工确认
  -> Broker Readonly Shadow
  -> 小额人工确认真实交易
```

所有 Agent 必须把以下事项作为一等公民：

```text
交易安全
数据契约
数据质量
证据可追溯
阶段交接文档
人工确认
失败可见
```

系统长期战略 Alpha 研究重点包括：

```text
半导体细分领域
光互联
光计算
存算 / 存算一体
大模型应用
```

这些是研究假设，不是结论。Agent 必须构建能验证、证伪、跟踪、复盘这些假设的系统，而不是默认它们一定正确。

---

## 2. 产品边界

### 2.1 当前有效产品入口

当前 Streamlit Dashboard 仍是有效产品入口。

除非当前已批准的架构文档明确变更，否则 Agent 不得把 Streamlit 标记为：

```text
legacy
deprecated
待删除
```

### 2.2 产品 API 前缀

产品 API 必须使用：

```text
/product/**
```

未来产品 API 命名空间应包括：

```text
/product/agentops/**
/product/market/**
/product/tools/**
/product/model-gateway/**
/product/decisions/**
/product/position-sizing/**
/product/backtests/**
/product/risk-sentinel/**
/product/fundamental/**
/product/alpha/**
/product/paper-trading/**
/product/broker-shadow/**
```

除非当前架构文档明确批准例外，否则不得引入平行业务前缀，例如：

```text
/api/**
```

### 2.3 前端策略

当前仓库尚无稳定 React / TypeScript 前端基线。

涉及 UI 的任务，Architect Agent 必须明确选择：

```text
方案 A：使用当前 Streamlit Dashboard。
方案 B：新增 React + Vite + TypeScript 前端基线。
```

默认规则：

```text
AgentOps、行情健康、回测、Risk Sentinel、Alpha 研究 MVP 优先使用 Streamlit。
只有 React 基座真正完成并具备 CI / E2E 后，才迁移复杂交互页面。
```

---

## 3. 阅读顺序

任何非平凡任务开始前，必须按顺序阅读：

1. `docs/roadmap/MASTER_ROADMAP.md`
2. `docs/process/AGENT_DEVELOPMENT_PIPELINE.md`
3. `docs/process/BRANCH_WORKFLOW.md`
4. 使用 Issue-driven automation 时阅读 `docs/pipeline/AGENT_AUTOMATION_ARCHITECTURE.md`
5. 使用 OpenCode team automation 时阅读 `docs/pipeline/TEAM_PIPELINE_V2.md`
6. 配置本地 Windows / WSL runner 时阅读 `docs/pipeline/LOCAL_AGENT_RUNTIME_SETUP.md`
7. 分支可能自动合并时阅读 `docs/pipeline/AUTO_MERGE_POLICY.md`
8. 接收自动 handoff 时阅读 `docs/pipeline/AGENT_HANDOFF_CONTRACT.md`
9. `docs/policy/SELF_TEST_CHECKLIST.md`
10. 担任 Test Engineer Agent 时阅读 `docs/process/TEST_ENGINEER_WORKFLOW.md`
11. `docs/design/AGENTS.md`
12. `docs/policy/RISK_POLICY.md`
13. `docs/policy/EXECUTION_POLICY.md`
14. 当前任务状态，如存在：`.agent/current_task.yaml`
15. 当前任务 handoff，如存在：`.agent/handoff/<stage>.md`
16. 当前任务需求文档：`docs/features/<feature-id>/requirements.md`
17. 当前任务架构文档：`docs/features/<feature-id>/architecture.md`
18. 当前任务开发指南，如存在：
    - `docs/features/<feature-id>/development-guide.md`
    - 兼容旧路径：`docs/design/YYYY-MM-DD-<feature>-development-guide.md`
19. 当前 handoff 报告，如相关：
    - `docs/features/<feature-id>/`
    - `feedback/bugs/`

历史报告只在与当前功能、回归或 bug fix 直接相关时阅读。

功能级架构和开发指导默认放在 `docs/features/<feature-id>/`；仅在兼容旧流程或迁移历史文档时保留 `docs/design/`。`docs/superpowers/plans/` 仅用于内部 planning scratchpad，不得作为 PM、Architect、Developer、Test Engineer、Reviewer、Acceptance Agent 的 canonical handoff 位置。

---

## 4. 硬安全不变量

以下规则不可协商。违反即为 S0/S1 缺陷。

1. 默认不允许真实自动交易。
2. Risk Agent / Risk Engine 拥有一票否决权。
3. 所有真实订单必须能从 signal -> evidence -> sizing -> risk -> human confirmation -> execution -> reconciliation 全链路追踪。
4. 数据源失败默认阻断 signal 和 real trading 路径。
5. 不得买入创业板、科创板、ST、退市整理股票，除非未来已批准策略明确变更。
6. 任何策略不得绕过股票池过滤。
7. 每个回测必须包含佣金、滑点、印花税、涨跌停和停牌假设。
8. LLM 不得直接决定买入、卖出、最终仓位、risk override 或真实订单。
9. LLM 只能生成结构化研究、解释、标签、排序、摘要、解读，供确定性规则下游使用。
10. 所有 secrets 必须来自环境变量。不得提交 `.env`、key、token、cookie、账户凭据或券商凭据。
11. 任何核心交易逻辑变更必须包含测试。
12. demo、mock、fixture、paper trading、cache、stale、fallback、shadow 数据不得伪装成真实 live trading 能力。
13. 当 `allow_demo=False` 时，产品 live-data 路径不得返回 demo 数据。
14. live 数据不可用时，signal 和 real trading 路径必须 fail closed。
15. `LEVEL_3_AUTO` 不得作为普通用户可随便选择的选项暴露。
16. 产品 live workflow 必须使用产品 API、Provider 契约、Tool Registry 和数据质量门禁，不得绕过。
17. Model fallback 可以降级报告，但不得伪造研究证据、交易结论或信号置信度。
18. 任何触碰 `/product/**`、provider、tool、model gateway、decision snapshot、position sizing、backtest、risk、execution、stock pool 的功能必须包含负向测试。

---

## 5. 战略 Alpha 投资框架

处理 Alpha research、fundamental research、watchlist、scoring、strategy validation 的 Agent 必须使用 Alpha 风格分类器。

### 5.1 Quality Compounder Alpha：长期垄断复利型

长期垄断复利型只看三点：

```text
1. 垄断性 / 护城河
2. 定价权
3. 持续性
```

适用于现金流稳定、竞争优势长期存在、具备长期复利潜力的股权投资候选。

必要评分字段：

```text
alpha_style = quality_compounder
monopoly_score
pricing_power_score
durability_score
quality_compounder_score
quality_alpha_rating
confirm_evidence_ids
disconfirm_evidence_ids
next_review_date

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dafienoly/quant-trading-agent](https://github.com/dafienoly/quant-trading-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
