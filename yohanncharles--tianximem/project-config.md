---
trigger: always_on
description: 维护者注释（HTML 注释在注入 context 前会被剥离，不花 token）：
---

<!--
维护者注释（HTML 注释在注入 context 前会被剥离，不花 token）：
  本文件是 Claude Code 每次会话启动时唯一必然加载的文档。
  内容准则：只放「任何动作都要守」的约束 + 路由指针。
    · 模块专属的约束 → 该模块的 CLAUDE.md（Claude 读该目录下文件时按需加载）
    · 论证、依据、实验数据 → PRD 与 docs/（不要搬进这里）
    · 数字 → 只住 eval/reports/（见 docs/README.md 维护约定）
  本文件已按官方建议控制篇幅（<200 行）——超了会降低遵循度。
-->

# TianXiMem — AML 参赛系统

**只提供 `Add` / `Search` 两个 HTTP 端点**（外加一个探活用、不碰下游的 `GET /health`——平台默认探它，S4）；答案生成、评判、聚合全部由 AML 完成。本系统的输出是**按名次排列的证据**，不是答案。

**权威规格**：[`AML Agentic Memory 增强框架 PRD.md`](./AML%20Agentic%20Memory%20增强框架%20PRD.md)。本文件与各模块 `CLAUDE.md` 只做导航与速查；**冲突时一律以 PRD 为准**。

**当前状态**：**实现已开工，不是空脚手架。** 本表是模块级状态的**唯一权威**（别处一律指回这里）。

| 状态 | 模块 |
| --- | --- |
| ✅ **已实现** | [`store/`](src/tianximem/store/)（SQLite 真源 + Qdrant + schema）· [`pairing/`](src/tianximem/pairing/)（**记忆块组合 / 一次 Add = 唯一边界，D24**）· [`embed/`](src/tianximem/embed/)（`Embedder` 协议 + Qwen3-Embedding-8B + **`text-embedding-v4`**（提交口径，2026-09-28）+ 落盘缓存 + 查询侧 instruction 兼容层）· [`common/`](src/tianximem/common/) 的 **`render.py`**（渲染唯一实现）、**`annotate.py`**（**相对时间就地注解**，只改 `content`、不碰索引，**D22**）、**`config.py`**（**全包唯一读环境变量的地方**，③-d）与 **`tokens.py`**（`o200k_base` 计数，§6.4）· [`retrieve/`](src/tianximem/retrieve/)（**策略与参数所有权** + §8 判据 + `dedup_candidates`）· [`rank/`](src/tianximem/rank/) 的 **`reranker.py`**（**远端精排接入**）、**`neighbor.py`**（**扩窗 + 段合并**，§10/§11.2）与 **`packaging.py`**（**段级打包 + 双预算**）· [`service/`](src/tianximem/service/)（HTTP 层 + **Add/Search 端到端编排**，③-c + **`GET /health` 探活**，S4 + **请求原文采集**（诊断旁路，默认关；**S6** 的唯一直接观察口））· [`observability/`](src/tianximem/observability/) 的 **`metrics.py`**（**§14 的出口**：latency + rerank 计数 → run record；⚠ **`embed/` 与 `agent/` 两个发射方还没接**）· [`configs/`](configs/) 的 `default.yaml` + `local.yaml` + **`submit.yaml`**（提交期 profile，2026-09-28 建；**由模型派生的阈值待 Step 5 重标定**） · [`eval/smoke/preflight.py`](eval/smoke/preflight.py)（**契约预检，③-e**）· [`eval/datasets/`](eval/datasets/)（**加载层 + schema 落差预处理**）· [`eval/harness/`](eval/harness/)（**HTTP 驱动（含 `Add` 的有界重试）/ 切批 / **正文形态 `add_shape.py`** / 裁判包装（含 `extra_pipeline.py` 等适配器）/ run record + `api_config.py` + `annotate.py` 注解原型**）· [`eval/reports/schema.py`](eval/reports/schema.py)（**run record 的形状**）· [`tools/check_env.py`](tools/check_env.py)（**环境自检，含 V7 探针**）、[`tools/probe_reranker.py`](tools/probe_reranker.py)（**精排探针**）、[`tools/t2_retrieval_dump.py`](tools/t2_retrieval_dump.py)（**T2 的纯 BM25 转储**）与 [`tools/reindex.py`](tools/reindex.py)（**从 SQLite 全量重建 Qdrant**）、[`tools/diagnose_run.py`](tools/diagnose_run.py)（**跑批诊断：把「低分」与「模型不行」分开**——判读树 + **「证据在不在」那条判据的自校准**）与 [`tools/ab_answer_prompt.py`](tools/ab_answer_prompt.py)（**答案 prompt 的 A/B：只重答拒答题**）· [`eval/experiments/`](eval/experiments/)（**通用 runner `run.py`** + **冻结口径 [`recipes.py`](eval/experiments/recipes.py)** + T1/T2/A3 三个 arm）· [`eval/reports/ledger.md`](eval/reports/ledger.md)（**结果台账**）· [`configs/runs/`](configs/runs/)（**各 arm 的冻结配置**）· [`deploy/`](deploy/) 的 `Dockerfile` + `compose.yaml`（**整栈容器：服务 + Qdrant**）+ `.env.example`（**服务器形态，2026-09-28 起**）——测试在 [`tests/`](tests/) |
| ⬜ **未实现** | [`llm/`](src/tianximem/llm/) · **`eval/` 还没写的**：`datasets/contracts.py`、`experiments/` 的 **A0 一个 arm**（A4 已移出 v1——它要 agent 真的存在，而 v1 不做 agentic，D13）、`baselines/`（**两个参考实现的代码已 Vendor 就位，B1 包装仍未做**——[`refind/`](eval/baselines/refind/) 是 MIT 的 B1，[`invmem-candidate/`](eval/baselines/invmem-candidate/) **只读、不是基线**）、`smoke/` 的 S1/S2/S3 探针与 `quota.py` · 尚未接线的消融开关（**逐项状态以 [`docs/config-reference.md`](docs/config-reference.md) §2 的表为准**——`checker.*` / `agent.*` / `rrf` 未接，`neighbor` 只有 `radius` 可关，而它**默认就是 0**（**D31**，2026-10-01）） |
| ⛔ **v1 不做** | [`agent/`](src/tianximem/agent/)（D13） |

> **⚠ 状态描述是本仓最易过期的东西**：改动状态时，**连同搜一遍所有声称"未实现 / 未开始"的地方**——本表是唯一权威，别处只应指回这里。

**目标**：AML 榜分优于 ReFind 的 44.97。核心判断是**检索不是瓶颈（召回已 96–99%）、选择与排序才是**，所以主线是 Rerank + Context Packaging（§4 / §11）。

---

## 路由：动 X 之前先读 Y

| 要动的东西 | 先读 |
| --- | --- |
| `src/tianximem/<模块>/` 下任何代码 | 该目录的 `CLAUDE.md`（要写什么、边界在哪、本层的坑） |
| 契约层（service、Add/Search 形状） | [`docs/contract.md`](docs/contract.md) |
| 任何阈值 / 权重 / 开关 | [`docs/config-reference.md`](docs/config-reference.md)（**开关的唯一声明处**）+ [`configs/CLAUDE.md`](configs/CLAUDE.md)（**每个键住在 `.env` 还是 yaml**）。**全包只有 [`common/config.py`](src/tianximem/common/config.py) 读环境变量**——有静态测试钉住 |
| 一个"已锁定"的决定 | [`docs/decisions.md`](docs/decisions.md)（逐条 `Dn` + 待决事项——**编号别在这里列范围，它会过期**） |
| 跑对照实验 | [`docs/experiments.md`](docs/experiments.md)（协议）+ [`eval/experiments/CLAUDE.md`](eval/experiments/CLAUDE.md)（怎么跑） |
| 数据集加载 / harness | [`docs/benchmark-data.md`](docs/benchmark-data.md) + [`eval/datasets/CLAUDE.md`](eval/datasets/CLAUDE.md) |
| 读参考实现（InvMem / ReFind 的代码——**代码在哪、什么别学**） | [`docs/reference-implementations.md`](docs/reference-implementations.md) |
| **取回 / 校验 `benchmark_data/`**（新机器、数据缺了、要确认手上的是不是同一份） | [`docs/benchmark-data.md`](docs/benchmark-data.md) 的"出处链" + **"本地修订"**（7 个 pipeline 有一处已声明的偏离：上游那份跑不起来）+ [`tools/fetch_benchmark_data.py`](tools/fetch_benchmark_data.py)。`make fetch-data` / `make data-check` / `make data-patch` |
| 提交周期与截止日 | [`docs/submission.md`](docs/submission.md) §0（**第二期 09-20 已开，材料截止 10-31**） |
| 发 Smoke / Full | [`docs/submission.md`](docs/submission.md)（配额与版本冻结） |
| **部署到服务器**（镜像 / 整栈 / 服务器上要满足什么） | [`deploy/CLAUDE.md`](deploy/CLAUDE.md) **§0** + [`deploy/Dockerfile`](deploy/Dockerfile) 的文件头 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [YohannCharles/TianXiMem](https://github.com/YohannCharles/TianXiMem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
