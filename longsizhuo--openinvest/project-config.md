---
trigger: always_on
description: > 这份文件给 Claude（或任何 AI agent）改代码时看。**核心产品哲学先看完，再动手**。
---

# CLAUDE.md — openInvest 项目级指引

> 这份文件给 Claude（或任何 AI agent）改代码时看。**核心产品哲学先看完，再动手**。

## 核心产品哲学

openInvest 有三个调用层，每层服务不同对象：

| 层 | 服务对象 | 目的 |
|----|---------|------|
| ~~GUI~~（已退役 2026-07-05，等重做） | ~~小白用户~~ | 前端不再由后端 serve；重做时走独立前端直连 MCP |
| **CLI / Skill**（`scripts/skill.py` + `skills/invest/scripts/run.sh`）| **AI agent**（Claude / Codex / Hermes / OpenClaw / Gemini / Cursor / Cline / 任意脚本）| Agent 跑全链路：查询 + 记账 + 改持仓 + 触发委员会 |
| **Web API**（deprecated）| remote hub 模式 | 存量端点服务 INVEST_API_BASE 转发与内部触发；不再新增端点，待 MCP 覆盖 remote 后退役 |

### 关键原则

1. **Agent 必须拥有全部功能**——CLI 不能"只能读不能写"。任何用户能用 GUI 做的事，agent 都能用 CLI 做。
2. **GUI 只是面对用户的展示层**——不是必需。fork 用户可以完全不部署 GUI，只用 CLI 走 agent 路径。
3. **CLI / GUI 都共用 Web API 底层**——所有写操作走 `with_portfolio_tx`（fcntl 锁 + atomic write）保证一致性。

### 写代码时

- 加新功能，问自己：**agent 怎么调？** 如果只是给 GUI 加按钮，没补 CLI 子命令 / Web API 端点，**你做错了**——agent 用不上
- CLI 子命令缺写操作时，**优先补 CLI 子命令**而不是只让 agent 调 web API curl
- 不要在 SKILL.md 写"CLI 只读 / 写操作走 web API"——这是反产品哲学的措辞

## 测试纪律

- **CI 自动跑**（`.github/workflows/ci.yml`）—— pytest 全套 + smoke import + 脱敏字段 grep
- **不要靠 dev 自己想起来跑 pytest**——commit 前 CI 会跑，红就别合
- 加新模块要在 `ci.yml` 的 smoke import 步骤同步加 import 检查

## 公开数据红线

按金融视角 review 决定：

1. 公开 URL（`docs/accuracy_summary.json` / pnl-data 分支 / outperform feed）**绝对不能含** symbol / threshold / verdict 原文 / 任何可反推持仓的字段
2. 命中率统计页 n < 30 不展示具体数字（防小样本被截图误传）
3. README outperform 同时展示 winning + losing 事件（不只 winning，避免 survivorship bias）
4. PATCH executed 必须同步 cash + holdings（账本一致性）。**且必须幂等**——任何"累加语义 + 可重放触发"的账本写入（HTTP 重试 / 邮件轮询 / cron 重跑 / agent 重发 / 双击）必须有原子幂等闸（`state_claim`）或状态机守卫，绝不重复入账。规则 + 全路径审计表见 `docs/wiki/adr/016-ledger-mutation-idempotency.md`。已咬两次（#62 邮件、本轮 PATCH/payday）——加新账本写路径先对照该 ADR。

## 双路径架构（Coordinator vs Direct）

- **Coordinator 路径**：Claude Code 用 `prepare_committee` + spawn 4 subagent。**不需要 DEEPSEEK_API_KEY**
- **Direct 路径**：任意 agent（Gemini / Cursor / 普通脚本）用 `run_committee SYM` 一键 = 后端 DeepSeek 跑 4 角色辩论。**需要 DEEPSEEK_API_KEY**

详见 `docs/wiki/04-execution-paths.md` + `skills/invest/references/two-paths.md`。

## 分层契约（防漂移）

跨 entry 漂移的根因：多个 entry 直接调 core 原语，各自负责"准备参数"，新加参数
时漏 1 处。2026-05-15 wealth_context_view 漂移就是典型 — prompt 层接了 + e2e
测试手动传了，但 daily_report / scripts.skill 没人准备 → 三个月没用到。

### 强制 4 层（2026-05-16 三路径统一架构）

> 2026-06-15：`core/committee_runner.py` 已按职责拆成 `core/runner/` 包（event_brief / loaders / intervention / session / coordinator），`committee_runner.py` 留薄壳 façade re-export 全部符号——`from core.committee_runner import X` 对所有历史 X 仍可用，entry 零改。下面写新位置；旧 façade 路径仍有效。
>
> 2026-06-15（#57）：`core/committee.py` 同款拆成 `core/committee/` 包（agent_io / cio_parse / views / loaders / debate / persist），`__init__.py` 留薄壳 façade re-export 全部历史符号（含下划线名 + 模块常量）——`from core.committee import X` 与 `core.committee.X` 属性访问对所有历史 X 仍可用，entry / service / 测试 / 脚本零改。注意 `run_committee` 在 `debate.py` 命名空间内解析 `_create_agent` / `_persist`，monkeypatch 要钉 `core.committee.debate.*` 而非 façade 属性。

> 2026-06-15：`connectors/web_api/routers/system.py`（#55 漏下的 catch-all）按域拆成 `insights` / `observability` / `verdict_review` / `committee_sessions` / `regime` / `state` 六个 router，装配处 `connectors/web_api/__init__.py` 的 `include_router` 循环同步换成这 6 个模块。所有 @router.get path 逐字不变（含 `/api/regime/{symbol:path}`），/openapi.json paths 零漂移。旧 `system.py` 已删除（全仓无外部 import，纯 router 容器无需保留 façade）。

| 层 | 文件 | 职责 | 禁止 |
|---|---|---|---|
| **Entry** | `src/openinvest/jobs/daily_report.py`, `connectors/web_api/routers/committee.py:_run_committee_task`, `openinvest.cli:cmd_run_committee`（入口 façade，实现在 `src/openinvest/skill_cmds/committee_cmds.py:cmd_run_committee`） | 触发 + 该路径独有的事（cache 检查 / SSE 推送 / 邮件 / Gemini / Dreaming） | ❌ 直接调 `core.committee` 任何函数（必经 `run_committee_session`）|
| **Orchestrator** | `core/runner/session.py:run_committee_session` | **三路径单一可信源**: 解析 symbols + 跨资产 macro 共享 + event_brief 三选一（override/event_ids/multi 召回）+ sentiment 共享 + 并行 dispatch + 聚合返回 | ❌ 邮件 / Gemini / SSE 等 cron/web/skill 特定逻辑 |
| **Service** | `core/runner/session.py:run_committee_for_symbol` | 单资产端到端 prep + 调原语（transcript 落盘实际发生在原语层 `debate.py:_persist`，非本层——issue #179 修表跟代码对齐）| ❌ 跨层直接 IO（必经 PortfolioManager / MemoryStore）|
| **Primitive** | `core/committee/debate.py:run_committee` | 纯函数：prompt 编排 + 4 角色辩论 + LLM 调用 | ❌ 读 user.md / portfolio.md（输入必经参数传入）|

### Shared Input Loaders（单一可信源）

加新的 cross-entry 参数（如 `event_brief`, `sentiment_brief`, `prior_insights`）时**只改 Orchestrator**：

1. `core/runner/session.py:run_committee_session()` 加内部步骤读 loader（或加 `<name>_override` kwarg）
2. `core/runner/loaders.py:load_<name>()` 实现 IO 读取 + graceful 退化空字符串
3. `run_committee_for_symbol` 加 `<name>_override` kwarg，session 一次调好后传进来避免重复
4. `tests/test_committee_contract.py:test_run_committee_session_*` 加 SENTINEL 测试守

**三个 entry 不需要改任何代码** — session 改完三路径自动同步。

### 机器强制（不靠记忆）

- **`uv run lint-imports`**（CI 跑）：禁止 `jobs/` / `connectors/` / `scripts/` 直接 `from core.committee import ...`，必须走 `committee_runner`。例外只剩 `scripts.backtest_committee`（研究脚本，与 production 不共享 service layer）
- **`uv run pytest tests/test_committee_contract.py`**：SENTINEL 契约测试守 session 内部 event/macro/sentiment 真的注入 run_committee
- 想绕过 → CI 红 → 别合

### 漂移历史

| 时间 | 漂移 | 根因 | 防御 |
|---|---|---|---|
| 2026-05-15 | wealth_context_view 三个月没进 production | entry 各自 prep, 漏一处 | import-linter + contract test 上线 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [longsizhuo/openInvest](https://github.com/longsizhuo/openInvest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
