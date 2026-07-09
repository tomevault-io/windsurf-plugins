---
trigger: always_on
description: This project is a self-contained Markdown-first LLM wiki. Any agent can use this folder as its working directory and run all core workflows from local files and npm scripts.
---

# Agent Wiki Rules

This project is a self-contained Markdown-first LLM wiki. Any agent can use this folder as its working directory and run all core workflows from local files and npm scripts.

Obsidian is optional. Codex skills are optional. The source of truth is this repository.

## Bootstrap

When starting in this workspace:

1. Read this `AGENTS.md`.
2. Read `README.md`.
3. Read `wiki/index.md` before answering from the knowledge base.
4. Run `npm run wiki:status` before broad maintenance, link repair, or dashboard work.
5. Use root npm scripts or `node scripts/karpathy-wiki.mjs ...` for automation.

Do not create a separate `codex.md`; this file is the agent rule source of truth.

## Operating Model

- `raw/` is the factual source layer. Do not rewrite source captures after they are stored except for small metadata fixes.
- `wiki/` is the compiled knowledge layer. Synthesize, link, correct, split, and merge wiki pages as understanding improves.
- `templates/` contains shared capture and wiki templates.
- `scripts/` contains the local CLI. Do not depend on globally installed skills.
- `tools/wiki-dashboard/` is read-only graph/dashboard code. It reads Markdown and generated JSON; it is never the source of truth.
- `.obsidian/`, if added by a user, is optional editor configuration and must not become a runtime dependency.

## Karpathy-Style Wiki Pages

Treat the wiki layer as an atomic LLM-maintained wiki, not as a folder of source summaries.

- Prefer one durable knowledge unit per wiki page: one concept, entity, method, API, workflow, comparison, or recurring question.
- A single raw source can update many wiki pages. Do not create only one summary page when the source contains multiple reusable ideas.
- Create missing concept pages when a source repeatedly mentions an important idea that has no durable home yet.
- Keep `wiki/index.md`, `wiki/log.md`, and README files as navigation or maintenance records, not graph knowledge nodes.
- Split pages that mix unrelated knowledge units; merge pages that are duplicate names for the same knowledge unit.
- Link every source-backed claim to raw evidence, but keep the wiki prose synthesis-oriented rather than copied from the source.

## Commands

Prefer root npm scripts:

```bash
npm run wiki:status
npm run wiki:lint
npm run wiki:garden
npm run wiki:universes
npm run wiki:repair-links
npm run wiki:search -- "query terms"
npm run wiki:capture -- --title "Source title" --url "https://example.com"
npm run wiki:images -- --source raw/source-note.md
npm run dashboard
npm run dashboard:open
```

Dashboard/frontend commands are on-demand visualization tools. Do not refresh, start, or open the dashboard during routine ingest or wiki edits unless the user explicitly asks to view the knowledge graph, open the frontend, inspect the dashboard, or work on visualization.

Treat short user requests such as `看知识图谱`, `打开知识图谱`, `打开前端`, `打开 dashboard`, `show the graph`, `open the frontend`, or `open the dashboard` as complete requests to run `npm run dashboard:open`. That command should refresh the graph, ensure the local Agent Wiki frontend is running, and open `http://127.0.0.1:5173/` in the browser.

## Firecrawl MCP

This workspace provides `.mcp.json` for hosted Firecrawl MCP at `https://mcp.firecrawl.dev/v2/mcp`.

- Treat Firecrawl MCP as an optional agent capture tool, not as a required project dependency.
- The hosted keyless tier supports quick `scrape`, `search`, and `interact` workflows without a Firecrawl API key, with rate limits.
- Full Firecrawl tools such as `crawl`, `map`, `agent`, and `extract` require Firecrawl auth; only use them when the user has configured access.
- Prefer Firecrawl MCP when direct capture fails, when pages need rendering/interactions, or when web search is needed before selecting sources.
- After using MCP, ingest selected evidence into `raw/` with `npm run wiki:capture` via stdin or `--content-file`; do not treat MCP output alone as durable vault state.
- Respect site terms, robots policy, privacy constraints, and user authorization before crawling or mirroring content.
- If Firecrawl MCP tools are not visible in the current thread, the agent may need a thread/app reload after `.mcp.json` is added.

## Ingest

Use this when adding a webpage, article, PDF, transcript, or long-form note.

1. Create a raw source note in `raw/` using `templates/raw-source.md` or `npm run wiki:capture`.
2. Preserve bibliographic details and evidence metadata: title, author, date, URL, captured date, type, status, `snapshot_path`, `content_hash`, `capture_method`, and `source_quality`.
3. Preserve inline image order and mirror remote images when practical.
4. For image-rich webpages, run `npm run wiki:images -- --source raw/source-note.md` after capture or after adding snapshot paths. This downloads images into `raw/assets/<source-note>/`, writes `image-index.json`, and updates the raw note's `## Images` section.
5. Promote only the most useful visual evidence into wiki pages. Keep exhaustive image inventories in raw notes.
6. Extract durable claims and entities into existing or new pages under `wiki/`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NimaChu/agent-wiki](https://github.com/NimaChu/agent-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-09 -->
