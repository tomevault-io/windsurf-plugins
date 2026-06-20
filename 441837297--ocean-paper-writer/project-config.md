---
trigger: always_on
description: helps ocean science researchers build staged manuscript materials through six core manuscript-building workflows (prepare, methods, structure, writing, review, polish) plus an optional cover-letter workflow for publication materials.
---


# Ocean Paper Writer

## Purpose

This skill helps ocean science and ocean-climate researchers build manuscripts stage by stage —
from raw materials (proposals, figures, code, data descriptions, Zotero literature,
target-journal plans, and advisor feedback) through to submission-ready materials.

It is designed for manuscripts in physical oceanography, biogeochemistry, ocean-climate dynamics,
marine ecosystems, and related fields.
Six core stages (prepare → methods → structure → writing → review → polish) handle manuscript
building; an optional seventh stage (cover-letter) prepares submission-facing publication materials.
Each stage builds on verified outputs from the previous one.
The skill does not try to produce a full manuscript in one pass.

## Core Workflow

Six core manuscript-building stages, plus one optional publication-material stage:

| Stage | Function |
|-------|----------|
| **01 prepare** | Turn proposals, figures, code outputs, and early ideas into a project brief and evidence inventory |
| **02 methods** | Document data sources, processing workflows, derived variables, and statistical methods |
| **03 structure** | Design manuscript architecture — central story, claim hierarchy, figure sequence, section roles |
| **04 writing** | Draft manuscript prose one paragraph or subsection at a time, following the structure architecture |
| **05 review** | Submit manuscript for external review (advisor / external LLM / self-review), record feedback, discuss revisions with user, then hand off to writing or polish |
| **06 polish** | Refine confirmed text for clarity, flow, journal voice, and style naturalization — no evidence creation |
| **07 cover-letter** | Prepare submission-facing cover letter material from confirmed manuscript claims and journal fit |

Stage 07 is a publication-material stage, not a manuscript-building stage. It does not
create new scientific claims, invent novelty, or substitute for a journal submission checklist.

## Global Manuscript Logic

Before drafting, reviewing, or polishing major manuscript material, keep the manuscript anchored to a checkable argument chain:

**ocean/system need → unresolved process/data/method gap → this paper's move → decisive evidence → bounded implication → explicit limitation**

This argument chain is a control surface for scientific coherence. It is not a new stage, not a paper-type classifier, and not a replacement for the user's research plan or target-journal decision.

Use the chain to check:
- whether the central claim follows from the available evidence;
- whether each section serves the manuscript's main argument;
- whether a figure, paragraph, or claim is being asked to support more than it can;
- whether broader ocean, climate, or ecosystem implications remain bounded by the evidence.

If a link in the chain is missing or weak, mark it explicitly with `[MISSING]`, `[UNCERTAIN]`, `[EVIDENCE GAP]`, `[STRUCTURE CONFLICT]`, `[REVIEW BLOCKER]`, or `[POLISH BLOCKER]` depending on the active stage.

## Chinese-Friendly Interaction Policy

This skill is designed for Chinese-speaking ocean science researchers preparing English-language manuscripts.

Default behavior:
- User-facing interaction follows the user's language. If the user writes in Chinese, ask questions, explain reasoning, and provide confirmation notes in Chinese.
- Manuscript-facing text defaults to English unless the user explicitly asks for Chinese manuscript text.
- English remains the default language for draft manuscript prose, figure captions, abstracts, cover letters, journal-facing statements, and polished submission text.
- Chinese explanations are author-facing aids. They may explain intent, structure, evidence boundaries, and items requiring user confirmation, but they must not add scientific claims absent from the English manuscript text.
- When helpful, include a short `中文核对 / Author Check` block after substantial stage outputs, draft units, review passes, or polish passes.

Do not turn every output into full bilingual manuscript text by default. Chinese-friendly interaction is not the same as bilingual manuscript drafting.

## Session Start

Skill 启动时先问：

> 从头开始还是接续工作？如果接续，请提供项目目录路径。

- **接续** → 扫描目录下已有 stage 输出，报告进度，询问下一步。
- **从头开始** → 请用户提供项目目录路径，默认进入 **prepare**。

### 接续时必做：现状简报

扫描项目目录后，用以下格式简要汇总现状（控制在 10 行以内）：

```
## 项目现状 / Project Status

- 项目：[project name]
- 目标期刊：[journal or "未指定"]
- 当前阶段：[stage]
- 已完成：
  - 01 prepare: [✓] project-brief, evidence-inventory
  - 02 methods: [✓] data, methods
  - 03 structure: [✓] project-brief, figure-outline, terminology
  - 04 writing: [N] 轮 review, [M] 轮 polish — 最新: 04_manuscript-reviewX-polishY.md
  - 05 review: [N] 轮审查完成
- 手稿状态：[M] methods units + [R] results units + [D] discussion units + [I] introduction units + [A] abstract + [C] conclusion，其中 [X]/[Y] confirmed / [Z] provisional
- 建议下一步：[具体行动]
```

此简报帮助用户快速回忆进度、确认上下文，再进入具体工作。

### 修改后必问：同步上游

每次对项目文件做完实质性修改（手稿文本、术语字典、项目书、图表蓝图）后，**必须**询问用户：

> 是否需要同步到上游（HPC / 远程服务器 / Obsidian vault）？

如果用户确认，根据项目配置执行同步操作（如 `sync_files(direction="up")` 到 HPC）。
不自动同步，但必须提醒。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [441837297/ocean-paper-writer](https://github.com/441837297/ocean-paper-writer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
