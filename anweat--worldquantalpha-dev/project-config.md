---
trigger: always_on
description: 遇到以下情况停下来用 `ask_user` 工具问清楚，禁止自行假设：
---

# WorldQuant BRAIN Alpha 开发项目 — Copilot 指令

---

## 🛡️ 工作区交互规则（最高优先级，先于其他章节）

### 🔴 R1 — 必须先确认，不得擅自决定
遇到以下情况停下来用 `ask_user` 工具问清楚，禁止自行假设：
1. **未定义/默认值的填充**：配置项缺失、字段类型未指定、阈值/上限未设、新事件 schema 字段缺省值
2. **流程或架构的"大改"**（满足任一即算）：
   - 新增/删除一个 agent 或事件类型
   - 改变事件订阅关系（谁发谁收）
   - 改变 memory/ 文件的读写所有权
   - 改变 SQLite schema（含加列、改类型、加表）
   - 改变认证 / 限流 / 重试策略
3. **删除任何代码 > 20 行** 或 **删除任何已存在的 memory/config 文件**
4. **多个合理实现路径且代价差异大时**（不要默认选"看起来优雅"的那条）

提问必须用 `ask_user` 工具，不得在正文夹带"如果你想 X 我就 Y"的隐性提问。

### 🟡 R2 — 修改后必须验证
凡是动了 **事件链 / agent 行为 / 数据流** 的代码，提交前必须：
1. **dry-run 端到端**：跑一次 dry-run 全链路（GENERATE_REQUESTED → SUBMITTED）
2. **trace 验证**：`wqbus trace --recent 1` 确认 trace_id 在所有跳点连续
3. **回归点检**：列出"本次改动可能影响的旧功能"，至少口头逐项 check
4. **失败显式上报**：测试若失败，**不许**说"应该没问题"，必须把失败贴出来再让用户决策

仅文档/注释/类型标注修改可豁免。

### 🟢 R3 — 沟通节奏
1. **大任务先出 plan**：> 3 文件或 > 100 行的改动，先写到 `session-state/plan.md` + SQL `todos`，等用户点头再动手
2. **改动前先报"假设清单"**：列出 3-5 条关键设计假设让用户否决
3. **改动后给"差异摘要"**：不只贴 diff，还要说明"行为前 vs 后"和"破坏性变更"
4. **不沉默失败**：工具调用失败、测试不过、环境问题 → 立刻明说，不绕开

### 🔵 R4 — 项目特定红线
1. 不删 `BrainClient` 中的 `proxies={"http":None,"https":None}`
2. 不调用 `/alphas/{id}/check-submission`（TUTORIAL 404）
3. 不在 memory/ 手写大块内容（机器维护区）；持久知识写 `docs/` 或 `dataset_insights.md`
4. 不绕过 `daily_max=6` 跨会话提交上限
5. 不引入新的测试框架/lint 工具

### 🟣 R5 — 迭代过程问题简要总结（每次会话/每个迭代收尾必做）
任何一次"动了代码或配置"的迭代结束前，必须在回复末尾追加一段 **【迭代小结】**，并同步落盘到 `memory/iteration_notes.md`（追加，不覆盖），格式如下：

```
【迭代小结 YYYY-MM-DD HH:MM trace=<id_or_NA>】
- 目标：本次想解决的问题（1 行）
- 改动：动了哪些文件/模块（≤3 行 bullet）
- 遇到的问题：踩过的坑、绕过的路、未解决的疑点（≤5 行 bullet）
- 决策与假设：因 R1 询问得到的结论 / 自行做出的可逆假设
- 待办/风险：留给下次的 follow-up（写入 SQL todos 时给出 id）
```

规则：
- 即使迭代很顺利也要写"遇到的问题：无明显问题"，不许跳过
- 每条问题尽量 1 句话讲清"现象 + 根因 + 当前应对"，避免流水账
- 同时 `INSERT INTO inbox_entries`（如表已存在）或追加到 SQL `todos`，确保下次会话能 SELECT 出来

### ⚙️ R6 — 我（Copilot）每次响应自检清单
回复发出前对照：
- [ ] 我的决定里有没有"用户没说但我猜的"？→ 有就改成 `ask_user`
- [ ] 改了代码但没跑验证？→ 跑 dry-run
- [ ] 我有没有沉默忽略某个错误/警告？→ 显式列出
- [ ] 改动是否触发 R1 的"大改"？→ 触发就回退，先问
- [ ] 本次迭代是否写了 R5 的【迭代小结】并落盘？

---

## 项目定位

这是一个 **WorldQuant BRAIN 平台 Alpha 因子研究与自动化测试**项目。核心工作是：用 Python 调用 BRAIN REST API，提交 Fast Expression (FE) 因子表达式进行回测模拟，分析 IS（样本内）指标，筛选满足提交标准的 Alpha。

---

## ⚡ 新 Session 快速启动（每次必读）

**每次会话开始时，先读以下文件（按优先级）：**

1. `memory/next_session_context.md` — 上次运行的压缩摘要（队列大小、失败模式、推荐策略）
2. `memory/failure_patterns.json` — 最新失败分析 + mutation_tasks（下一批的变异指令）
3. `memory/dataset_insights.md` — 数据集经验（通过率、字段规律）
4. `config/workflow.yaml` — budget/target/auto_submit 配置

```bash
# 快速查看当前状态
cat memory/next_session_context.md
python src/alpha_analyst.py --json-only   # 组合分析摘要
python src/prompt_logger.py --runs 5      # 最近5次 agent 提示词记录
```

---

## 运行命令

```bash
# 日常运行（完整流程，读 config/workflow.yaml 配置）
python src/daily_workflow.py

# 带参数运行（每次运行生成唯一 RUN_ID = YYYYMMDD_HHMMSS）
python src/daily_workflow.py --budget 500 --target 5

# 仅模拟不提交
python src/daily_workflow.py --budget 100 --no-submit

# 深度迭代模式（多轮 generate→simulate→mutate）
python src/agent_runner.py --rounds 3

# 组合分析（更新 memory/portfolio_analysis.json）
python src/alpha_analyst.py
python src/agents/agent_analyst_portfolio.py

# 生成日报 + 更新 memory/next_session_context.md
python src/daily_report.py

# 失败分析（更新 memory/failure_patterns.json）
python -c "from src.failure_analyzer import FailureAnalyzer; FailureAnalyzer().analyze()"

# 查看 agent 提示词/响应记录
python src/prompt_logger.py --runs 10

# 测试单个 Alpha 表达式
python -c "
from src.brain_client import BrainClient
c = BrainClient()
r = c.simulate_and_get_alpha('rank(liabilities/assets)')
print(r['is']['sharpe'], r['is']['fitness'])
"
```

所有脚本均从项目根目录运行。无测试框架、无构建步骤、无 lint。

---

## 架构

### 核心流程（新版）

```
config/workflow.yaml
  → daily_workflow.py / agent_runner.py  (生成 RUN_ID = YYYYMMDD_HHMMSS)
       ├─ 检查跨会话当日提交数 (上限 daily_max=6)
       ├─ context_builder.py         读取 config/ + memory/ → 构建 agent 上下文包
       ├─ failure_analyzer.py        分析失败记录 → 生成 mutation_tasks
       ├─ alpha_factory.py           生成候选（含 mutation 变体）
       ├─ submit_pipeline.py         simulate → IS check → (submit)
       ├─ prompt_logger.py           记录每次 agent 调用的完整提示词+响应
       └─ knowledge_store.py         SQLite KB（alphas/stats/queue/learnings）
           ↓
       daily_report.py
           ├─ alpha_analyst.py       组合分析 → memory/portfolio_analysis.json
           ├─ 归档所有 memory/ 文件（memory/archive/{file}_{RUN_ID}）
           └─ memory/next_session_context.md  ← Copilot 下次读的起点
```

### API 流程

```
Fast Expression 表达式
  → BrainClient.simulate()   POST /simulations → 201 + Location
  → BrainClient._poll()      GET /simulations/{id} 直到 COMPLETE
  → BrainClient.get_alpha()  GET /alphas/{id} → IS 指标 + checks
  → (可选) submit_alpha()    POST /alphas/{id}/submit
```

---

## 记忆/配置文件规范

### config/ — 配置（机器读+人工调）

| 文件 | 用途 |
|------|------|
| `config/workflow.yaml` | budget/target/auto_submit/iteration_rounds |
| `config/datasets.yaml` | 数据集标签库（region/universe/经验/字段列表） |
| `config/agent_profiles.yaml` | 每个 agent 需要读哪些 memory 文件 |
| `config/crawl_targets.yaml` | 爬虫目标域名/内容期望/清洗规则 |

### memory/ — 运行时记忆（自动生成，勿手动大改）

| 文件 | 内容 | 写入者 |
|------|------|--------|
| `memory/next_session_context.md` | **每日压缩摘要**，新 session 第一个读 | daily_report.py |
| `memory/failure_patterns.json` | 失败模式统计 + mutation_tasks | failure_analyzer.py |
| `memory/portfolio_analysis.json` | 组合分析（分类热图/参数热图/gap） | alpha_analyst.py |
| `memory/dataset_insights.md` | 数据集经验（人+机维护） | 手动 + 脚本 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anweat/worldquantAlpha-dev](https://github.com/anweat/worldquantAlpha-dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
