---
trigger: always_on
description: This is a persistent knowledge base. It replaces re-deriving answers from raw documents with a living, compounding intelligence layer maintained by LLM agents.
---

# Knowledge Engine - Wiki Protocol

## Purpose

This is a persistent knowledge base. It replaces re-deriving answers from raw documents with a living, compounding intelligence layer maintained by LLM agents.

Every question answered here stays answered. Every source ingested compounds into structured knowledge. Agents read from this layer instead of starting from scratch.

---

## Architecture

### Layer 1: Sources (Read-Only)

Location: `knowledge-engine/sources/`

Rule: IMMUTABLE. Never modify source files. They are evidence.

Types: PDFs, emails, conversations, web captures, images, exported chat logs, spec documents, contracts, research outputs.

Sub-structure:
```
sources/
  pdfs/          - PDF documents
  emails/        - Email exports
  conversations/ - Chat/meeting transcripts
  web-captures/  - Saved web pages
  images/        - Screenshots, diagrams
```

### Layer 2: Wiki (LLM-Owned)

Location: `knowledge-engine/wiki/`

Rule: LLM agents create and maintain all content. Humans browse and curate.

Structure:
```
wiki/
  {client-slug}/     - One directory per client engagement
  _shared/           - Cross-client concepts, markets, technologies
  _templates/        - Page templates (never instantiate in place)
```

### Layer 3: Schema (Machine-Readable)

Location: `knowledge-engine/schema/`

Files:
- `entities.json` - All named entities (companies, people, products, technologies)
- `graph.json` - Relationship graph between entities and wiki pages
- `tags.json` - Tag registry with page counts and definitions

Rule: Updated by wiki agent after every ingest operation. Schema must stay in sync with wiki content at all times.

Note: `graph.json` and `tags.json` are populated by the bridge's entity extraction during ingest. They start as empty structures and grow with each ingested source. Do not expect them to be pre-populated.

---

## Special Files

| File | Purpose | Update rule |
|---|---|---|
| `index.md` | Master catalog. One entry per wiki page. | After every wiki operation |
| `log.md` | Append-only activity timeline. | After every ingest, query-filed, or lint |
| `schema/entities.json` | Entity registry | After every ingest |
| `schema/graph.json` | Relationship graph | After every ingest |
| `schema/tags.json` | Tag definitions and counts | After every ingest |

---

## Page Format

All wiki pages use YAML frontmatter. The full template:

```markdown
---
title: [Page Title]
type: [overview|entity|concept|source-summary|comparison|deliverable]
client: [client-slug]
sources:
  - [path/to/source/file]
tags:
  - [tag1]
  - [tag2]
related:
  - "[[client/related-page]]"
created: YYYY-MM-DD
updated: YYYY-MM-DD
confidence: [high|medium|low]
---

## Summary

[2-3 sentences. Specific. No abstractions. What is this page about and why does it matter?]

## Key Facts

- [Fact 1] (source: filename, p.N)
- [Fact 2] (source: filename)

## Detail

[Main content organized by subtopic]

## Open Questions

- [What we don't know yet]
- [What needs verification]

## Cross-References

- [[related/page-1]] - [relationship description]
- [[related/page-2]] - [relationship description]
```

Frontmatter field rules:
- `type`: Must match one of the six defined page types
- `client`: Use the slug from Client Directories section below. Use `_shared` for cross-client pages
- `sources`: Relative paths from `knowledge-engine/` root
- `confidence`: Set to `low` if based on a single source or unverified claim. Explain why in the page body.
- `updated`: Agents must update this field on every edit

---

## Page Types

| Type | Purpose | Example |
|---|---|---|
| `overview` | Client or topic summary | `example-client/overview.md` |
| `entity` | Company, person, product, or technology profile | `example-client/product-overview.md` |
| `concept` | Technical concept, framework, or methodology | `_shared/demand-forecasting.md` |
| `source-summary` | Summary of a single ingested source | `example-client/proposal-v2.md` |
| `comparison` | Side-by-side analysis of alternatives | `_shared/vendor-comparison.md` |
| `deliverable` | Work product description and outcome | `example-client/audit-report.md` |

Each type has a distinct role. Do not use `entity` for abstract concepts. Do not use `concept` for named products. When in doubt: if it has a proper name and exists in the world, it is an entity. If it is an idea or methodology, it is a concept.

---

## Client Directories

Each directory under `wiki/` represents one engagement.

| Slug | Client | Domain | Status |
|---|---|---|---|
| `example-client` | Your Client Name | Your domain | Active |
| `_shared` | Shared cross-client knowledge | - | Always active |

Add your own clients by creating directories under wiki/ and running `bridge.py init --clients your-client-name`

Adding a new client manually:
1. Create directory `wiki/[slug]/`
2. Create `wiki/[slug]/overview.md` using the `client-overview.md` template
3. Add entry to `index.md` under Clients section
4. Add entity to `schema/entities.json`
5. Log the creation in `log.md`

---

## Cross-Reference Syntax

Use Obsidian-compatible wikilinks throughout all wiki pages.

Format: `[[path/page-name]]`

Examples:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tashisleepy/knowledge-engine](https://github.com/tashisleepy/knowledge-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
