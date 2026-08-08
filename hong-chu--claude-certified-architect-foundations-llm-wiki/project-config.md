---
trigger: always_on
description: > This file is the **schema**. It tells the agent how this repo is
---

# CLAUDE.md — LLM Wiki Schema

> This file is the **schema**. It tells the agent how this repo is
> laid out, what files mean, and which workflows to run. Humans read
> the wiki; the agent maintains it.
> Pattern adapted from
> [Andrej Karpathy's *llm-wiki* gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f).

## TL;DR for the agent

You are the maintainer of a wiki about **the Claude Certified Architect – Foundations certification exam**: the Claude/Anthropic platform knowledge, prompt-engineering and agent-design principles, API features, and exam objectives needed to study for and pass the exam.

The user curates raw material into `sources/`. You compile it into a
clean, interlinked set of pages under `wiki/`. You never modify
`sources/`. You always update `wiki/log.md`.

## Three layers

```
<vault root>/
├── CLAUDE.md       # Layer 3 — schema (this file)
├── README.md       # human entry point
├── sources/        # Layer 1 — raw, immutable inputs (the user owns this)
├── wiki/           # Layer 2 — derived markdown (you own this)
│   ├── index.md
│   ├── log.md
│   ├── start-here/                 # cross-cutting "how to think" — read first
│   │   (overview, first-principles, root-cause-decision-tree,
│   │    never-right-answers, and exam-wide question pages)
│   ├── domain-1-agentic-architecture/ # Agentic Architecture & Orchestration (27%)
│   ├── domain-2-tool-design-mcp/      # Tool Design & MCP Integration (18%)
│   ├── domain-3-claude-code-config/   # Claude Code Configuration & Workflows (20%)
│   ├── domain-4-prompt-engineering/   # Prompt Engineering & Structured Output (20%)
│   ├── domain-5-context-reliability/  # Context Management & Reliability (15%)
│   └── reference/                     # products & SDKs (entities): SDK, Claude Code, MCP, …
└── derived/        # optional: charts, slides, exports built from wiki/
```

### Organizing principle — domain-first
This wiki is for **exam study**, so pages are grouped by **exam domain**,
not by page type. Each `domain-N-*/` folder is a self-contained study
unit containing that domain's:
- **synthesis** (the domain hub, named the same as its folder, e.g.
  `domain-1-agentic-architecture/domain-1-agentic-architecture.md`),
- **concept** pages (one idea/technique/term each), and
- **comparison** pages (`x-vs-y.md` decision pairs).

The five **page *types*** (concept · entity · synthesis · comparison ·
question) still define how a page is *written* — see Page templates
below — they just no longer define *where it lives*. Placement rules:
- A page that belongs to one domain → that `domain-N-*/` folder.
- A cross-cutting page (spans all domains: principles, decision trees,
  trap catalogs, exam-wide questions) → `start-here/`.
- An **entity** (a product/SDK/spec referenced across domains) →
  `reference/`.
- A question scoped to one domain → that domain folder; otherwise
  `start-here/`.

Obsidian resolves `[[wiki-links]]` by **filename**, so a page can be
moved between folders without breaking links — keep filenames unique
and `kebab-case.md`.

**Sort-order convention** (so the file tree reads top-down):
- Each domain's **hub** is named the **same as its folder** (the
  "folder-note" convention) so the filename is the domain's real title,
  e.g. `domain-4-prompt-engineering/domain-4-prompt-engineering.md`. No
  `00-` prefix. (With **Show inline title** off, the page displays its
  H1 — "Domain 4 — Prompt Engineering & Structured Output" — not the
  filename. Tradeoff: the hub sorts mid-folder, not first.)
- `start-here/` pages carry a two-digit reading-order prefix
  (`00-overview`, `01-first-principles`, `02-root-cause-decision-tree`,
  `03-never-right-answers`, then focused question pages `04+`).
- When renaming a page, update its `[[wiki-links]]` everywhere
  (`wiki/`, `sources/*.meta.md`, `README.md`) and re-run the broken-link
  check.

### Layer 1 — `sources/`  (immutable)
- The user drops in PDFs, papers, screenshots, transcripts, blog
  dumps, notebooks, links collected as `.md` clippings, etc.
- **You read but never write here.** If a source needs cleanup, copy
  it, don't mutate it.
- Each source should ideally have a sibling `*.meta.md` with: title,
  author, url, date, why-it-matters. If missing, create it on first
  ingest.

### Layer 2 — `wiki/`  (the wiki — you own it)
- One concept, entity, synthesis, comparison, or question per file.
- File names: `kebab-case.md` (e.g. `mixture-of-experts.md`).
- Cross-link liberally with **wiki-style links**:
  `[[mixture-of-experts]]`, `[[entities/openai]]`. Every page should
  link out to ≥ 2 others.
- Every page must end with a `## Sources` section listing the
  `sources/` files (or external URLs) it draws from.

### Layer 3 — `CLAUDE.md`  (the schema)
- This file. Update it when conventions evolve. Treat changes here
  as schema migrations — note them in `wiki/log.md`.

## Page templates

Pages are written for *humans first* and agents second. Every page
opens with a one-sentence summary as a markdown blockquote (`> …`),
followed by structured prose, and closes with a `## Continue reading`
footer that points the reader at 2–3 curated next pages. Wiki-links
use `[[wiki-link]]` syntax; every page has a `## Sources` section.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hong-chu/claude-certified-architect-foundations-llm-wiki](https://github.com/hong-chu/claude-certified-architect-foundations-llm-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
