---
trigger: always_on
description: > Version: 1.1.0 | Status: Stable | Last updated: 2026-04-06
---

# AGENTS.md — LLM Knowledge Base Schema
> Version: 1.1.0 | Status: Stable | Last updated: 2026-04-06

This file is the **single source of truth** for any LLM agent operating on this knowledge base. It defines the directory structure, file conventions, operational rules, and quality standards the agent must follow. The agent reads this file at the start of every session.

---

## 1. Repository layout

```
/
├── AGENTS.md          ← this file (agent reads first, always)
├── raw/               ← source material, never edited by agent
│   ├── articles/
│   ├── papers/
│   ├── repos/
│   ├── datasets/
│   └── images/
├── wiki/              ← LLM-compiled knowledge base (agent owns this)
│   ├── _index.md      ← master index: one line per article, always kept current
│   ├── _concepts.md   ← flat list of all named concepts with one-line definitions
│   ├── _graph.md      ← adjacency list of concept→concept links
│   ├── concepts/      ← one .md per named concept
│   ├── summaries/     ← one .md per raw/ source document
│   └── topics/        ← topic-level overview articles (cross-concept)
├── insights/          ← human-written notes only. Agent never writes here.
│   └── *.md           ← your own thinking: observations, connections, questions
├── output/            ← query results, slides, charts (agent writes, human reads)
│   ├── reports/
│   ├── slides/        ← Marp .md files
│   └── figures/       ← matplotlib .png files
└── learning/          ← structured learning layer (agent writes, human reviews)
    ├── _review.md     ← spaced repetition queue: concept, due date, interval, ease
    ├── flashcards/    ← one .md per concept with Q&A pairs
    └── gaps.md        ← detected knowledge gaps and open questions
```

---

## 2. Agent identity and scope

The agent is the **sole author and maintainer** of everything under `wiki/`, `output/`, and `learning/`. The human never edits these directories directly.

The agent **never modifies** anything under `raw/` or `insights/`. Raw files are immutable source inputs. Insights files are immutable human outputs — your own thinking, not the agent's synthesis.

The distinction matters: `wiki/` contains what the LLM compiled. `insights/` contains what you actually thought. A summary of a source is noise. An insight you formed from reading it is signal. These belong in different directories with different authorship rules.

The agent **always reads** `AGENTS.md` before any operation. If the schema has changed since the last session, it adapts immediately.

---

## 3. File naming conventions

| Directory | Pattern | Example |
|---|---|---|
| `wiki/concepts/` | `kebab-case.md` | `transformer-attention.md` |
| `wiki/summaries/` | mirrors `raw/` filename | `raw/papers/attention-is-all-you-need.pdf` → `wiki/summaries/attention-is-all-you-need.md` |
| `wiki/topics/` | `kebab-case.md` | `sequence-modelling.md` |
| `output/reports/` | `YYYY-MM-DD-slug.md` | `2026-04-05-attention-survey.md` |
| `output/slides/` | `YYYY-MM-DD-slug.md` | `2026-04-05-weekly-review.md` |
| `output/figures/` | `YYYY-MM-DD-slug.png` | `2026-04-05-loss-curve.png` |
| `learning/flashcards/` | mirrors `wiki/concepts/` | `transformer-attention.md` |

All filenames: lowercase, hyphens only, no spaces, no special characters.

---

## 4. Article schema

Every file under `wiki/concepts/` and `wiki/topics/` opens with this frontmatter block:

```yaml
---
title: "Transformer Attention"
type: concept          # concept | topic | summary
created: 2026-04-05
updated: 2026-04-05
confidence: high       # high | medium | low | speculative
sources:
  - ../summaries/attention-is-all-you-need.md
  - ../summaries/bert-paper.md
related:
  - ../concepts/multi-head-attention.md
  - ../concepts/positional-encoding.md
  - ../topics/sequence-modelling.md
tags: [attention, transformer, nlp]
---
```

`confidence` reflects how well-sourced the article is. The agent downgrades confidence when sources are thin or when the linter finds contradictions.

---

## 5. Index maintenance

The agent **updates `wiki/_index.md` after every write operation**. Format:

```markdown
| Article | Type | Confidence | Updated | Tags |
|---|---|---|---|---|
| [Transformer Attention](concepts/transformer-attention.md) | concept | high | 2026-04-05 | attention, transformer |
```

The agent **updates `wiki/_concepts.md`** whenever a new concept article is created:

```markdown
- **Transformer Attention** — mechanism by which tokens attend to all other tokens with learned weights. → [article](concepts/transformer-attention.md)
```

The agent **updates `wiki/_graph.md`** when backlinks change:

```markdown
transformer-attention → multi-head-attention
transformer-attention → positional-encoding
transformer-attention → sequence-modelling (topic)
```

These three index files are the primary navigation layer. The agent reads them before deciding which full articles to load for any query.

---

## 6. Compilation workflow

When the agent receives a new raw document (e.g. `"file this to our wiki: raw/papers/new-paper.pdf"`):

1. **Read** `AGENTS.md`, `wiki/_index.md`, `wiki/_concepts.md`
2. **Summarise** the raw document → write `wiki/summaries/<filename>.md`
3. **Identify** all named concepts in the document

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arturseo-geo/llm-knowledge-base](https://github.com/arturseo-geo/llm-knowledge-base) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
