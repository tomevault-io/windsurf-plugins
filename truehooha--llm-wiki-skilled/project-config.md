---
trigger: always_on
description: > This is the authoritative schema document. The LLM must follow these conventions when maintaining the wiki. Treat this as the system prompt for wiki operations.
---

# LLM Wiki Schema

> This is the authoritative schema document. The LLM must follow these conventions when maintaining the wiki. Treat this as the system prompt for wiki operations.

## Mission

Build and maintain a persistent, compounding knowledge base. The wiki is not a dump of raw text—it is a structured, interlinked artifact that grows richer with every source. Cross-references are pre-built. Contradictions are flagged. Synthesis reflects everything read so far.

## Non-negotiables

1. **Raw sources are immutable.** The LLM never edits files in `raw/`. Ever.
2. **The LLM owns the wiki layer.** All markdown files under `wiki/` are created, updated, and maintained by the LLM unless explicitly overridden by the operator.
3. **Every action is logged.** Every ingest, query result filed, and lint pass appends an entry to `wiki/log.md`.
4. **The index is always current.** `wiki/index.md` must reflect the state of the wiki after every operation that touches pages.
5. **Cross-references are first-class.** Every page must link to related pages. Orphan pages are bugs.

## Directory Layout

```
LLM-wiki/
├── AGENTS.md          # This file. Authoritative schema.
├── README.md          # Human quickstart.
├── raw/               # Immutable source documents.
│   ├── sources/       # Text sources (articles, papers, transcripts).
│   └── assets/        # Downloaded images, data files.
├── wiki/              # LLM-generated markdown. LLM owns this tree.
│   ├── index.md       # Content catalog. Always up to date.
│   ├── log.md         # Append-only timeline.
│   ├── entities/      # Concrete things (people, places, organizations, products).
│   ├── concepts/      # Abstract ideas (theories, frameworks, methodologies).
│   ├── sources/       # One summary page per ingested source.
│   ├── syntheses/     # Cross-source analysis, comparisons, answers.
│   └── templates/     # Reusable page templates.
├── .agents/            # Project-level skill definitions (OpenCode autodiscovery).
│   └── skills/
│       ├── llm-wiki-ingest/   # Ingest workflow skill.
│       ├── llm-wiki-query/    # Query workflow skill.
│       └── llm-wiki-lint/     # Lint workflow skill.
└── verification/      # TDD fixtures and acceptance cases.
```

## Page Types & Conventions

### Entity Pages (`wiki/entities/`)

- Filename: `kebab-case.md`
- Purpose: A concrete thing that appears across multiple sources.
- Frontmatter:
  ```yaml
  ---
  type: entity
  aliases: ["Alternative Name", "Abbreviation"]
  tags: [tag-one, tag-two]
  created: YYYY-MM-DD
  updated: YYYY-MM-DD
  source_count: 0
  ---
  ```
- Sections:
  - `# Entity Name` — H1 title, exact canonical name.
  - `## Identity` — What it is, one concise paragraph.
  - `## Aliases` — List of known aliases.
  - `## Key Attributes` — Structured facts (table or bullet list).
  - `## Evidence` — Links to source summaries that mention this entity, with brief quote or context per link.
  - `## Related` — Links to related entity and concept pages.
  - `## Open Questions` — Uncertainties or gaps about this entity.

### Concept Pages (`wiki/concepts/`)

- Filename: `kebab-case.md`
- Purpose: An abstract idea, theory, or methodology.
- Frontmatter:
  ```yaml
  ---
  type: concept
  aliases: ["Synonym", "Related Term"]
  tags: [tag-one, tag-two]
  created: YYYY-MM-DD
  updated: YYYY-MM-DD
  source_count: 0
  ---
  ```
- Sections:
  - `# Concept Name`
  - `## Definition` — Precise definition in one paragraph.
  - `## Scope` — What this concept covers and what it does not.
  - `## Contrasts` — Differences from related concepts.
  - `## Evidence` — Source links with context.
  - `## Related` — Cross-links.
  - `## Open Questions`

### Source Pages (`wiki/sources/`)

- Filename: `YYYY-MM-DD--source-title-slug.md`
- Purpose: Summary and extraction from a single raw source.
- Frontmatter:
  ```yaml
  ---
  type: source
  source_path: raw/sources/original-filename.pdf
  title: "Exact Title of Source"
  author: "Author Name"
  date: YYYY-MM-DD
  tags: [tag-one, tag-two]
  created: YYYY-MM-DD
  ---
  ```
- Sections:
  - `# Source Title`
  - `## Summary` — 3-5 sentence overview.
  - `## Key Claims` — Numbered list of main assertions.
  - `## Notable Quotes` — Direct quotes with page/section references if available.
  - `## Entities Mentioned` — Links to entity pages created or updated.
  - `## Concepts Mentioned` — Links to concept pages.
  - `## Follow-ups` — Questions or leads from this source.

### Synthesis Pages (`wiki/syntheses/`)

- Filename: `YYYY-MM-DD--question-slug.md` or `YYYY-MM-DD--topic-slug.md`
- Purpose: Cross-source analysis, answers to questions, comparisons.
- Frontmatter:
  ```yaml
  ---
  type: synthesis
  question: "The exact question this page answers"
  tags: [tag-one, tag-two]
  created: YYYY-MM-DD
  updated: YYYY-MM-DD
  ---
  ```
- Sections:
  - `# Synthesis Title`
  - `## Question / Purpose`
  - `## Answer / Analysis`
  - `## Comparison Table` (if applicable)
  - `## Citations` — Links to source pages with brief evidence snippets.
  - `## Implications` — Why this matters.
  - `## Follow-up Questions`

## Citation Rules


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TrueHOOHA/LLM-Wiki-Skilled](https://github.com/TrueHOOHA/LLM-Wiki-Skilled) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
