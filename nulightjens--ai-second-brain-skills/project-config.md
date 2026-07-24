---
trigger: always_on
description: This folder is a personal LLM wiki — an AI-maintained knowledge base built on Andrej Karpathy's [LLM wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f).
---

# LLM Wiki Schema

This folder is a personal LLM wiki — an AI-maintained knowledge base built on Andrej Karpathy's [LLM wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f).

The core idea: instead of retrieving chunks from raw documents at query time (RAG), **incrementally build and maintain a persistent wiki** that sits between the human and the raw sources. Knowledge is compiled once and kept current — not re-derived on every query. The wiki compounds.

You are the LLM. The human curates sources and asks questions. You do the bookkeeping: read, extract, integrate, cross-reference, keep consistent. Never complain about the grunt work — it is the whole point.

---

## The map (read this first, every time)

This file is the map for the vault. Think of it as the floor plan posted at the entrance of a building. Before doing anything in this folder, read this file all the way through. The map tells you:

1. Where the rooms are (the folder structure)
2. What lives in each room (`raw/`, `wiki/`, etc.)
3. What workflow to follow for each task (ingest, query, lint)
4. What you must never do (the guardrails)

**Layout for this vault: `{{LAYOUT}}`.** Flat means `wiki/` is a single directory of markdown files. Nested means `wiki/entities/`, `wiki/concepts/`, `wiki/sources/`, `wiki/analyses/`.

## Routing table

For each task, read these files first. Skip the rest. This is the main tool for avoiding wasted context.

| Task                      | Read first                                               | Then                                |
|---------------------------|----------------------------------------------------------|-------------------------------------|
| Ingest a new source       | this file, `wiki/index.md`                               | the source in `raw/`                |
| Answer a query            | this file, `wiki/index.md`                               | the specific wiki pages you need    |
| Lint / self-heal          | this file, `wiki/index.md`, last 10 of `wiki/log.md`     | the audit procedures                |
| Add a cross-reference     | `wiki/index.md`                                          | the two pages being linked          |
| Check the timeline        | last entries of `wiki/log.md`                            | —                                   |
| Diagnose a contradiction  | both affected pages, their `sources:` frontmatter        | the original sources in `raw/`      |

When in doubt, read `wiki/index.md` before reading anything else. It is the routing table — short on purpose, read it every time.

## The three layers

1. **`raw/`** — immutable source documents (articles, PDFs, transcripts, notes). You read from this layer but **never modify it**. This is the source of truth. If you would change anything in `raw/`, stop and ask the human.
2. **`wiki/`** — LLM-owned markdown files. Summaries, entity pages, concept pages, analyses, audits. You create, update, and maintain everything here.
3. **This file + `AGENTS.md`** — the schema. Workflows, conventions, guardrails. Update only when the human asks.

## Naming conventions

Claude navigates this wiki by name. There is no database, no vector store — naming *is* the index. Stay strict about these rules and `"pull my karpathy source"` just works.

- **Wiki pages**: `kebab-case.md` matching the page title. Example: `wiki/mechanical-tension.md`, `wiki/andrej-karpathy.md`.
- **Source files in `raw/`**: `YYYY-MM-DD-source-title.<ext>`. Example: `raw/2026-04-11-karpathy-llm-wiki.md`. The date prefix makes recent sources easy to find with `ls raw/ | sort -r | head`.
- **Audit files**: `wiki/audits/audit-YYYY-MM-DD.md`.
- **Index categories**: use the exact headers in `wiki/index.md` — `Entities`, `Concepts`, `Sources`, `Analyses`, `Audits`. Don't invent new categories without updating this schema file.

A missing or inconsistent name is worse than a missing page — it breaks routing.

## Page format

Every wiki page starts with this frontmatter:

```yaml
---
title: <page title>
tags: [tag1, tag2]
sources:
  - raw/source-file.md
  - https://external-url (accessed YYYY-MM-DD)
related: [[other-page]], [[another-page]]
last_updated: YYYY-MM-DD
---
```

Rules:

- Use `[[wikilinks]]` for cross-references between pages. No markdown links for internal navigation.
- One concept or entity per page. If a page is answering three questions, split it.
- Page title and filename match (`Mechanical Tension` → `wiki/mechanical-tension.md`).
- `sources:` must not be empty. If you can't cite a source, do not write the page.

## `wiki/index.md` — the catalog

Content-oriented catalog. Organized by category (Entities, Concepts, Sources, Analyses, Audits). Each entry is one line:

```
- [[page-name]] — one-line summary
```

**Always read `wiki/index.md` first** on any query. It replaces the need for embedding-based RAG at small-to-moderate scale (up to hundreds of pages).

Update the index on every ingest. A page that exists in `wiki/` but not in `index.md` is invisible — it might as well not exist.

## `wiki/log.md` — the timeline


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NulightJens/ai-second-brain-skills](https://github.com/NulightJens/ai-second-brain-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
