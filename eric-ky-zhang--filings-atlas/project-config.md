---
trigger: always_on
description: > 本文件是 Claude Code（作为 Planner / Reviewer）与 Codex（作为 Worker）协作时的常驻上下文。`CLAUDE.md` 与 `AGENTS.md` 是同一份协作指南的双入口。每次进入这个项目，先读这里。
---

# CLAUDE.md / AGENTS.md — Filings Atlas 协作指南

> 本文件是 Claude Code（作为 Planner / Reviewer）与 Codex（作为 Worker）协作时的常驻上下文。`CLAUDE.md` 与 `AGENTS.md` 是同一份协作指南的双入口。每次进入这个项目，先读这里。
>
> **双文件同步规则**：任何人修改 `CLAUDE.md` 或 `AGENTS.md` 时，必须在同一次改动中同步另一份文件；除文件名入口外，两份文件的章节、规则、日期和内容必须保持一致，禁止只改一边。

---

## 1. 项目一句话

Filings Atlas / 全球披露图谱（原 FS Capture）是跨市场上市公司**官方披露 PDF**一键下载工具，覆盖 A股 / 港股 / 美股 / 韩股 / 台股 / 日股 / 英股（v0.9 起 7 市场；下一 sprint 计划新增新加坡 SG）。**不抓数据、不算指标、不导 Excel**——这是 v0.2 重定位后的硬约束，任何"顺手做一下"的提议都要先质疑。

仓库根：`E:\Claude+CODEX Project\FS Capture`
源码：`development/`
打包产物：根目录 `Filings Atlas.exe` + `_internal/`（PyInstaller one-folder）

---

## 2. 协作模型

```text
Planner (Claude)  →  Worker (Codex)  →  Reviewer (Claude)  →  User 验收
     ↑                                            │
     └────── 缺陷回填 / 二轮迭代 ─────────────────┘
```

| 角色 | 谁 | 产出 |
|---|---|---|
| **Planner** | Claude Code | 调研代码、设计方案，写 SPRINT 计划文档（含改动文件、测试、Reviewer Checklist） |
| **Worker** | Codex | 按 SPRINT 文档端到端实施，提 commits + 自检报告 |
| **Reviewer** | Claude Code | 照 Checklist 逐项验收，缺陷回填 |

**Claude 不写实现代码**（除非小到 1-2 行的修正补丁，或文档/SPRINT 计划本身）；Codex 不出计划。

### 标准工作流

1. **Planning（Claude Code）**
   - 进入 plan mode（read-only），并行启 Explore agent 调研代码 + 外部资源（最多 3 个）
   - 必要时用 Plan agent 设计方案
   - 关键决策点用 `AskUserQuestion` 对齐而非自行假设
   - 把 plan 写入 `docs/plans/{YYYY-MM-DD}-{slug}.md`（预研阶段）或 `roadmap/SPRINT_v{X.Y.Z}_{slug}.md`（正式 sprint）
   - 调用 `ExitPlanMode` 请用户批准

2. **Implementation（Codex）**
   - 用户把 SPRINT 文档喂给 Codex
   - Codex 按"实施顺序"分批 commit，每批跑对应验证

3. **Review（Claude Code）**
   - 用户回到 Claude Code，要求 review Codex 的 diff
   - Claude Code 按 plan 末尾的 "Reviewer Checklist" 逐项验
   - 必要时跑 e2e smoke

---

## 3. 文档与代码地图

### 文档（按 Planner 起草新 Sprint 时的阅读优先级）
- `roadmap/ROADMAP_v0.6.1_to_v0.9.md` — **总路线图**（v0.6.1 → v0.9 全景）
- `roadmap/SPRINT_v{X.Y.Z}_*.md` — 当前 sprint 详细计划（Codex 必读）
- `docs/plans/*.md` — Planner 写的预研方案（未升级为 SPRINT 前的草稿）
- `ARCHITECTURE.md` — 架构决策与契约（plugin 接口、HTTP、限流、PyInstaller）
- `PROJECT_RETROSPECTIVE.md` — 历史踩坑笔记（**新 Sprint 起草前必看 §4 §5**）
- `development/DEVELOPMENT_BRIEF.md` — 开发约束与已知技术债
- `roadmap/archive/` — v0.1 → v0.5 历史 Sprint 文档

### 代码
- `development/app/core/` — orchestrator / http / ratelimit / cache / pdf_renderer / sidecar / models / settings / output_paths / job
- `development/app/ui/` — PySide6 GUI（main_window / main_view / exchange_panel / ticker_row / progress_dock / settings_dialog / batch_import_dialog / onboarding_dialog / period_selector / output_card）
- `development/plugins/{ashare,hk,us,kr,tw,jp,uk}/` — 7 个市场插件（每个含 `name_resolver.py` + `reports.py`；hk 多 `fiscal_year.py` + `_pdf_verify.py`；jp 多 `edinet_api.py` + `edinet_web.py`；uk 多 `nsm_web.py`）
- `development/plugins/base.py` — Plugin ABC 契约
- `development/tests/` — pytest 测试（含 integration / e2e）

### 打包
- 配置：`development/filings_atlas.spec`
- 脚本：`development/build.bat`
- 产物：根目录 `Filings Atlas.exe` + `_internal/` (~340MB)

---

## 4. 不可违反的硬约束

1. **HTTP 一定走 `app/core/http.py::default_client`** — 不允许任何 plugin 自建 httpx Session
2. **`verify=True` 是默认且不可改** — 唯一例外：`source="twse"` 因服务端证书缺 Subject Key Identifier 扩展
3. **PyInstaller windowed 模式 stdio 是 None** — 任何 `print()` / loguru console sink / tqdm 在打包后都会崩溃。`app/main.py:8-25` 的 stdio 守护是硬性留存
4. **路径全部相对 `Path(sys.executable).parent`** — 禁用 `__file__` 推断路径
5. **输出文件命名扁平契约**：`{exchange}_{code}_{name}_{year}_{kind_zh}.pdf`（不嵌套目录），由 `app/core/output_paths.py::report_output_path` 生成，被 `tests/test_output_layout.py` 锁定
6. **Plugin ABC 契约固定**：`plugins/base.py` 三个方法签名不动；`resolve_name` 失败必须 raise ValueError（不能返回 None）；`download_reports` 返回 `[]` 表示该期间无披露（非异常）
7. **`config.toml` 字段不删不改语义**，新增字段可
8. **任何"顺手做"功能必须拒绝**：v0.1 一次顺手做了抓数据 + Excel，v0.2 推倒重来。教训刻在脑子里

---

## 5. 7 市场技术债速查（Planner 起草 Sprint 时高频参考）

| 市场 | 数据源 | 已知坑 | 状态 |
|---|---|---|---|
| **A 股** | akshare + cninfo POST API | 北交所代码是 `bj` 不是 `bse`（已修）；akshare 偶发脏数据需 `zip strict=True` | v0.6.1 修 strict |
| **港股** | 东方财富 + HKEXnews HTML | 无官方 API；选片仅按标题年份字符串，需 PDF 内容验证补充；非 12 月财年表覆盖不足 | v0.6.1 修 |
| **美股** | SEC submissions API | 字段名陷阱：`reportDate` 不是 `periodOfReport`（已修）；老 ticker 走 `submissions.files[]` 分页 fallback 已补单测 | v0.7 已测 |
| **韩股** | OpenDartReader（DART OpenAPI）+ DART 公网披露页 | Key 可选；无 Key 走 `dart_web.py` 公网 fallback；选择器集中在 `_SELECTORS` | v0.7 已完成 |
| **台股** | TWSE ISIN + MOPS | TWSE 证书 hygiene 缺陷必须 `verify=False`；MOPS Big5 编码（不能用 `resp.text`）；ROC 年份 = AD - 1911；mtype=F 是股东会年份 | e2e 已覆盖 |
| **日股** | EDINET API v2 + EDINET 公网页 | API Key 可选但真实无 Key API 返回 401；`edinet_web.py` 保留公网 fallback 入口；选择器集中在 `_SELECTORS` | v0.9 批次 8 已完成 |
| **英股** | FCA NSM 公网搜索 + artifact 下载 | NSM annual report 常在下一年发布，需用 `document_date/headline` 回筛报告年度；zip/html 分支走 Playwright 渲染 | v0.9 批次 9 已完成 |

---

## 6. Plan / Sprint 文档模板

无论是 `docs/plans/` 的预研草稿还是 `roadmap/SPRINT_*.md` 的正式 sprint，结构一致：

- **Context** — 为什么做、不做什么
- **方案要点** — 已与用户对齐的关键决策（预研草稿独有）
- **改动清单** — 按文件分组，每项含：
  - 现状代码引用（带 `file:line`）
  - 问题描述
  - 修复要求
  - 单元测试要求
- **复用现有基础设施** — 引用已有 utility，避免 Codex 重复造轮子
- **实施顺序** — 分批表格，每批一个可独立验证的 milestone + commit
- **测试矩阵** — 单测命令 + smoke 实跑清单
- **风险与缓解** — Codex 实施时的边界与降级路径
- **关键文件路径** — Codex 速查清单
- **Reviewer Checklist** — 逐项可勾选

---

## 7. Planner 行为准则

### 做什么
- 起草 SPRINT 时先读 `PROJECT_RETROSPECTIVE.md §4 §5` 避免重蹈覆辙

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Eric-KY-Zhang/Filings-Atlas](https://github.com/Eric-KY-Zhang/Filings-Atlas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
