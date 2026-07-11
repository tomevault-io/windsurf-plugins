---
trigger: always_on
description: This repository is an AI-assisted personal content knowledge system. Follow `CLAUDE.md` and `KNOWLEDGE_BASE_GUIDE.md`.
---

# Agent Operating Rules

This repository is an AI-assisted personal content knowledge system. Follow `CLAUDE.md` and `KNOWLEDGE_BASE_GUIDE.md`.

If this is a new session, read `START_HERE.md` first.

## Natural-Language Workflows

Treat the following phrases as repository workflow intents. They are natural-language shortcuts, not application slash commands. Accept close Chinese or English variants.

### `加入知识库` / `add to knowledge base`

When the user provides or references material:

1. Inspect the complete material and determine provenance.
2. Put owner-authored input under `raw/`; put external material under `sources/`.
3. Preserve the original meaning and record a source URL or origin note when available.
4. Do not copy non-redistributable full text into a public repository; store permitted metadata and an original summary instead.
5. Check existing aliases and related pages.
6. Create candidate wiki pages under `.kb/staging/wiki/` and candidate sidecars under `.kb/staging/links/` when useful.
7. Report created files, provenance decisions, uncertainties, and review items.

If no material, path, attachment, or URL is available, ask the user what should be added.

### `增加 Wiki 索引` / `add wiki index`

For the specified material or topic:

1. Read the relevant originals, not only existing wiki summaries.
2. Reuse canonical concepts and aliases where possible.
3. Create or update candidate concept, entity, map, or source-note pages under `.kb/staging/wiki/`.
4. Include repository-relative citations to `raw/`, `sources/`, or `products/`.
5. Create matching candidate graph sidecars under `.kb/staging/links/`.
6. Do not overwrite reviewed wiki pages or promote candidates automatically.

### `审核并发布索引` / `review and publish index`

Review the requested staged wiki pages and sidecars. Validate citations, aliases, paths, edge semantics, and source hashes. Promote accepted files to `wiki/` and `.kb/links/`, then update `wiki/changelog.md`. Report anything left in staging.

### `查询知识库` / `query knowledge base`

Follow the Query workflow below. Answer with repository-relative evidence paths and distinguish owner judgment, external claims, published expression, and inference. Do not modify files unless asked.

### `在知识库中进行 AI 内容创作` / `compose from knowledge base`

For an article, course, newsletter, social post, or video script:

1. Establish the topic, audience, format, objective, tone, and important constraints from the request.
2. Search reviewed wiki pages and graph sidecars, then read the relevant originals under `raw/`, `sources/`, and `products/`.
3. Separate owner judgments, external claims, reusable published expression, and new inference.
4. Produce a source plan or outline before long-form drafting when the task is substantial.
5. Write unreviewed prose to `.kb/staging/drafts/`; put course outlines or control files in `.kb/staging/course-drafts/`. Include a Sources section with repository-relative paths.
6. Mark unsupported claims, evidence gaps, and questions for human review. Do not invent citations.
7. Do not publish to `products/` automatically. After the owner accepts the draft, move it to `raw/drafts/` for further development or to `products/` only when publication-ready.
8. After publication, offer to backfill reusable concepts and relationships into staging.

### `回填知识库` / `backfill knowledge base`

Inspect the requested existing content using the Backfill workflow. Generate candidates and reports in `.kb/staging/` or `.kb/reports/`; do not rewrite original or product bodies.

### `检查知识库` / `lint knowledge base`

Run deterministic checks described in Lint. Write a report under `.kb/reports/` when the result is substantial. Do not apply semantic corrections automatically.

### `迁移旧知识库` / `migrate legacy knowledge base`

Follow `docs/MIGRATION_GUIDE.md` and its safety model:

1. Begin with a read-only inventory. Do not move, copy, delete, rename, or rewrite legacy files during inventory.
2. Generate an inventory report under `.kb/reports/` and a proposed mapping under `.kb/staging/migration/`.
3. Classify files by provenance, review state, and intended use before topic. Mark ambiguity as `needs_review`.
4. Wait for mapping approval before copying content.
5. Pilot with 20–50 representative files. Copy rather than move, and never use destructive synchronization.
6. Keep generated wiki pages and graph data in staging.
7. Validate counts, hashes or bodies, frontmatter, links, attachments, aliases, privacy, and copyright risk.
8. Expand only in reviewed batches with one coherent Git commit per accepted batch.
9. Keep the old vault read-only until cutover and recovery checks pass.

## Directory Semantics

- `raw/`: original personal input from the repository owner.
- `sources/`: external, citable source material.
- `products/`: reviewed, published, or delivered personal output.
- `wiki/`: human-readable knowledge index and synthesis layer.
- `.kb/links/`: reviewed, machine-readable relationship graph.
- `.kb/staging/`: AI-generated output awaiting human review.

## Source Of Truth

- `raw/`, `sources/`, and `products/` are authoritative in different roles.
- `wiki/` is an index and synthesis layer, not independent proof.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mrbear1024/ai-content-kb](https://github.com/mrbear1024/ai-content-kb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
