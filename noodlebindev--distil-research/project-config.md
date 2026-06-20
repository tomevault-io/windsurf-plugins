---
trigger: always_on
description: Distil a folder of raw research files (Perplexity / ChatGPT deep research / xAI exports) into a clean, compounding knowledge base — operator-first playbook plus organised file structure with archive and quarantine layers. Use this skill whenever the user says "distil research", "distil my research", "/distil research", "clean up my research folder", "organise my research", "build a playbook from my research", or otherwise refers to processing accumulated research files in ~/Research/ or similar.
---


# Distil Research

## What this skill does

The user accumulates research files (Perplexity, ChatGPT deep research, xAI) in topic folders under `~/Research/`. Files pile up. Naming is inconsistent. Information overlaps. Most files are never reopened.

This skill converts that pile into a **compounding knowledge base** by:

1. Scanning the target folder
2. Extracting only the highest-signal **sections** (not files — see below)
3. Producing a clean, standalone **operator-first playbook** the user can re-read and reuse
4. Moving low-signal files to `_archive/`, ambiguous ones to `_quarantine/`
5. Tracking every decision in a manifest so the process is auditable and incremental

The output should feel like an operator's quick-reference written by a domain expert — not a research summary, not a literature review.

## Core principle: sections are the unit, not files

Research exports are mixed quality **inside** the same file. A Perplexity export might have one excellent tactical paragraph buried under three pages of generic best-practice padding.

Treat files as containers of sections. A single source file can end up:
- **Fully kept** — multiple high-signal sections extracted
- **Partially kept** — only 1–2 sections extracted, rest discarded
- **Archived** — even if 1 small idea was salvageable, the file as a whole is low-density
- **Quarantined** — ambiguous; needs human review before discarding

Do not treat "this file has *something* useful" as a reason to keep the whole file in the active set. Extract the signal, then move the file.

## Two modes (auto-detect from target path)

### Topic mode — frequent

Target: a single topic folder like `~/Research/<topic>/` (one folder dedicated to a single research theme). Most invocations.

### Housekeeping mode — occasional

Target: `~/Research/` itself, containing multiple topic subfolders. Different output: cross-topic overlap report + confidence-scored grades + recommendations.

## Topic mode workflow

### Step 1 — Locate the target

If the user says `/distil research` or "distil my research" without a path, ask which topic folder. Don't guess. Acceptable shortcuts: "the X one" → `~/Research/X/`.

Resolve `~` to `/Users/priyeshpatel/`.

### Step 2 — Read existing state

Look for `_distilled/manifest.json`. Present = **incremental** run. Absent = **first run**.

**Where files live (post-iter-3 structure):**

```
~/Research/[topic]/
├── <new file from Perplexity>.md   ← top level is the inbox
├── _sources/                       ← canonical kept files
├── _distilled/                     ← playbook + manifest
├── _archive/                       ← low-signal
└── _quarantine/                    ← ambiguous
```

After every run, the top level should hold only the four `_` folders. Any loose files at the top level are either (a) new files the user just dropped in, or (b) leftovers from a pre-iter-3 run that need migrating into `_sources/`.

For incremental runs:
- Scan the **top level** for new/unprocessed files (anything not inside an `_` folder)
- Scan **`_sources/`** for existing canonical files
- Hash everything found (sha256 of contents)
- Compare against `manifest.json` `files[].hash`
- Identify: new, changed, deleted, unchanged

Process only new and changed files. Unchanged files keep their existing classification.

**Migration note (one-time):** if the manifest exists but there are kept-verdict files at the top level (e.g., from a pre-iter-3 run on an existing folder), treat them as canonical and move them into `_sources/` as part of step 8. Don't reclassify; honour the existing verdict.

### Step 3 — Classify sections (tier filter)

For each new/changed file, parse into sections. For markdown, use headings (`#`, `##`, `###`) as natural boundaries. For PDFs or unstructured text, infer sections from topic shifts or treat the whole document as one section.

**Tier filter** (apply first):

- **Tier 1** — Specific tactics with concrete detail. Numbers, thresholds, named tools or platforms, step-by-step procedures, examples with real values.
- **Tier 2** — Non-obvious frameworks or mental models. Insights that change how you think about the problem class.
- **Tier 3** — Counterintuitive or contrarian findings, with reasoning.
- **Tier 4** (lower priority) — Tool / resource pointers. Only kept if clearly high-quality and non-generic.

**Discard signals** (push toward archive):
- Generic best-practice fluff
- LLM hedging ("it depends", "consider", "various factors")
- Repetition of widely-known facts
- Vague summaries without concrete grounding
- Bulleted lists that restate the section heading
- Content the user already knows because of their stated domain expertise

### Step 3.5 — Large-corpus handling (>20 files in target)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [noodlebindev/distil-research](https://github.com/noodlebindev/distil-research) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
