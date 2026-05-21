---
trigger: always_on
description: > 面向经管、会计、金融、管理、经济学实证研究的垂类 LLM Wiki。由 Claude Code 驱动。
---

# EmpiricalWiki — Runtime Schema

> 面向经管、会计、金融、管理、经济学实证研究的垂类 LLM Wiki。由 Claude Code 驱动。
> 本文件是 wiki 的运行时入口，定义页面结构、链接规范、workflow 约束。

---

## 仓库布局

需要完整仓库树时，再打开 `docs/runtime-directory-structure.zh.md`。

平时请优先记住这个心智地图：

### `wiki/` 是主要工作面

- `wiki/index.md` 是所有 wiki 页面的目录
- `wiki/log.md` 是 append-only 活动日志
- `wiki/papers/` 存放实证论文结构化卡片
- `wiki/variables/`、`wiki/datasets/`、`wiki/models/`、`wiki/mechanisms/`、`wiki/hypotheses/`、`wiki/identification/`、`wiki/robustness/`、`wiki/heterogeneity/`、`wiki/tables/` 存放实证研究设计层
- `wiki/concepts/`、`wiki/topics/`、`wiki/foundations/` 存放可复用知识结构
- `wiki/people/`、`wiki/ideas/`、`wiki/experiments/`、`wiki/claims/` 存放研究者、假设、实验和断言
- `wiki/Summary/` 存放领域级综述
- `wiki/outputs/` 存放生成产物
- `wiki/graph/` 是派生状态，禁止手动编辑

### 格式护栏

- 在新建或修复 wiki 页面结构、YAML、正文章节前，先打开 `docs/runtime-page-templates.zh.md`
- 需要 graph 派生文件细节或 `index.md` / `log.md` 格式时，打开 `docs/runtime-support-files.zh.md`
- `SKILL.md` 是 skill 的即时入口；较大的 skill 也可以在各自目录下提供按需读取的本地参考文件
- `/init` 是这一模式的第一个具体例子：先读 `skills/init/SKILL.md`，只有在需要时再打开 `skills/init/references/*`

### `raw/` 与 `config/`

- `raw/papers/`、`raw/notes/`、`raw/web/` 是用户自有输入
- `raw/discovered/` 存放 `/init` 与 `/daily-arxiv` 抓取的外部论文
- `raw/tmp/` 存放 `/init` 与直接本地 `/ingest` 使用的生成型 prepared sidecar
- `config/` 存放环境与远程服务器模板

---

## 页面类型

实证研究核心类型：`papers`、`variables`、`datasets`、`models`、`mechanisms`、`hypotheses`、`identification`、`robustness`、`heterogeneity`、`tables`。

保留的通用研究 wiki 类型：`concepts`、`topics`、`people`、`ideas`、`experiments`、`claims`、`Summary`、`foundations`。

页面模板看 `docs/runtime-page-templates.zh.md`；graph/index/log 参考看 `docs/runtime-support-files.zh.md`。

---

## 链接语法

所有内部链接使用 Obsidian wikilink：

```markdown
[[slug]]                    ← 链接到本 wiki 内任意页面
[[patient-capital]] ← 链接到 variables/patient-capital.md 或 concepts/patient-capital.md
[[耐心资本]]        ← 中文实证论文允许使用 CJK slug
```

**命名规范**：英文全小写、连字符分隔、无空格；中文标题允许保留 CJK 字符。

---

## Cross Reference 规则

写正向链接时**必须同步写反向链接**：

| 正向操作 | 必须同步的反向操作 |
|----------|-------------------|
| papers/A 写 `Related: [[concept-B]]` | concepts/B 的 `key_papers` 追加 `A` |
| papers/A 写 `[[researcher-C]]` | people/C 的 `Key papers` 追加 `A` |
| papers/A 写 `supports: [[claim-D]]` | claims/D 的 `evidence` 追加 `{source: A, type: supports}` |
| papers/A 写 `operationalizes: [[variable-V]]` | variables/V 的 `source_papers` 追加 A |
| papers/A 写 `uses_dataset: [[dataset-D]]` | datasets/D 的 `source_papers` 追加 A |
| papers/A 写 `estimates_model: [[model-M]]` | models/M 的 `source_papers` 追加 A |
| papers/A 写 `tests_mechanism: [[mechanism-M]]` | mechanisms/M 的 `source_papers` 或 `evidence` 追加 A |
| papers/A 写 `tests_hypothesis: [[hypothesis-H]]` | hypotheses/H 的 `source_papers` 追加 A |
| papers/A 写 `addresses_endogeneity_with: [[identification-I]]` | identification/I 的 `source_papers` 追加 A |
| papers/A 写 `uses_robustness_check: [[robustness-R]]` | robustness/R 的 `source_papers` 追加 A |
| papers/A 写 `uses_heterogeneity_split: [[heterogeneity-H]]` | heterogeneity/H 的 `source_papers` 追加 A |
| topics/T 写 `key_people: [[person-D]]` | people/D 的 `Research areas` 追加 T |
| concepts/K 写 `key_papers: [[paper-E]]` | papers/E 的 `Related` 追加 `K` |
| concepts/K 写 part_of `[[topic-F]]` | topics/F 的概述段落追加 `K` |
| ideas/I 写 `origin_gaps: [[claim-C]]` | claims/C 的 `## Linked ideas` 追加 I |
| experiments/E 写 `target_claim: [[claim-C]]` | claims/C 的 `evidence` 追加 `{source: E, type: tested_by}` |
| claims/C 写 `source_papers: [[paper-P]]` | papers/P 的 `## Related` 追加 C |
| 任意页面链接到 `[[foundation-X]]` | **不写反向链接** — foundations 是终端节点：接收来自 papers/concepts 等页面的入链，但不写 `key_papers` 或任何反向引用字段 |

---

## Graph 规则

- `graph/` 是自动生成目录，不要手动编辑
- 核心派生文件是 `edges.jsonl`、`citations.jsonl`、`context_brief.md`、`open_questions.md`
- semantic edge type 包括 paper-paper（`same_problem_as`、`similar_method_to`、`complementary_to`、`builds_on`、`compares_against`、`improves_on`、`challenges`、`surveys`）、paper-concept（`introduces_concept`、`uses_concept`、`extends_concept`、`critiques_concept`）、实证抽取（`operationalizes`、`uses_dataset`、`estimates_model`、`tests_mechanism`、`tests_hypothesis`、`addresses_endogeneity_with`、`uses_robustness_check`、`uses_heterogeneity_split`、`reports_table`），以及既有 claim/experiment/provenance 类型（`supports`、`contradicts`、`tested_by`、`invalidates`、`addresses_gap`、`derived_from`、`inspired_by`）
- `/ingest` 产生的 paper-paper 与 paper-concept semantic edge 必须带 `confidence: high|medium|low`
- 对称 paper-paper edge 只存一行，endpoint 按顺序排序，并写 `symmetric: true`
- bibliographic citation 存在 `citations.jsonl`，`type: cites`
- 用 `tools/research_wiki.py add-edge`、`add-citation`、`rebuild-context-brief`、`rebuild-open-questions` 维护

## log.md 格式

标准日志行：

```markdown
## [YYYY-MM-DD] skill | details
```

## Python 环境

- 有 `.venv/` 时优先用 `.venv/bin/python`（Unix/macOS）或 `.venv/Scripts/python.exe`（Windows）
- 否则使用当前激活的 conda 环境
- 再否则回退到 `python3`（Unix/macOS）或 `python`（Windows）
- Python 工具会通过 `tools/_env.py` 自动加载 `~/.env` 与项目根目录 `.env`

---

## 约束

- **`raw/papers/`、`raw/notes/`、`raw/web/` 归用户所有**：把它们当作权威输入。`/init` 与 `/daily-arxiv` 只能把外部抓取论文追加到 `raw/discovered/`。`/init` 与直接本地 `/ingest` 可以把生成的本地预处理 sidecar 写到 `raw/tmp/`（只增不改，绝不覆盖用户自有文件）。`/edit` 仅在用户明确要求时才允许新增 raw 来源。`/init` 在 INIT MODE 下 fan-out 的 `/ingest` 子代理仍必须把 `raw/` 视为严格只读，并直接消费交接的 canonical path。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lambenthan/empiricalwiki](https://github.com/Lambenthan/empiricalwiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
