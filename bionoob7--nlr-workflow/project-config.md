---
trigger: always_on
description: > Claude Code reads this file automatically at the start of every session.
---

# CLAUDE.md — Narrative Literature Review Project Instructions

> Claude Code reads this file automatically at the start of every session.
> Keep it updated as the project evolves.

---

## Project Identity

**Title (working):** Multimodal Digital Twins for Cancer: Integrating Pathology, Radiology, and Multi-Omics AI

**Type:** Narrative literature review (original review article)

> **Important:** This is a *narrative* review, NOT a systematic review. See Writing Style Guidelines for the prose conventions that follow from this distinction.

**Target journals (in priority order):**
- [npj Digital Medicine] TBD — update once confirmed

**Target length:** ~8,000–12,000 words (main text, excluding references)

**Intended audience:** Computational oncology researchers, clinical AI developers

---

## Directory Layout

```
literature-review/
├── CLAUDE.md               ← YOU ARE HERE (always read first)
├── PROMPTS.md              ← reusable prompt snippets for common tasks
├── search/                 ← raw search outputs (CSV/JSON from APIs)
├── screening/              ← ai_screened.csv · borderline_review.csv · final_screened.csv
├── pdfs/                   ← full-text PDFs + pdf_map.csv (PMID/DOI → filename mapping)
├── extractions/            ← one .md file per paper, structured notes
├── synthesis/              ← thematic clusters, gap analysis, tables
├── outline/                ← outline drafts (v1, v2, …)
├── draft/                  ← section drafts (one file per section)
├── references/             ← .bib export from Zotero, final reference list
└── scripts/                ← Python scripts for automation
```

**Rule:** Never write files outside this structure without asking first.

---

## Zotero Integration

- **Import method:** NBIB bulk import (not Zotero API) — run `scripts/export_nbib.py`, then Zotero → File → Import → select `search/included_papers.nbib`
- **Note:** Zotero auto-creates a collection named after the file (`included_papers`); rename it to `DigitalTwins-Cancer-Review` after import

When adding papers to Zotero, always:
1. Import via NBIB to get complete metadata (volume, issue, pages, MeSH terms)
2. After import, rename the auto-created collection to `DigitalTwins-Cancer-Review`
3. Log the Zotero item key in the extraction note

### Citation workflow (drafting → Word)

- **Plugin requirement:** Zotero must have **Better BibTeX** installed. It provides
  stable, reproducible citekeys and exports `references/library.bib`.
- **Drafting:** sections use `[FirstAuthorYear]` placeholders (Phase 5).
- **Phase 7a:** placeholders are replaced with pandoc citation format — `[@citekey]`
  for single, `[@key1; @key2]` for multiple. Output to `manuscript/manuscript_cited.md`
  with YAML metadata `zotero: {client: zotero, csl-style: vancouver}`.
- **Phase 7b:** run pandoc with the BBT lua filter (**Zotero must be open**):
  `pandoc -s --lua-filter=zotero.lua manuscript_cited.md -o manuscript.docx`
  The filter contacts Zotero's JSON-RPC API and embeds live citation fields into the docx.
  `manuscript/zotero.lua` is the filter file (downloaded from retorque.re).
- **Phase 7c (manual):** Open `manuscript.docx` in Word → Zotero toolbar →
  Document Preferences → select citation style → OK. Then click
  "Add/Edit Bibliography" to insert the formatted reference list.
- **Do NOT** use pandoc `--citeproc` or `--bibliography`: they produce static text that
  the Zotero Word plugin cannot edit afterward.

---

## Literature Scope

### In-scope topics
- Cancer digital twins (any organ/cancer type)
- Multimodal AI integrating ≥2 of: pathology (WSI), radiology (CT/MRI/PET), multi-omics (genomics/transcriptomics/proteomics/metabolomics)
- Foundation models applied to oncology multimodal data
- Computational pathology + imaging integration
- Treatment response prediction using multimodal data
- Digital twin frameworks for treatment simulation/personalization

### Out-of-scope
- Single-modality studies (unless methodologically landmark)
- Digital twins in non-cancer disease (unless directly transferable)
- Clinical trial design papers without AI/computational component
- Pure omics studies without imaging integration

### Date range
- Primary focus: 2019–2026
- Landmark earlier papers: include if foundational

---

## Writing Style Guidelines

- **Voice:** Third-person academic, active where possible ("We argue…" only in Discussion/Conclusion)
- **Tone:** Authoritative but balanced; acknowledge limitations and open questions
- **Structure:** Narrative synthesis, not annotated bibliography — papers should serve arguments, not the reverse
- **Citation style:** Use `[FirstAuthorYear]` placeholders during drafting (e.g., `[Chen2023]`); final replacement done in Phase 6
- **Avoid:** Bullet-point lists in the main text; starting consecutive sentences with the same subject; vague openers like "Recently, …" or "In recent years, …"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bionoob7/nlr-workflow](https://github.com/bionoob7/nlr-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-07 -->
