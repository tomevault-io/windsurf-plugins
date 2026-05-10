---
trigger: always_on
description: Enables cross-entity queries: "everything Shadow knows about project X" across all three systems.
---

# Shadow — Developer Guide

## What is Shadow

Shadow is a local-first engineering companion that runs as a background daemon, learns from your work, and interacts via Claude CLI (MCP) and a web dashboard. It's 100% LLM-based — Claude is the brain, Shadow is the persistence and observation layer.

## Architecture

```
User ← Claude CLI (MCP tools) → Shadow daemon (port 3700)
                                    ├── SQLite DB (~/.shadow/shadow.db)
                                    ├── Web dashboard (React, localhost:3700)
                                    ├── Heartbeat (every 30min)
                                    │   ├── detect active projects
                                    │   ├── summarize (Opus, text-free → session summary)
                                    │   ├── extract (Opus, JSON → memories + mood)
                                    │   ├── cleanup (Sonnet, MCP → resolve stale obs)
                                    │   └── observe (Opus, JSON → new observations)
                                    ├── Daemon jobs
                                    │   ├── suggest (LLM, project-aware)
                                    │   ├── consolidate (memory maintenance, 6h)
                                    │   ├── reflect (soul reflection, daily)
                                    │   ├── remote-sync (git ls-remote, 30min)
                                    │   ├── pr-sync (gh pr view for awaiting_pr runs, 30min)
                                    │   ├── context-enrich (MCP enrichment)
                                    │   └── auto-plan / auto-execute (autonomy)
                                    ├── Hooks (7: session + tool use + prompts + responses + errors + subagents + statusline)
                                    └── service manager (launchd on macOS, systemd --user on Linux)
```

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Runtime | Node.js 22+ (ESM) |
| Language | TypeScript 5.9+ (strict) |
| Storage | SQLite (node:sqlite DatabaseSync, WAL mode, busy_timeout=5000) |
| Search | FTS5 (BM25) + sqlite-vec (cosine) — hybrid via RRF |
| Embeddings | @huggingface/transformers, all-MiniLM-L6-v2 (384 dims, local) |
| CLI | Commander.js 14 |
| Validation | Zod 4 |
| LLM Backend | Claude CLI (`--print --output-format json`) or Agent SDK |
| MCP | JSON-RPC over HTTP on `/api/mcp` (69 tools); a stdio server also exists for legacy clients (`shadow mcp serve`) |
| Dashboard | React 19, Vite, Tailwind CSS 4, React Router 7 |
| Daemon | launchd (macOS, KeepAlive=true) or systemd --user (Linux, Restart=always) |

## Dashboard Routes

| Route | Page | Purpose |
|-------|------|---------|
| `/morning` | Morning | Daily brief: active projects, metrics, runs, memories, observations, suggestions |
| `/profile` | Profile | Settings: identity, behavior, models, soul, thoughts, enrichment, autonomy |
| `/chronicle` | Chronicle | Bond system: radar, tier path, timeline, unlocks |
| `/memories` | Memories | Search + layer filter + pagination |
| `/suggestions` | Suggestions | Filter tabs, pagination, accept/dismiss, bulk actions |
| `/observations` | Observations | Filter by status/severity, votes, ack/resolve/reopen |
| `/repos` | Repos | Repo profile cards with correction panel |
| `/projects`, `/projects/:id` | Projects | Cards + drill-down to detail |
| `/team` | Team | Contacts management |
| `/systems`, `/systems/:id` | Systems | Cards + drill-down |
| `/workspace` | Workspace | Tasks + runs: execute/session/dismiss/PR |
| `/tasks` | Tasks | Task list with status/project filters |
| `/runs` | Runs | Full run pipeline with parent/child aggregation |
| `/activity` | Activity | Unified jobs+runs timeline, SSE live status |
| `/logs` | Logs | Tail `~/.shadow/daemon.log` with level filter |
| `/usage` | Usage | Token usage by period and model |
| `/digests` | Digests | Daily/weekly/brag with navigation |
| `/events` | (redirects to /activity) | — |
| `/guide` | Guide | Tabbed reference: overview, concepts, CLI, MCP tools, jobs |

**Dev**: `npm run dashboard:dev` → Vite on :5173, proxies API to :3700. **Build**: `npm run dashboard:build` → outputs to `src/web/dashboard/dist/`, served by daemon at :3700 via `server.ts` (which checks this path first, then falls back to `src/web/public/index.html` for legacy).

## Database Schema

28 base tables + FTS5 / vec0 virtual tables (SQLite, WAL mode,
busy_timeout=5000ms). Source of truth: `src/storage/migrations.ts`.

| Table | Purpose | Key columns |
|-------|---------|-------------|
| `schema_migrations` | Applied migration versions | version, applied_at |
| `repos` | Tracked repos | name, path (unique), default_branch, test/lint/build commands, last_fetched_at |
| `projects` | Groups of repos+systems | kind (long-term/sprint/task), status, repo_ids_json, system_ids_json |
| `user_profile` | Single-row profile | bond_axes_json, bond_tier (1-8), bond_reset_at, proactivity_level, focus_mode |
| `chronicle_entries` | Immutable narrative (v49) | kind ('tier_lore'\|'milestone'), tier, milestone_key, body_md, model |
| `unlockables` | Tier-gated content slots (v49) | tier_required, kind, title, description, payload_json, unlocked |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andresgomezfrr/shadow](https://github.com/andresgomezfrr/shadow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
