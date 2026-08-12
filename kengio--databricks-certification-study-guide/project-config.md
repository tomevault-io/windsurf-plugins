---
trigger: always_on
description: Study notes for Databricks certifications: Data Engineer (Associate & Professional), Data Analyst Associate, ML Associate & Professional, GenAI Engineer Associate.
---

# CLAUDE.md

Study notes for Databricks certifications: Data Engineer (Associate & Professional), Data Analyst Associate, ML Associate & Professional, GenAI Engineer Associate.

## Repository Structure

```text
databricks-certification-study-guide/
├── certifications/
│   ├── data-engineer-associate/    # 5 topic folders + resources/
│   ├── data-engineer-professional/ # 8 topic folders + resources/
│   ├── data-analyst-associate/     # 5 topic folders + resources/
│   ├── ml-associate/               # 4 topic folders + resources/
│   ├── ml-professional/            # 4 topic folders + resources/
│   └── genai-engineer-associate/   # 4 topic folders + resources/
├── shared/
│   ├── fundamentals/       # Cross-cert concept files
│   ├── cheat-sheets/       # Quick-reference sheets
│   ├── appendix/           # Glossary, comparisons, errors, troubleshooting
│   ├── code-examples/      # python/ and sql/ (always .md files)
│   └── interview-prep/     # 15 topic files, 108 open-ended questions
├── learning-paths/         # Per-role study paths
├── i18n/                   # Translations index + th/ (Thai in-tree translation)
│   ├── README.md           # Index of all language translations
│   ├── STATUS-TEMPLATE.md  # Template for non-Thai community forks
│   └── th/                 # Thai translation (README.md, glossary.md, STATUS.md)
├── anki/                   # Spaced-repetition Anki decks
│   ├── README.md, format.md, build.py     # Workflow, spec, builder
│   ├── decks/                              # Markdown source (committed)
│   └── build/                              # TSV output (gitignored)
├── practice/               # Static adaptive practice quiz
│   ├── README.md, format.md, build.py     # Workflow, spec, markdown→JSON builder
│   ├── index.html, app.js, styles.css     # Vanilla JS quiz UI (no deps)
│   └── data/<cert>.json                    # Generated question banks (committed for static hosting)
└── images/databricks-ui/   # Screenshots by feature area
```

Each certification folder contains numbered topic folders (`01-topic/`, `02-topic/`, …) with a `README.md` index and individual `.md` topic files. Each also has `resources/` with `exam-tips.md`, `official-links.md`, `practice-questions/`, `mock-exam/`, and `mock-exam-2/`.

## Translations

- **English** is the canonical source. All content edits land in English first.
- **Thai** is the only in-tree translation, at `i18n/th/`. The maintainer reads Thai and reviews Thai PRs.
- **Other languages** use the fork model — community forks register themselves in `i18n/README.md`. We don't merge non-Thai translations into upstream.
- Translation policy is in `TRANSLATING.md`; the Thai glossary is `i18n/th/glossary.md`. Thai translations must conform to the glossary.
- Product names (Delta Lake, Unity Catalog, Lakeflow Jobs, MLflow, etc.) and code blocks stay in English in all translations.
- When English content changes that has a Thai counterpart, mark the Thai file 🔄 in `i18n/th/STATUS.md` until re-synced.

## Content Guidelines

### Content Placement

- **Check `shared/` first** — if content applies to multiple certifications, put it there
- **Reference, don't duplicate** shared content across certifications

### Code Examples

- **Always `.md` files**, never `.py` or `.sql` — store in `shared/code-examples/python/` or `sql/`
- Fenced code blocks with language tags; group related snippets under `##` headings; add YAML frontmatter with `tags`

### File Size

- **Target: 300–600 lines**; hard limit: ~800 lines (~20–25 KB)
- **Exception:** `mock-exam/questions.md` files — do not split
- **Split** when 2+ distinct sub-topics can stand alone → `03-topic-part1.md` + `03-topic-part2.md`:
  1. Same number prefix; append `-part1` / `-part2`
  2. Each part gets own YAML frontmatter and intro
  3. Terminal sections (Use Cases → Official Docs) go in **Part 2 only**; Part 1 ends with forward link
  4. Update topic `README.md` index; delete original; fix all links repo-wide

### Markdown Conventions

- Run markdownlint on every modified file; blank lines before/after headings (MD022)
- Language tags on all code blocks (`sql`, `python`, `scala`)
- **One `#` H1 per file (the title).** Exception: the top-level `README.md` uses a centered `<h1 align="center">` HTML block instead of a markdown `#` to render the banner — this is intentional; do not "fix" it
- **Callout type names are lowercase**: `> [!note]`, `> [!important]`, `> [!warning]`, `> [!tip]`, `> [!success]-`, `> [!abstract]`, `> [!info]`
- **Topic-file preamble callouts** (after the `## Overview`): two standard callouts in this order:
  1. `> [!abstract]` — 2–4 bullets summarising the topic's key concepts (the "what")
  2. `> [!tip] What the Exam Tests` — 2–4 bullets framing what an exam taker should learn from the topic (the "why study this")

  These are *preamble* / orientation callouts. The terminal `## Exam Tips` section (between Common Issues & Errors and Key Takeaways) is for *cram-style* advice consumed right before the exam, and serves a distinct purpose.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kengio/databricks-certification-study-guide](https://github.com/kengio/databricks-certification-study-guide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
