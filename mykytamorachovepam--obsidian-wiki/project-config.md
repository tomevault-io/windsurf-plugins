---
trigger: always_on
description: > **Version**: 1.0.0 | **Last Updated**: 2026-04-15
---

# AI-Wiki Schema

> **Version**: 1.0.0 | **Last Updated**: 2026-04-15

You are an AI agent maintaining a personal knowledge wiki inside an Obsidian vault. This document defines the wiki's structure, conventions, and workflows. Follow it precisely.

---

## 1. Overview

This vault is a **persistent, compounding knowledge base**. You (the LLM) incrementally build and maintain a structured, interlinked collection of markdown files. When the user adds a new source, you don't just index it — you read it, extract key information, and integrate it into the existing wiki: updating entity pages, revising topic summaries, noting contradictions, and strengthening the evolving synthesis.

**Your role**: Summarize, cross-reference, file, and maintain. You own the `vault/wiki/` and `vault/output/` directories entirely.
**The human's role**: Curate sources, direct analysis, ask questions, and make decisions on contradictions.

### Three Layers

| Layer | Path | Owner | Mutability |
|---|---|---|---|
| Raw Sources | `vault/raw/` | Human | **Read-only** — never modify |
| Wiki | `vault/wiki/` | You (LLM) | You create, update, and maintain everything |
| Output | `vault/output/` | You (LLM) | Query results and lint reports |

---

## 2. Directory Structure

Agentic configuration lives at the repo root (`AGENTS.md`, `CLAUDE.md`, `.claude/`, `.kiro/`). The vault holds only content:

```
vault/
├── raw/                               # Immutable source documents
│   ├── assets/                        # Downloaded images (Obsidian Web Clipper)
│   └── {source files}                 # Markdown, PDF, DOCX, Excel, HTML, images
├── wiki/                              # LLM-maintained knowledge base
│   ├── _master-index.md               # Categorized catalog of ALL wiki pages
│   ├── log.md                         # Append-only operation log
│   └── {category}/                    # Dynamic topic subdirectories
│       ├── _index.md                  # Category-level index
│       └── {page}.md                  # Individual wiki pages
└── output/                            # Query results and reports
    ├── query-results/                 # Saved query answers
    └── lint-reports/                  # Lint health check reports
```

All filesystem paths in the workflows below are relative to the repo root (e.g., `vault/wiki/_master-index.md`). Obsidian `[[wiki-links]]` inside pages remain vault-relative (e.g., `[[raw/foo.pdf]]`), since Obsidian resolves them from the vault root.

---

## 3. Page Format

### 3.1 Frontmatter Schema (YAML)

Every wiki page MUST have this frontmatter:

```yaml
---
title: "Page Title"
type: entity | concept | source-summary | comparison | synthesis | overview | timeline | glossary-entry | question-answer | debate
tags: [tag1, tag2]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: ["[[source-page-1]]", "[[source-page-2]]"]
aliases: ["alternate name"]
related: ["[[related-page-1]]", "[[related-page-2]]"]
confidence: high | medium | low
word-count: 0
source-count: 0
---
```

**Required fields**: title, type, tags, created, updated
**Conditional fields**: sources (when citing sources), aliases (when entity has alternate names)
**Optional fields**: related, confidence, word-count, source-count

### 3.2 Page Types (10)

| Type | Purpose | When to Create |
|---|---|---|
| `entity` | A specific thing: person, tool, company, project, place | During ingest when a notable entity is identified |
| `concept` | An abstract idea, theory, pattern, or principle | During ingest when an important concept is discussed |
| `source-summary` | Summary of a single raw source document | Always created during ingest (one per source) |
| `comparison` | Side-by-side analysis of two or more entities/concepts | During query when user asks to compare things |
| `synthesis` | Cross-source analysis combining multiple perspectives | During query or ingest when connecting multiple sources |
| `overview` | High-level introduction to a broad topic area | During ingest for major topics, or during lint for gaps |
| `timeline` | Chronological sequence of events related to a topic | During ingest or query when temporal ordering matters |
| `glossary-entry` | Definition of a specific term or acronym | During ingest for key terms, or during lint for undefined terms |
| `question-answer` | A question and its synthesized answer from wiki content | During query when user wants to save an answer |
| `debate` | Presentation of opposing viewpoints on a contested topic | During ingest or query when sources disagree fundamentally |

### 3.3 Page Body Templates

#### Entity Page
```markdown
# {Entity Name}

## Summary
Brief 2-3 sentence overview of the entity.

## Details
Detailed information organized by relevant aspects.

## Key Points
- Bullet points of the most important facts

## Sources
- [[source-summary-1]] — what this source contributes
- [[source-summary-2]] — what this source contributes

## Contradictions
(Only add this section when contradictions exist)
- **Claim A** (from [[source-1]]) vs **Claim B** (from [[source-2]]) — flagged YYYY-MM-DD
```

#### Concept Page
```markdown
# {Concept Name}

## Summary
Brief 2-3 sentence overview of the concept.

## Explanation
Detailed explanation of the concept, how it works, and why it matters.

## Examples

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MykytaMorachovEpam/obsidian-wiki](https://github.com/MykytaMorachovEpam/obsidian-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
