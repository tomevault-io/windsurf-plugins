---
trigger: always_on
description: This file describes the target state that detectors A to I check a project against.
---

# Conventions

This file describes the target state that detectors A to I check a project against.

## Naming

Every name is lowercase snake_case unless a row below says otherwise.

| Element | Convention | Examples |
|---|---|---|
| Folders | lowercase snake_case | `data/`, `topic_models/`, `video_captions/` |
| Markdown files | lowercase snake_case, except owner files | `outline.md`, `meeting_notes.md`, `glossary.md` |
| Scripts | lowercase snake_case | `run_pipeline.py`, `build_report.R`, `fetch_records.sh` |
| Generated data files | stage name, then identifier, then an optional date, joined by underscores | `cleaning_batch_001.jsonl`, `models_batch_001_2026-04-14.parquet` |
| Dated items (correspondence, meeting notes, shared drafts, returned material) | `YYYY-MM-DD_slug` | `2026-09-23_budget_request.md`, `2026-08-14_draft_review.docx` |
| Date tokens | ISO `YYYY-MM-DD` only | `snapshot_2026-03-09.xlsx` |
| Unit folders keyed by the date of their run | `<unit>_YYYY-MM-DD/` | `row_counts_2026-03-09/` |
| Files a tool reads by a fixed name (`README.md`, `CLAUDE.md`, `AGENTS.md`, `CHANGELOG.md`, `LICENSE`) and owner files (defined below) | uppercase name, lowercase extension when there is one | `README.md`, `LICENSE`, `STYLE.md`, `DECISIONS.md` |
| Acronyms | write the acronym in capitals inside a snake_case name when the project's own docs write it in capitals; otherwise lowercase | `IRB_protocol/`, `NSF_budget.xlsx`, `2026-09-23_IRB_amendment.md` |

Names that break the convention, and the names they take:

| Found | Corrected |
|---|---|
| `Topic-Models/` | `topic_models/` |
| `MeetingNotes.md` | `meeting_notes.md` |
| `Run Pipeline.py` | `run_pipeline.py` |
| `readme.md` | `README.md` |
| `decisions.md` (a decision record) | `DECISIONS.md` |
| `Irb_protocol/` (the project's docs write IRB) | `IRB_protocol/` |
| `meeting_09-23-2026.md` | `2026-09-23_meeting.md` |
| `snapshot_20260309.xlsx` | `snapshot_2026-03-09.xlsx` |
| `review_09Mar2026.docx` | `2026-03-09_review.docx` |

An owner file holds one topic for the whole project or for one folder, and other files point to it for that topic. A file is an owner file when a doc points to it as the owner of a topic, in the pointer form given in § Documentation model, or when it is the project's decision record, design, plan, or style guide, such as `STYLE.md`, `DECISIONS.md`, `DESIGN.md`, `PIPELINE.md`, or `PLAN.md`. A row in a file table does not make a file an owner file, and a dated item keeps its `YYYY-MM-DD_slug` name. Any other Markdown file is lowercase, including `notes.md` and `todo.md`.

The date leads the name for dated items and trails it for generated data files and date-keyed unit folders. When a name lacks the year or another part of the date, do not supply it; list the file under Open questions.

Collections of like items sit in one flat folder, with each item keyed by a unique identifier such as a record ID, a DOI, or a date. Do not nest a collection into subfolders by meaning (by theme, by status, by year) when an identifier already tells the items apart; a flat folder can be listed, sorted, and joined against a table without knowing the grouping scheme.

```
records/                     not    records/
|-- rec_0001/                       |-- by_theme/
|-- rec_0002/                       |   `-- pricing/rec_0002/
`-- rec_0003/                       `-- pending/rec_0003/
```

## Layout

The top level of a project holds the role folders below, the root documentation files (`CLAUDE.md`, `README.md`, `CHANGELOG.md`, `LICENSE`, and project-wide owner files), and configuration files, with no wrapper folder (such as `work/` or `project/`) around the role folders. No other file sits directly at the root, whether tracked or gitignored. Each folder below fills one role, and a project uses only the roles it needs.

| Folder | Role |
|---|---|
| `data/` or `source/` | Canonical raw inputs, often gitignored when large |
| `scripts/` or `analysis/` | Code and automation |
| `outputs/` | Generated files, often gitignored when large |
| `references/` | Reference material, source documents, citations |
| `assets/` | Media, images, logos |
| `docs/` or a deliverable folder such as `manuscript/` or `report/` | Written deliverables |
| `emails/` or `correspondence/` | Sent and received correspondence |

Create a folder only when at least two items belong in it. A single file stays in its parent until a second item of the same kind arrives. The role folders in the table above, and the same-layout rules for unit folders and new root folders, take precedence over this rule.

A repeated unit, such as a check, a figure, or a pipeline stage, gets one folder per unit. The rule is that siblings share one inside layout; the names below show a set of checks.

```
checks/
|-- README.md          # index: one row per unit
|-- row_counts/
|   |-- README.md      # the unit report
|   |-- scripts/
|   `-- evidence/
`-- date_ranges/
    |-- README.md
    |-- scripts/
    `-- evidence/
```

A unit whose files support a verdict (a check) keeps them in `evidence/`; a unit whose files are read by a later stage keeps them in `outputs/`. A reader who knows one unit folder can then find the same parts in every other one.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TyrealQ/q-skills](https://github.com/TyrealQ/q-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
