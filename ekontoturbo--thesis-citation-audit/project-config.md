---
trigger: always_on
description: Use when someone asks to verify thesis citations, audit APA references, check thesis bibliography, weryfikacja cytowań pracy licencjackiej/magisterskiej, lub porównać tekst pracy ze źródłami PDF. Sentence-by-sentence verification of every APA citation in a thesis docx against source PDFs, with parallel sub-agents per subsection, and a consolidated markdown report. Supports Polish and English theses.
---


## What This Skill Does

Performs **sentence-by-sentence verification** of every APA citation in a thesis (.docx) against the actual source PDFs in a folder. Produces a master markdown report listing each citation with status (OK / wrong page / unsupported content / missing source / bibliography mismatch), plus an executive summary of critical problems and prioritized fixes.

This skill replicates a workflow that audited 4 versions of a real bachelor's thesis (~276 citations across 19 subsections) and caught dozens of fabricated page numbers, misattributed authors, source-file mismatches, and bibliography errors that a manual review would miss.

**Side effects:** writes ~20+ markdown files to a working directory + one final report. Dispatches many parallel sub-agents (real token cost). That's why `disable-model-invocation: true` — only the user can fire this.

## When to Use This Skill

- A student / author / reviewer needs every citation in a thesis checked against source PDFs.
- The thesis is in .docx format and sources are in a folder of PDFs (one folder per chapter is typical but not required).
- The user accepts that this takes 15–45 minutes wall-clock and burns tokens proportional to (# of subsections × avg PDFs per subsection).
- Languages: Polish, English, or mixed bilingual works.

## Required Inputs

The skill accepts three arguments (positional, all optional — if missing, ask the user):

1. **$1 — `docx_path`** — full path to the thesis docx, e.g. `C:\Users\name\Desktop\thesis_v10.docx`
2. **$2 — `sources_folder`** — full path to folder containing source PDFs (subfolders OK), e.g. `C:\Users\name\Desktop\Sources V2`
3. **$3 — `output_report_path`** — full path for the final consolidated report, e.g. `C:\Users\name\Desktop\CITATION_AUDIT_REPORT.md` (default: same folder as docx, named `CITATION_AUDIT_<docx_stem>.md`)

If any are missing, ask via `AskUserQuestion` before starting. Confirm all three paths exist before doing work.

## Dependencies

- **Python 3.11+** with `python-docx` (`pip install python-docx`). Check with `python -c "import docx"`.
- **`pdftotext`** (Poppler) on PATH — used by sub-agents to extract PDF text. Check with `which pdftotext` (Linux/macOS/Git Bash) or `where pdftotext` (PowerShell). On Windows install via `choco install poppler` or bundled with Git Bash + MSYS2.
- **Tools used:** `Agent` (general-purpose sub-agents, optionally `model: haiku`), `Bash`, `Write`, `Read`, `Edit`, `Glob`, `Grep`, `TaskCreate`/`TaskUpdate`.

If `python-docx` or `pdftotext` are missing, tell the user how to install and stop.

## Workflow

### Step 1 — Confirm task, plan in TaskCreate

Restate the task back to the user in 1–2 sentences, then create these tasks:

1. Extract docx and detect structure
2. Inventory citations + split into per-subsection files
3. Verify citations (parallel sub-agents, one per subsection)
4. Consolidate reports + executive summary

Mark task 1 `in_progress`. Use the tasks to track progress at each step.

### Step 2 — Extract docx and detect structure

Create a working directory: `<docx_folder>/_citation_audit/`. All intermediate files go there.

Run the extraction script:

```bash
python "C:\Users\erykc\.claude\skills\thesis-citation-audit\scripts\extract_thesis.py" "<docx_path>" "<workdir>"
```

The script writes:
- `<workdir>/thesis_full.txt` — every paragraph numbered `[P0000|Style] text`
- `<workdir>/structure.json` — detected chapters, subsections, bibliography, netografia ranges
- `<workdir>/citations.json` — every detected APA citation with paragraph number and surrounding context
- `<workdir>/bibliography.txt` — extracted bibliography + netografia

Read `structure.json`. Confirm to user: number of chapters, subsections, total citations. If the structure looks wrong (e.g. only 1 subsection detected on a multi-chapter thesis), the heading detection failed — ask the user to confirm the chapter structure manually or to tag headings in the docx as Heading 1/2/3 styles.

Mark task 1 completed, task 2 in_progress.

### Step 3 — Split into per-subsection files

```bash
python "C:\Users\erykc\.claude\skills\thesis-citation-audit\scripts\split_sections.py" "<workdir>"
```

This reads `structure.json` + `citations.json` and writes:
- `<workdir>/sections/section_<id>.txt` — one file per subsection, with paragraphs that have citations
- `<workdir>/sections/_index.json` — list of sections with citation counts + source folder hints

Mark task 2 completed, task 3 in_progress.

### Step 4 — Map sources folder

Glob the sources folder and produce a list of (subsection_hint → folder_path). Two common layouts:

- **Per-chapter subfolders** like `Sources/Chapter 1/1.1/` — common in Polish theses. Match subsection `1.1` to folder `Chapter 1/1.1/`.
- **Flat folder** — all PDFs in one folder. Sub-agents will glob the whole folder for each citation.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ekontoTURBO/thesis-citation-audit](https://github.com/ekontoTURBO/thesis-citation-audit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
