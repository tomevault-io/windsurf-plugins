---
trigger: always_on
description: Build Python data pipelines that liberate structured data from unstructured and semi-structured sources — government PDFs, FOIA releases, scanned reports, scraped HTML, panel-format spreadsheets — into tidy, documented, reproducible civic datasets. Works at six escalating levels, from a one-shot "just give me the CSV" extraction (L0) up to a fully published, governed, multi-source pipeline (L5) — start low and climb only as far as the task needs. Use whenever the user wants to extract tables or 
---


# Data Liberation

A skill for turning documents — PDFs, scraped pages, panel-format spreadsheets, scanned archives — into tidy, documented, reproducible datasets in the civic-data tradition. It operates at **six increasing levels of complexity**: getting a CSV out of a PDF is one command and one level; a fully published, governed, multi-source pipeline is the same skill, five levels up. Start at the lowest level that satisfies the request and offer to climb — never make a user buy the whole apparatus to get a table out of a PDF.

## The six levels

Each level *adds* to the one before. The levels describe **how far** a given engagement goes; the [six-phase workflow](#the-six-phase-workflow) below describes **how** the work gets done within a project.

| Level | Name | The "___ path" | What it adds | Primary reference |
|---|---|---|---|---|
| **L0** | Extract | "just the data" | The CSV(s). **No scaffold, no project** — read the source, emit the table, done. | [`extract-*` family](references/extract-pdf.md) (pdf · tabular · documents · web · images) |
| **L1** | + Documentation | "make it citable" | A data dictionary, a per-extract `provenance.csv`, a short README / Survey note, a one-line ethics note | [`data-modeling.md`](references/data-modeling.md#data-dictionary), [`context.md`](references/context.md) |
| **L2** | + Pipeline & Audit | "someone can re-run this" | A scaffolded project, `pipeline.py`, a pandera schema contract, `audit.py`, `reconcile.py`, the reject port | [`project-template.md`](references/project-template.md), [`pipeline.md`](references/pipeline.md) |
| **L3** | + Harmonization | "the multi-source path" | A concept catalog / crosswalk (`concepts.py`) with caveats — the cross-source-equivalence contract | [`data-modeling.md`](references/data-modeling.md#concept-catalogs) |
| **L4** | + Standards & Governance | "publishable, responsibly" | DCAT / PROV / DQV / FAIR naming (optional, never a gate); a full governance section | [`context.md`](references/context.md), [`project-template.md`](references/project-template.md#governance) |
| **L5** | + Publishing | "queryable, documented, distributed" | Datasette, a Quarto docs site, Git LFS, DocumentCloud | [`publishing.md`](references/publishing.md) |

**Guardrail: L0 and L1 must NOT run `scripts/scaffold.py`.** Getting to a CSV is frictionless by design; the scaffold is an L2+ commitment. Emitting a stray project skeleton for a one-shot extraction is the single most common way this skill becomes an adoption blocker.

### How to surface levels — infer, confirm, climb

Do not ask the user to read this table. Instead:

1. **Infer** the *lowest* level that satisfies the request, from the phrasing (table below).
2. **State the assumption and offer a redirect**, in one sentence: *"This reads like **L0** — just the data. I'll start there unless you want the reproducible-pipeline path (L2)."*
3. **Execute** that level fully.
4. **Offer the next rung** at the end: *"Done — here's `data.csv`. Want me to go to **L1** and add a data dictionary + provenance so this is citable?"*

When the user names a level, jump straight to it. When signals span levels, infer the highest explicitly named level but **start at the lowest prerequisite and climb**, confirming at each rung.

| User phrasing signals | Infer |
|---|---|
| "get the data out," "give me a CSV," "extract the table from this PDF," "what's in this spreadsheet," "scrape this page into a table" | **L0** |
| "so I can cite it," "document this," "data dictionary," "what do the columns mean," "make this reusable," "where did this come from," "provenance" | **L1** |
| "reproducible," "pipeline," "so someone can re-run it," "audit," "validate against the original," "reconcile," "schema," "add tests" | **L2** |
| "harmonize," "crosswalk," "combine these two sources," "compare across years/agencies," "concept catalog," "make them comparable" | **L3** |
| "standards," "DCAT / FAIR / PROV," "governance," "license," "PII / privacy," "CARE," "is it OK to publish," "FOIA redactions" | **L4** |
| "publish," "Datasette," "queryable," "put it online," "documentation site," "Quarto," "Git LFS / large files," "DocumentCloud," "embed the source docs" | **L5** |
| "scaffold a new civic data project," "set up a data liberation project" | **L2** (the full scaffold is the explicit ask) |

The level menu, if a user asks what the skill can do:

```
Data-liberation has six levels, each adding to the one before:
  L0  Extract            → source to CSV. No setup. "Just the data."
  L1  + Documentation    → data dictionary, provenance, README note. Now citable.
  L2  + Pipeline & Audit → scaffolded, reproducible, validated. "Someone can re-run this."
  L3  + Harmonization    → crosswalks across sources, with caveats. Multi-source.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brianckeegan/data-liberation-skill](https://github.com/brianckeegan/data-liberation-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
