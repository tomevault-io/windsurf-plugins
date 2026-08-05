---
trigger: always_on
description: > This is the authoritative schema document. The LLM must follow these conventions when maintaining the wiki. Treat this as the system prompt for wiki operations.
---

# LLM Wiki Schema

> This is the authoritative schema document. The LLM must follow these conventions when maintaining the wiki. Treat this as the system prompt for wiki operations.

## Mission

Build and maintain a persistent, compounding knowledge base. The wiki is not a dump of raw text—it is a structured, interlinked artifact that grows richer with every source. Cross-references are pre-built. Contradictions are flagged. Synthesis reflects everything read so far.

## OKF Conformance

The `wiki/` tree is an **[Open Knowledge Format (OKF)](SPEC.md)** bundle (v0.1). `wiki/` is the bundle root; every `.md` file under it that is not a reserved filename (`index.md`, `log.md`) is an OKF *concept document*. Files outside `wiki/` (`AGENTS.md`, `README.md`, `SPEC.md`, `raw/**`) are project-level artifacts, not part of the bundle, and are not subject to OKF conformance.

OKF (SPEC.md §9) requires only that every concept document carry a parseable YAML frontmatter block with a non-empty `type` field, and that reserved files follow §6/§7. This document defines the *producer conventions* layered on top of OKF: the four `type` values used (entity, concept, source, synthesis), their recommended sections, the cross-link and citation style, and the index/log contracts. These are permitted OKF extensions (SPEC.md §4.1 Extensions, §4.2 body). Where this document is silent, OKF (SPEC.md) governs.

**Obsidian-first deviation from OKF §5.** OKF §5 recommends standard markdown links for cross-linking. This bundle instead uses **Obsidian wikilinks** (`[[note]]`) for all internal note-to-note links, to provide first-class Obsidian graph-view support. Link format is not an OKF §9 conformance requirement, and OKF's permissive consumption model (§9) tolerates this. External URLs remain standard markdown links (`[text](https://…)`) — wikilinks are for internal notes only.

## Non-negotiables

1. **Raw sources are immutable.** The LLM never edits files in `raw/`. Ever.
2. **The LLM owns the wiki layer.** All markdown files under `wiki/` are created, updated, and maintained by the LLM unless explicitly overridden by the operator.
3. **Every action is logged.** Every ingest, query result filed, and lint pass appends an entry to `wiki/log.md`.
4. **The index is always current.** `wiki/index.md` must reflect the state of the wiki after every operation that touches pages.
5. **Cross-references are first-class.** Every page should link to related pages. Orphan pages are bugs.

## Directory Layout

```
LLM-Wiki/
├── AGENTS.md          # This file. Authoritative schema.
├── README.md          # Human quickstart.
├── SPEC.md            # Open Knowledge Format (OKF) v0.1 spec this bundle follows.
├── raw/               # Immutable source documents (outside the OKF bundle).
│   ├── sources/       # Text sources (articles, papers, transcripts).
│   └── assets/        # Downloaded images, data files.
├── wiki/              # OKF bundle root. LLM-generated markdown. LLM owns this tree.
│   ├── index.md       # Bundle-root index (OKF §6). Declares okf_version.
│   ├── log.md         # Update history (OKF §7).
│   ├── entities/      # Concept docs: concrete things (people, places, organizations, products).
│   ├── concepts/      # Concept docs: abstract ideas (theories, frameworks, methodologies).
│   ├── sources/       # Concept docs: one summary page per ingested source.
│   ├── syntheses/     # Concept docs: cross-source analysis, comparisons, answers.
│   └── templates/     # Reusable page templates (not concept docs; excluded from index/lint).
└── .agents/            # Project-level skill definitions (OpenCode autodiscovery).
    └── skills/
        ├── llm-wiki-ingest/   # Ingest workflow skill.
        ├── llm-wiki-query/    # Query workflow skill.
        └── llm-wiki-lint/     # Lint workflow skill.
```

> **Enforcement model.** Workflow rigidity is enforced entirely by the skills under `.agents/skills/` against the schema defined in this file. There are no helper scripts. `AGENTS.md` is the single source of truth for every page type, frontmatter field, section, index format, and log contract. Skills must never re-define the schema inline — they reference and enforce this document.

## Frontmatter Contract

Every concept document (every `.md` under `wiki/` except `index.md` and `log.md`) MUST include a YAML frontmatter block delimited by `---`.

**Required (OKF §4.1):**
- `type` — `entity` | `concept` | `source` | `synthesis`. Non-empty string. This is the only field OKF strictly requires.

**Recommended (OKF §4.1, in priority order):**
- `title` — Human-readable display name. If omitted, consumers may derive it from the filename. Expected on every page.
- `description` — A single sentence summarizing the concept. Used by `index.md` generators, search snippets, and previews. Expected on every page so the index can list it.
- `tags` — YAML list of short strings for cross-cutting categorization.
- `timestamp` — ISO 8601 (date or datetime) of last meaningful change. Replaces the legacy `updated` field.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TrueHOOHA/LLM-Wiki](https://github.com/TrueHOOHA/LLM-Wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
