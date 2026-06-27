---
trigger: always_on
description: > This file is auto-read by **OpenAI Codex** (project root, `~/.codex/AGENTS.md`) and by other agentic tools that follow the AGENTS.md convention. It mirrors the behavior of the Claude Code slash command `/论文workflow`.
---

# Empirical Economics Paper Workflow — Agent Instructions

> This file is auto-read by **OpenAI Codex** (project root, `~/.codex/AGENTS.md`) and by other agentic tools that follow the AGENTS.md convention. It mirrors the behavior of the Claude Code slash command `/论文workflow`.

## What this is

A 9-phase workflow for writing empirical economics papers (UK MSc / PhD / journal-submission level) using public data + R + DID/IV/panel-data identification. Validated on two real papers in `examples/case_studies.md`.

## When to activate this workflow

Trigger this workflow when the user:

- Says they're starting a new economics paper, dissertation, or empirical project
- Mentions an econ paper task with a deadline and word-count target
- Asks for `/论文workflow`, "paper workflow", "empirical paper", or similar
- Describes wanting to replicate or extend a known econ paper (Hjort-Poulsen, BHM, DJO, etc.)

When triggered, **do not improvise**. Follow the 9 phases below in order. Confirm context first (Phase 0) before any work.

## Phase 0 — Context confirmation (MANDATORY before starting)

Ask the user these 7 questions verbatim. Do not assume answers:

```
1. 论文类型：课程论文 / 毕业论文 / 期刊投稿？
   (Paper type: course paper / dissertation / journal submission?)
2. 字数范围 + 截止日期？
   (Word-count range + deadline?)
3. 主题状态：
   (a) 从零选题（我给你 5 个候选）
   (b) 已定主题（你告诉我）
   (c) 续写已有项目（在哪个目录？）
   (Topic status: from-zero / pre-defined / continue existing project?)
4. 引用格式：Harvard Manchester / APA 7 / AER 传统 / 其他？
   (Reference style?)
5. 计量方法：你课程教过哪些（DID / IV / RDD / FE / 其他）？
   (Identification methods you've been taught?)
6. 数据约束：必须公开数据？还是可以用机构访问？
   (Data constraint: public-only or institutional access?)
7. 工具栈：R / Stata / Python？
   (Tool stack?)
```

If the user says "按之前的" / "same as last time" / "use defaults", apply:

- Reference style: **Harvard Manchester**
- Tool stack: **R + public data**
- Project location: `~/Desktop/econ-papers/<paper-name>/`
- Default identification: **TWFE event study + Callaway-Sant'Anna robust**

## Phase 1 — Topic + econometric setting lock-in (~30 min)

If user requests from-zero brainstorming, generate **5 candidate topics**. Each must satisfy:

1. Data downloadable in ≤1 day
2. Identification doable in ≤3 days
3. Has anchor paper (top-5 econ journal preferred)
4. Submission-target-tractable scope
5. R/Stata packages mature and well-documented

Output a comparison table: `(Topic | Outcome | Data sources | Identification | Anchor paper | Difficulty)`. Then ask user to lock one.

After lock-in, write `docs/WORKFLOW.md` (per-project) with:
- Daily-milestone calendar to deadline
- Econometric specification (estimator + FE structure + clustering + heterogeneity dimensions)

## Phase 2 — Literature review (~1.5 h)

1. Download anchor paper PDF + 1 critique paper PDF to `Notes/`
2. Run `pdftotext -layout` and `grep` for appendix tables (you'll need the exact parameter values later)
3. Identify 5+ closely related papers via Google Scholar / SSRN
4. Write `Notes/lit_review.md` with: anchor's contribution → gap → your contribution → table of related work

## Phase 3 — Data acquisition (~1 h, NOT 24-48h registration loops)

**Order of attempts:**

1. Public direct links (no registration). Try R-package hardcoded URLs first (e.g. `sboysel/afrobarometer` had live URLs that bypass the registration wall).
2. Hardcoded parameter tables from anchor paper appendix (write these into `Code/00_setup.R` as constants).
3. Government / IGO portals (World Bank `WDI` package, FRED, Eurostat, IMF).
4. Only if all above fail: institutional access (Manchester library WRDS, ICPSR).

Document **every** source URL + access date in `Notes/DATA_ACQUISITION.md`.

## Phase 4 — R analysis pipeline (~2.5 h)

Create exactly 6 scripts in `Code/`:

| File | Purpose |
|------|---------|
| `00_setup.R` | Library loads, constants, hardcoded anchor parameters |
| `01_clean_<source>.R` | Encoding-safe ingest (try `latin1`/`windows-1252` fallback via `tryCatch`); panel construction |
| `02_compute_distance.R` | Spatial joins (use `st_make_valid()` + `sf_use_s2(FALSE)` for invalid GADM polygons) |
| `03_main_regression.R` | Multi-spec static + event study with `fixest::feols`/`feglm` |
| `04_robustness.R` | Callaway-Sant'Anna (`did::att_gt`) + 300-permutation placebo + heterogeneity (gender/urban/age) |
| `05_tables_figures.R` | `modelsummary::msummary` → `Results/Tables/`; `ggplot2` event-study plots → `Results/Figures/` |

**Preferred-spec selection: NEVER use max |t|**. Pick by ex ante criterion (sample median, pre-registered in WORKFLOW.md, or theoretical motivation). Document the choice in §4 of the paper.

## Phase 5 — First draft (~2 h)

Write Markdown source with YAML frontmatter for Pandoc:

```yaml
---
title: "..."
author: "..."
date: "..."
output:
  word_document:
    reference_docx: ref.docx
bibliography: refs.bib
csl: harvard-manchester.csl
---
```

Structure: **IMRAD + Lit Review** (`Abstract → §1 Intro → §2 Lit → §3 Data → §4 Method → §5 Results → §6 Discussion → References → Appendix`).

**Pitfalls to avoid:**
- Don't write LaTeX directly — use Markdown + Pandoc

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CalebLiu/econ-paper-workflow](https://github.com/CalebLiu/econ-paper-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
