---
trigger: always_on
description: > If you're an agent working on this codebase, read in this order:
---

# MindBase — North Star for Claude Code

> If you're an agent working on this codebase, read in this order:
> 1. **`docs/pivot-plan-2026-05-25.md`** — the CURRENT canonical plan. We
>    are pivoting from "AI second brain" to "AI research analyst that
>    builds a wiki." Everything before this date is legacy direction.
> 2. **`docs/llm-wiki.md`** — Karpathy's pattern, our foundational thesis.
> 3. **This file** — what's built, what's missing, hard contracts, repo
>    conventions.
>
> If a change conflicts with the pivot plan, the pivot plan wins. If a
> change preserves a feature the pivot plan kills (chat-as-home, daily
> journal, SRS UI surface, generic "PKM" framing), push back.

---

## Product thesis (one paragraph)

MindBase implements Andrej Karpathy's **LLM-Wiki pattern** (`docs/llm-wiki.md`),
turned into a product for users who can't or won't run `claude-code + obsidian + a
hand-written CLAUDE.md` themselves. The LLM **incrementally builds and maintains
a persistent wiki** from your raw sources — not RAG-retrieves at query time.
The wiki is a **compounding artifact**: cross-references are pre-baked,
contradictions pre-flagged, synthesis pre-written. The LLM owns the wiki layer;
the user owns sources, exploration, and asking the right questions.

This is NOT another Notion / Obsidian / NotebookLM. Specifically:
- **Notion** is a passive container — you organize, it stores.
- **NotebookLM** is RAG — knowledge is re-derived per query, never accumulates.
- **Obsidian** is a markdown editor — humans do all the maintenance.
- **MindBase** is the active maintainer that gardens your knowledge for you.

## The 3-layer architecture (from Karpathy)

| Layer | Owner | In MindBase |
|---|---|---|
| **Raw sources** — articles, PDFs, web clips, captures. Immutable. | User curates; LLM reads. | `~/mindbase-data/raw/<date>/<id>.{md,meta.json,original.pdf}` |
| **The wiki** — markdown concept pages, entity pages, summaries. | LLM writes & maintains. | `~/mindbase-data/wiki/{concepts,notes,sources,attachments}/` |
| **The schema** — conventions, page formats, ingest workflow. | User and LLM co-evolve. | ⚠️ **Currently hard-coded in prompts**. Should become a per-project user-editable file. |

## The 3 operations (from Karpathy)

| Operation | Karpathy's spec | MindBase status |
|---|---|---|
| **Ingest** | Drop source → LLM reads → **discusses takeaways** → writes summary → updates 10-15 wiki pages → appends to log. | ✅ `apps/server/src/routes/compile.ts` + `packages/core/src/compile/l1.ts`. **❌ No discuss-takeaways turn**. Black-box one-shot today. |
| **Query** | Search → read → synthesize → cite → **good answers get filed back as wiki pages**. | ✅ Hybrid search + chat. **❌ Answers never become wiki pages** — they vanish into chat history. |
| **Lint** | Periodic health check: contradictions / stale claims / orphan pages / missing concepts / suggested investigations. | ❌ **Not implemented**. This is a killer differentiator — Notion / Obsidian / NotebookLM cannot do this. |

## Special files (from Karpathy)

- **`index.md`** — content-oriented catalog the LLM reads first when answering. ❌ MindBase has internal `wikiIndex` (sqlite) but no markdown index file users or the LLM can browse.
- **`log.md`** — chronological append-only. ✅ `wiki/log.md` + `wiki/_changes.md`.

## Beyond Karpathy (where MindBase has an edge today)

- **Typed knowledge graph** — `contradicts` / `supersedes` / `elaborates` / `cites` edges between concepts (`packages/core/src/graph/index/wiki-index.ts`). Karpathy's raw pattern is untyped wikilinks. We can surface contradictions and supersession proactively.
- **Multi-surface capture** — `apps/browser-ext` (Chrome), `apps/test-game/apps/mobile` (mobile), `apps/mcp` (MCP server for other agents).
- **Built-in local PDF/web extraction** — pdfjs-dist, no API roundtrip. (Verified 2026-05-25.)
- **Daily Brief** auto-generation (`apps/server/src/lib/brief.ts`).
- **Spaced repetition** built into the data model (`apps/server/src/lib/srs-worker.ts`).
- **Tab-bar surface for ingested PDFs** with native viewer + extracted text toggle (`RawSourceView`).
- **AI classify-into-folder** for both notes AND raw imports (`POST /api/classify/raw/:rawId` as of 2026-05-25).

## Critical gaps vs the pattern (in priority order)

1. **No user-editable `schema.md` per project** — Karpathy says this is THE key config. Users can't tell the LLM "in this project, concept pages should have these sections" without code changes.
2. **No `lint` operation** — health check that surfaces contradictions, orphans, stale claims.
3. **No file-answer-back** — chat answers are ephemeral. Karpathy's pattern files them back as wiki pages.
4. **No conversational ingest** — current flow is one-shot. Should be: stream takeaways → ask user what to emphasize → commit.
5. **No first-class Project / scope** — everything bleeds into one global wiki. Karpathy's examples (reading a book / researching a topic / planning a trip) all imply *bounded* projects.
6. **No `index.md` markdown catalog** — only internal sqlite.
7. **Wiki is hidden behind a tab** — should be the main surface. (Notes vs Wiki confuses every new user.)

## Product positioning (work in progress, 2026-05-25)

Working hypothesis:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [frankchu91/mindbase-llm-wiki](https://github.com/frankchu91/mindbase-llm-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
