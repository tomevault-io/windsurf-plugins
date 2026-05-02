---
trigger: always_on
description: This file governs how Claude Code maintains the personal knowledge wiki in this vault. Read it at the start of every session before doing any wiki work.
---

# LLM Wiki — Schema & Operating Instructions

This file governs how Claude Code maintains the personal knowledge wiki in this vault. Read it at the start of every session before doing any wiki work.

---

## What This Wiki Is

This is a **persistent, compounding knowledge base** — not a RAG index. When you add a source, Claude reads it, extracts key knowledge, and integrates it into interlinked wiki pages. The wiki accumulates understanding over time; cross-references, contradictions, and syntheses are already there when you need them.

You (the human) are responsible for: sourcing material, exploration, and asking the right questions.
Claude is responsible for: summarizing, cross-referencing, filing, and maintaining the wiki.

---

## Directory Layout

```
claude-second-brain/
├── CLAUDE.md              ← This file. The schema.
├── raw-sources/           ← Raw source material. IMMUTABLE — Claude never modifies these.
│   ├── articles/          ← Web articles saved as markdown
│   ├── pdfs/              ← PDF files or extracted text
│   └── personal/          ← Personal notes flagged for ingestion
├── wiki/                  ← LLM-maintained knowledge base. Claude owns this entirely.
│   ├── index.md           ← Master index of all wiki pages
│   ├── log.md             ← Running log of all wiki activity
│   ├── overview.md        ← Evolving high-level synthesis
│   ├── sources/           ← One summary page per ingested source
│   ├── qa/                ← Filed answers to notable queries
│   └── [topic/entity pages at root of wiki/]
└── [existing vault files] ← Daily notes, misc, ideas, etc. Claude never touches these.
```

---

## Page Types

### `overview` — `wiki/overview.md`
The single evolving synthesis page. Updated when a new source meaningfully shifts the overall picture. Structured as a narrative with cross-links, not a list.

### `topic` — `wiki/[kebab-name].md`
Notes on a concept, technology, domain, or idea. Examples: `wiki/distributed-systems.md`, `wiki/machine-learning-ops.md`, `wiki/investing-frameworks.md`.

### `entity` — `wiki/[name].md`
Notes on a specific person, company, tool, project, or place. Examples: `wiki/anthropic.md`, `wiki/kafka.md`, `wiki/ray-dalio.md`.

### `source-summary` — `wiki/sources/[slug].md`
Summary of one ingested source. Slug is kebab-case derived from the title. One page per source.

### `qa` — `wiki/qa/[slug].md`
A filed answer to a notable query. Created when an answer synthesizes multiple pages in a way worth preserving.

### `admin` — `wiki/index.md`, `wiki/log.md`
Special system/administrative pages. Not synthesis content. There are exactly two: `index.md` (master index) and `log.md` (activity log).

---

## Frontmatter Format

All wiki pages use this YAML frontmatter:

```yaml
---
type: overview | topic | entity | source-summary | qa | admin
tags: [tag1, tag2]
sources: ["[[wiki/sources/source-slug]]"]
related: ["[[wiki/related-page]]"]
updated: YYYY-MM-DD
---
```

- `type`: Required. One of the six page types above.
- `tags`: Optional. Lowercase, hyphenated. Topics for the Obsidian tag pane.
- `sources`: Links to the `wiki/sources/` pages that informed this page.
- `related`: Links to other wiki pages directly relevant to this one.
- `updated`: Date this page was last meaningfully revised.

---

## Naming Conventions

- **File names**: kebab-case, all lowercase. `machine-learning-ops.md`, not `MachineLearningOps.md`.
- **Links**: Always use Obsidian `[[wikilinks]]` — never bare URLs or markdown `[text](path)` links for internal cross-references.
- **Link format**: Use the full path from vault root: `[[wiki/distributed-systems]]`, `[[wiki/sources/attention-is-all-you-need]]`.
- **Section headings**: Use `##` for top-level sections within a page, `###` for subsections.
- **Contradictions**: Mark with `> [!WARNING] Contradiction` callout. Explain both claims and their sources.
- **Uncertainty**: Mark claims you're unsure about with `[?]` inline.

---

## Ingest Workflow

Run this workflow whenever the user adds a new source. Do not skip steps.

**Step 1 — Read the source**
- If URL: fetch and read the full content.
- If file in `raw-sources/`: read it with the Read tool.
- If pasted text: treat as the source.

**Step 2 — Discuss with the user**
- Summarize the source in 3-5 bullets.
- Ask: what aspects matter most? Anything to emphasize or skip?
- Let the user's response shape which topics get deep treatment.

**Step 3 — Create source summary page**
- File: `wiki/sources/[slug].md`
- Sections: title, author/date, one-paragraph abstract, key claims (bulleted), notable quotes (max 3), your synthesis note, links to wiki pages this source touches.

**Step 4 — Update `wiki/index.md`**
- Add the new source to the Sources section with a one-line description and link.

**Step 5 — Identify affected wiki pages**
- Run `INDEX_PATH=qmd.sqlite pnpm dlx @tobilu/qmd query -c wiki "<source topic and key claims>"` to surface related existing wiki pages.
- Also Glob `wiki/*.md` and `wiki/sources/*.md` to ensure completeness.
- List all pages to create or update.

**Step 6 — Update or create wiki pages**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jessepinkman9900/claude-second-brain](https://github.com/jessepinkman9900/claude-second-brain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
