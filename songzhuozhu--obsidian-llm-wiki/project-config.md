---
trigger: always_on
description: This file is the single authoritative rule entry point for the `inspool-wiki-en/` workspace and guides agents that maintain this knowledge base.
---

# Inspool Wiki Maintenance Rules

This file is the single authoritative rule entry point for the `inspool-wiki-en/` workspace and guides agents that maintain this knowledge base.

## Core Goal

Gradually compile scattered raw material into a structured, cross-referenced, continuously evolving English wiki.

The agent's job is not to improvise one-off answers. The job is to maintain the knowledge structure itself.

## Default Language

- The default language preference for this workspace is English.
- Unless the user explicitly requests another language, agents should write, update, and organize documents in English.
- This applies to page content, documentation, indexes, logs, and command templates.
- Raw source material may remain in its original language and does not need forced translation, but the wiki layer should default to English.

## Three-Layer Structure

### 1. `raw/`

- Stores raw source material such as web clips, articles, book excerpts, meeting notes, PDF transcripts, screenshots, and similar inputs.
- This is the factual source layer.
- The body is read-only by default.
- Only two workflow-level changes are allowed by default: moving file paths and updating a small amount of frontmatter processing metadata.
- Unless the user explicitly asks for it, do not rewrite, trim, or replace the body content under `raw/`.

### 2. `wiki/`

- Stores the structured knowledge pages maintained by the agent.
- Agents may create, update, rename, and cross-link pages here.
- When answering questions, read and synthesize from this layer first.

### 3. `AGENTS.md`

- Defines structural conventions, page rules, workflow steps, and quality standards.
- If practice reveals a better workflow, evolve this file only with user agreement.

## Directory Conventions

### `raw/`

- `raw/unprocessed/`: newly captured material waiting to be handled
- `raw/processed/`: material that completed ingest and approval
- `raw/assets/`: images, attachments, and other static assets saved from source material
- `raw/index.md`: processing-state index

### `wiki/`

- `wiki/sources/`: summary pages for individual sources
- `wiki/entities/`: pages for people, organizations, products, projects, tools, and other entities
- `wiki/concepts/`: pages for concepts, methods, frameworks, and terms
- `wiki/synthesis/`: topical overviews, comparisons, interim conclusions, and written-back Q&A results
- `wiki/meta/`: overview pages, conventions, scope notes, and maintenance metadata
- `wiki/index.md`: content index
- `wiki/log.md`: timeline log

Notes:

- Users may manually create subfolders under `wiki/concepts/`, `wiki/entities/`, and `wiki/synthesis/` for classification.
- Those subfolders are an organizational layer maintained by the user, not immutable semantic truth.
- Agents must remain compatible with arbitrary nesting inside those directories.

## Page Writing Principles

- Write in English by default unless the user explicitly asks for another language.
- Prefer clear headings and short paragraphs.
- Prefer Obsidian-style wiki links such as `[[Some Concept]]`.
- Add source-page references to important facts when possible.
- Separate facts, judgments, and speculation.
- If sources conflict, record the conflict explicitly instead of smoothing it away.

## User-Managed Classification Directories

- Users may create subfolders under `wiki/concepts/`, `wiki/entities/`, and `wiki/synthesis/`, then move pages into them.
- Treat that as normal maintenance behavior, not an error condition.
- Page type is determined by the top-level directory, such as `concepts / entities / synthesis`, not by deeper subfolder names.
- Deeper subfolders are mainly for user organization; do not overfit your reasoning to those paths.
- Unless the user explicitly asks for it, do not reorganize those classification directories on their behalf.
- When reading, updating, or linting pages, scan all subfolders recursively.

## Path Stability Rules

- Do not hard-code full file paths under `wiki/concepts/...`, `wiki/entities/...`, or `wiki/synthesis/...` as if those paths were part of the knowledge semantics.
- Prefer Obsidian wiki links in body text and frontmatter instead of bare path strings.
- If `[[Page Name]]` or another stable wiki link works, do not write a plain-text path instead.
- When a page is moved into a new classification subfolder, treat that as valid and keep operating normally.
- If classification changes leave stale path references behind, fix the references instead of asking the user to revert the structure.
- The move from `raw/unprocessed/` to `raw/processed/` is a workflow transition, not a stable semantic path.
- Therefore, source-page `raw_note` fields, body links to raw material, and any other explicit raw paths must be updated after approval.

## Evidence Nodes and Graph Principles

- Pages under `wiki/sources/` are evidence nodes in the graph.
- Pages under `wiki/entities/`, `wiki/concepts/`, and `wiki/synthesis/` are knowledge nodes.
- For key conclusions on knowledge pages, prefer links to local `sources` pages instead of direct external URLs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [songzhuozhu/obsidian-llm-wiki](https://github.com/songzhuozhu/obsidian-llm-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
