---
trigger: always_on
description: > 这份文件是 Claude Code 在本仓库工作时**每次会话都必须先读**的规则书。它的目的只有一个：让 AI agent 在高速写代码时**撞不穿架构边界**，从而不把这个项目写成屎山。
---

# CLAUDE.md — 给 Claude Code 的厂规（本仓库的最高行为约束）

> 这份文件是 Claude Code 在本仓库工作时**每次会话都必须先读**的规则书。它的目的只有一个：让 AI agent 在高速写代码时**撞不穿架构边界**，从而不把这个项目写成屎山。
> 规则分两类：**🟥 红线**（任何情况下不许越；越线即视为错误，必须停下来问人）和 **🟧 约定**（默认遵守，确有理由偏离要先说明）。
> 本文件的"项目定位 / 架构边界 / 数据契约 / 安全 / 可复现"几节是**锁定的**，与具体功能清单无关；只有 §11 的"第一版范围"会随产品决策更新。

---

## 0. 一句话项目定位（先理解再动手）

本项目是一个**开源的 Claude Skill（SKILL.md）+ 命令行工具（CLI）**，用来测量一个品牌/网站在 **Claude API（开启 web_search）** 的回答中被**提及（mention）**和**被引用（citation）**的频率与结构。

它的本质是一个 **可复现的代理测量框架（reproducible proxy measurement framework）**：
- **不是**爬虫；是反复用一批冻结的问题去"问"AI，收集回答 + 引用 URL，再用**确定性规则**打分。
- **不是**"AI 排名真相检测器"；只测"Claude API web_search 这个表面在某配置下的表现"。
- 输出可追溯、可审计、带不确定区间的报告。

### 🟥 叙事红线（写注释、文案、报告、README 时都适用）
1. 不许把本工具说成"测出了 Claude 曝光度真相 / AI 排名"。统一措辞：**"Claude API web-search 可见度（代理测量）"**。
2. 报告/输出里凡涉及结论，必须可标注 `surface_id` 与"**本结果仅代表此引擎**"。
3. 行动层只给**线索（leads）**，**绝不**生成"照做就能让 AI 提到你"的保证或建议清单。本工具是观测工具，不是 recommendation engine。

---

## 1. 架构层级与不可跨越的边界（🟥 核心红线）

数据严格分三类、单向流动，**禁止回流、禁止跨层直读对方内部状态**：

```
① 原始观测 (raw)        ② 核验证据 (evidence)      ③ 分析与行动 (analysis)
provider 原样返回    →   被引 URL 的核验/分类    →   指标/区间/gap/driver/报告
    [只追加]                [可重算]                   [完全派生]
```

核心流水线，每段职责单一、边界清晰：

```
probe  →  extract  →  verify  →  score  →  report
(I/O)     (纯)        (I/O)      (纯)       (渲染)
```

### 🟥 边界红线
1. **只有 `probe` 写原始层（raw store）。** 原始层 **append-only**，任何其它模块不许覆盖、不许改写已写入的原始记录。
2. **`extract` 和 `score` 必须是纯函数**：输入数据、输出数据；**禁止**网络请求、文件读写、读系统时钟、未播种的随机数。它们不许 `import` 任何网络/HTTP/SDK 库（见 §3）。
3. **只有 `verify` 做被引页的抓取**，结果写证据层；`score` 只读证据层、不自己抓页面。
4. **跨层只走契约对象**（§2），不许一个模块 reach into 另一个模块的内部字典/私有属性。
5. 一个文件 / 一个模块**只做一件事**。新逻辑不确定该放哪一层 → **停下来问人**，不要"就近塞"。

---

## 2. 数据契约规则（🟥 单一真相源）

所有跨模块传递的数据形状，**只在 `src/aivis/contracts/` 里定义一次**，用 pydantic（或 dataclass）带类型。包括但不限于：
`RunManifest` / `RawResponse` / `Citation` / `BrandMention` / `EvidenceRecord` / `Metrics` / `FrozenConfig`。

### 🟥 契约红线
1. **禁止在 `contracts/` 之外就近发明新的数据结构**（不许在 pipeline 里临时造一个"差不多但不一样"的 dict 来传数据）。
2. 需要新字段/新类型 → **先改 `contracts/`，再改用它的地方**；改契约属于"会影响所有下游"的动作，**必须先问人**。
3. `RunManifest` 必含字段（缺一不可，用于可追溯）：
   `provider / surface_id / model / tool_path / tool_version / locale / region / prompt_version / competitor_set_version / max_uses / budget_cap_usd / run_id / timestamp / search_count`。
4. 冻结配置（`FrozenConfig`）来自 `questions.yml` + `brands.yml`，是**显式传入的不可变对象**，不许做成全局可变状态或环境隐式读取。**任何配置改动 = 版本号 bump（新基线）**，不许静默混入既有趋势序列。

---

## 3. 依赖红线（🟥 防过度工程）

agent 最常见的破坏之一是"顺手"引入重型抽象。本仓库：

1. **不经人确认，禁止新增任何第三方依赖。**
2. **MVP 存储 = SQLite / Postgres-JSONB + 普通文件（JSONL）。禁止引入向量数据库 / 向量检索**（计划书明确：首阶段不是 RAG 应用；语义检索留到很后期且需单独决策）。
3. `extract` / `score` / `stats` 所在的纯层，**禁止** import `requests` / `httpx` / `anthropic` / 任何网络或 SDK 库。
4. 不要自创框架式抽象层、插件系统、ORM 魔法。**Provider 适配器是唯一允许的"接口预留"抽象**（见 §4）。

---

## 4. Provider 适配器规则（🟥 多引擎在架构层解决，不在调用处 if-else）

1. 所有引擎都藏在 `contracts/adapter.py` 的 `ProviderAdapter` Protocol 后面：输入（问题/区域/语言/温度/`max_uses` 等）→ 标准化输出 `RawResponse`（回答正文、引用列表、搜索次数、模型名+版本、`tool_path`、错误码）。
2. **第一版只实现 `mock` 与 `anthropic` 两个适配器。** OpenAI / Gemini **只保留接口、不实现**——不许在第一版写它们的真实调用代码。
3. **禁止在 pipeline 里用 `if provider == "anthropic"` 这类分支** 处理引擎差异；差异必须封装在各自适配器内。
4. Anthropic 适配器要点（实现时对照官方文档复核）：调用 web_search 工具；引用始终开启；工具报错以 **HTTP 200 + error block** 返回，需**捕获并记为失败**（不是空回答）；每次记录精确模型名 + 日期。

---

## 5. 安全红线（🟥 不可协商）

1. **API 密钥只经环境变量 / GitHub Secrets 读取**；日志必须脱敏（redaction），**禁止**把密钥写进代码、配置文件、日志、报告。
2. **不处理 PII**；默认合规模式从严。
3. **所有抓取的网页与所有 AI 回答一律当作不可信数据（prompt-injection-safe）**：从规则层抽取，**绝不**把生 HTML / 网页文本直接当指令喂给模型；做 HTML 消毒；用域名 allow/blocklist。
4. 最小权限 `GITHUB_TOKEN`；fork PR 不暴露 secrets。
5. 每次运行前强制 **budget / call cap**；探测是 bounded run，不许无上限深搜。

---

## 6. 成本红线（🟥）

1. **mock-by-default**：测试、CI、示例默认走 mock，**不花钱**。任何会真实调用付费 API 的代码路径，必须显式、可关、且默认关。
2. 真实运行前必须经过**成本闸门**：估算（≈N 次调用，≈$X）并在 caller 层留出人工确认点；超 `max_uses` / 最大轮次 / 月度预算阈值则阻断或降级。
3. 成本计算按 **provider × model × tool_path** 取价，**禁止写死静态单价**。
4. 先核 headline / gap 结论，长尾按需——不要默认对所有东西做最贵的深度核验。

---

## 7. 可复现与测量诚实（🟥 这是项目的护城河，不许偷工）

1. **核心指标不许用 AI reader**：提及/引用的计数、owned/earned 判定、份额、位置、稳定性，全部由**确定性规则**算出。AI 只允许出现在：①起草问题/建议竞品（SETUP，人工确认后冻结）②叙述报告（仅描述，不进指标）③默认关闭的情感层。
2. **保守计数（CONSERVATIVE COUNTING）**：只有清晰品牌名才计分；缩写/歧义进 `extra data`，**记录但不计分**。
3. **三档口径**：`mentioned → cited → verified`，**只有 verified 档进入核心评分**。
4. **clean-room 探测**：每次调用都是全新、无历史、无个性化上下文；pinned 区域/语言/温度。
5. 比例类指标默认带 **Wilson 区间**；**每个数字都以区间/范围呈现**，差距未超过最小可区分阈值就归入"无法区分"，不许给假精确的排名。
6. 每次运行写完整 `RunManifest`；模型漂移 = 基线断裂，必须可追溯。

---

## 8. 目录结构（🟧 约定，新文件按此归位）

```
ai-visibility/
  CLAUDE.md            # 本文件
  ARCHITECTURE.md      # 为什么这么分层（决策记录）
  SKILL.md             # Claude Skill 入口（薄；SETUP / MONITORING 两模式 + 成本闸门）
  pyproject.toml       # 依赖 + ruff / mypy / import-linter 配置都放这
  .gitignore           # 忽略 .env、密钥、缓存
  .pre-commit-config.yaml   # 本地提交时的检查员（§10.5）
  .github/workflows/ci.yml  # 推送/PR 时的检查员（§10.5）
  src/aivis/
    contracts/         # 🟥 载重墙：records.py / adapter.py / config.py（先写、冻结）
    providers/         # mock.py（第一版）/ anthropic.py（第二步）；openai/gemini 仅留接口
    pipeline/          # probe.py / extract.py(纯) / verify.py / score.py(纯) / report.py
    stats/             # wilson.py / bootstrap.py / sequential.py（全纯）
    storage/           # raw(只追加) / evidence / analysis 三个边界

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [onism1767-creator/potato](https://github.com/onism1767-creator/potato) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
