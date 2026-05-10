---
trigger: always_on
description: 本文件是 **AlphaEvo** 仓库的 AI 协作规则与项目规范唯一真源。
---

# AGENTS.md

本文件是 **AlphaEvo** 仓库的 AI 协作规则与项目规范唯一真源。

---

## 1. 项目定位

**AlphaEvo — Self-Evolving Stock Strategy Research Agent** — 自我进化股票策略研究Agent。

核心叙事：AI 像研究员一样，自己提出策略、验证策略、分析失败、持续改进策略。

**不承诺收益神话，强调研究闭环。**

### 核心闭环

```
提出策略假设 → 自动选择样本 → 历史/滚动验证 → 输出多维评估
→ 失败归因 → 自动改写策略 → 再次测试 → 策略进化树
```

---

## 2. 仓库结构

### 架构七层

| 层级 | 目录 | 职责 |
|------|------|------|
| **Data Layer** | `src/alphaevo/data/` | 多数据源接入，统一 MarketSnapshot 输出 |
| **Strategy Layer** | `src/alphaevo/strategy/` | 策略 DSL 定义、解析、序列化、版本管理 |
| **Sampler Layer** | `src/alphaevo/sampler/` | 分层采样（市场环境/风格/策略适用范围） |
| **Backtest Layer** | `src/alphaevo/backtest/` | 指标注册表 + 条件评估 + 回测引擎 + 组合回测 |
| **Evaluator Layer** | `src/alphaevo/evaluator/` | 多维评估（胜率/盈亏比/回撤/环境适应性） |
| **Reflection Layer** | `src/alphaevo/reflection/` | 失败归因 + 修正建议 + 下一版策略生成 |
| **Orchestrator** | `src/alphaevo/orchestrator/` | 端到端流程编排 |

### 辅助模块

| 模块 | 目录 | 职责 |
|------|------|------|
| **Models** | `src/alphaevo/models/` | Pydantic v2 数据模型 |
| **CLI** | `src/alphaevo/cli/` | Typer + Rich 命令行界面 |
| **Config** | `src/alphaevo/core/config.py` | 统一配置管理 (Pydantic Settings) |
| **Leaderboard** | `src/alphaevo/leaderboard/` | 策略排行榜，综合评分 |
| **Utils** | `src/alphaevo/utils/` | 公共工具函数 (utcnow, fmt_pct, safe_div 等) |
| **Alpha Factory** | `src/alphaevo/alpha_factory/` | LLM 驱动因子合成、验证、注册 |
| **Research Log** | `src/alphaevo/research_log/` | 进化过程结构化记录与渲染 |
| **Experience Store** | `src/alphaevo/reflection/experience.py` | 进化经验持久化 (SQLite)，跨策略学习 |
| **Self-Critic** | `src/alphaevo/reflection/critic.py` | 变更质量门控 + 多候选排名 |
| **Meta-Learner** | `src/alphaevo/reflection/meta_learner.py` | 数据驱动的进化策略自适应 |
| **Pattern Library** | `src/alphaevo/strategy/library/` | 跨策略可复用模式提取与注入 |

### 关键入口

| 文件 | 用途 |
|------|------|
| `src/alphaevo/cli/main.py` | CLI 入口 |
| `src/alphaevo/strategy/dsl/parser.py` | 策略 YAML 解析器 |
| `src/alphaevo/data/adapter.py` | 数据适配器抽象接口 + DataManager |
| `src/alphaevo/models/` | Pydantic 数据模型 |
| `strategies/builtin/` | 内置策略模板 (YAML) |
| `tests/unit/` | 单元测试 |
| `src/alphaevo/core/config.py` | 统一配置系统 |
| `src/alphaevo/strategy/dsl/serializer.py` | 策略序列化 (Strategy → YAML) |
| `src/alphaevo/backtest/indicators.py` | 指标注册表 + 条件评估器 |
| `src/alphaevo/backtest/engine.py` | 回测引擎 |
| `src/alphaevo/orchestrator/pipeline.py` | 主流程编排 |

---

## 3. 硬规则

### 代码规范

- 遵循现有目录边界，不跨层放置逻辑。
- 新模块必须有对应的 `__init__.py` 和类型导出。
- 所有公共接口使用 Pydantic v2 模型，不用裸 dict。
- 策略定义必须同时有**人类可读描述**和**可执行 DSL**。
- 不写死密钥、路径、模型名、端口。所有可配置项通过 `AppConfig` 或环境变量读取。
- 复用 daily_stock_analysis 的数据能力时通过 adapter 层隔离，不直接依赖其内部实现。
- 时间相关默认值统一使用 `datetime.now(timezone.utc)` (UTC)。

### Git 规范

- 未经确认，不执行 `git commit`、`git tag`、`git push`。
- Commit message 使用 Conventional Commits 格式：`feat:`, `fix:`, `docs:`, `refactor:`, `test:`, `chore:`。
- PR 必须包含：改了什么、为什么、验证情况、风险点。

### 安全规范

- 不承诺投资收益，README 和所有输出必须包含免责声明。
- 不存储用户真实交易数据，仅处理公开市场数据。
- API Key 等敏感信息只通过环境变量传入，不落配置文件。

---

## 4. 技术栈

### 核心依赖（pip install alphaevo 即装）

| 技术 | 用途 | 版本要求 |
|------|------|----------|
| Python | 运行时 | >= 3.10 |
| Pydantic | 数据模型 & 校验 | >= 2.0 |
| Typer | CLI 框架 | >= 0.9 |
| Rich | CLI 美化 | >= 13.0 |
| pandas | 数据处理 | >= 2.0 |
| PyYAML | 策略 DSL 解析/序列化 | >= 6.0 |
| tenacity | 重试逻辑 | >= 8.2 |
| SQLite | 策略版本 & 评估存储 | 内置 |

### 可选依赖（按需安装）

| 安装方式 | 包含 | 用途 |
|----------|------|------|
| `pip install alphaevo[llm]` | litellm >= 1.40 | LLM 驱动的策略生成/反思（进化功能必需） |
| `pip install alphaevo[data-yfinance]` | yfinance >= 0.2 | 美股/港股/A股(有限)数据源 |
| `pip install alphaevo[data-akshare]` | akshare >= 1.12 | A 股全量数据源 |
| `pip install alphaevo[data-full]` | yfinance + akshare + efinance + tushare + baostock | 全部数据源 |
| `pip install alphaevo[tui]` | textual >= 0.40 | TUI 升级 |
| `pip install alphaevo[charts]` | plotext >= 5.0, matplotlib >= 3.7 | 终端/图形图表 |
| `pip install alphaevo[dev]` | pytest, ruff, mypy, pre-commit | 开发工具 |

> **设计决策**: `litellm` 从核心依赖移至 optional。纯回测场景（无 LLM 进化）不需要 litellm 及其 200+ MB 子依赖。`alphaevo run` 和 `alphaevo leaderboard` 仅需核心依赖即可运行；`alphaevo evolve` 和 `alphaevo strategy create` 需要 `[llm]`。

---

## 5. 配置管理

### 配置优先级（高 → 低）

1. CLI 参数（`--model`, `--adapter` 等）
2. 环境变量（`ALPHAEVO_LLM_MODEL`, `ALPHAEVO_DATA_ADAPTER` 等）
3. 项目配置文件（`.alphaevo/config.yaml`，在项目根目录）
4. 用户配置文件（`~/.alphaevo/config.yaml`）
5. 内置默认值

### 关键配置项

| 配置项 | 环境变量 | 默认值 | 说明 |
|--------|----------|--------|------|
| `llm.model` | `ALPHAEVO_LLM_MODEL` | `gemini/gemini-2.0-flash` | 策略生成/反思用的 LLM |
| `llm.reflect_model` | `ALPHAEVO_LLM_REFLECT_MODEL` | 空（复用 llm.model） | 反思专用模型 |
| `llm.api_key` | `ALPHAEVO_API_KEY` | — | LLM API Key（**仅通过环境变量**） |
| `data.adapter` | `ALPHAEVO_DATA_ADAPTER` | `yfinance` | 数据源：yfinance / akshare / dsa |
| `data.cache_dir` | `ALPHAEVO_CACHE_DIR` | `~/.alphaevo/cache/` | 数据缓存目录 |
| `db.path` | `ALPHAEVO_DB_PATH` | `~/.alphaevo/alphaevo.db` | SQLite 数据库路径 |
| `backtest.slippage` | — | `0.001` | 默认滑点 |
| `backtest.commission` | — | `0.0003` | 默认手续费率 |
| `backtest.fill_policy` | `ALPHAEVO_BACKTEST_FILL_POLICY` | `conservative` | 同 K 线止损/止盈冲突处理：conservative / optimistic / close_first |
| `backtest.walk_forward_folds` | `ALPHAEVO_BACKTEST_WALK_FORWARD_FOLDS` | `3` | Walk-Forward 默认折数 |
| `backtest.walk_forward_train_pct` | `ALPHAEVO_BACKTEST_WALK_FORWARD_TRAIN_PCT` | `0.7` | Walk-Forward 训练集比例 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZhuLinsen/alphaevo](https://github.com/ZhuLinsen/alphaevo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
