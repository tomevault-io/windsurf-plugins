---
trigger: always_on
description: **Generated:** 2026-02-08 16:13:50
---

# MAMA PROJECT KNOWLEDGE BASE

**Generated:** 2026-02-08 16:13:50  
**Commit:** 254557e  
**Branch:** refactor/mcp-server-core-dedup

---

## OVERVIEW

MAMA (Memory-Augmented MCP Assistant) — Contract-first memory system for Claude. Tracks WHY you decided, not just WHAT you chose. Prevents vibe coding breakage across sessions. Monorepo with 5 packages: MCP server (npm), Claude Code plugin (marketplace), MAMA OS standalone agent (npm), shared core (npm), MemoryBench (internal).

**Stack:** JavaScript (MCP/plugin/legacy core), TypeScript (standalone + newer core surfaces), pnpm workspaces, Vitest, SQLite + pure-TS cosine similarity, Transformers.js (local embeddings), GitHub Actions

---

## STRUCTURE

```
MAMA/
├── packages/
│   ├── mama-core/                  # Shared foundation (embeddings, db, memory API, context compile)
│   ├── mcp-server/                 # MCP server for Claude Desktop/Code (4 tools: save/search/update/checkpoint)
│   ├── claude-code-plugin/         # Claude Code plugin (commands + hooks + local mama-core copies)
│   ├── standalone/                 # MAMA OS agent (Discord/Slack/Telegram, multi-agent swarm, CLI, web UI)
│   └── memorybench/                # Memory retrieval benchmarking framework (bun, internal)
├── docs/                           # User-facing documentation (Diátaxis framework)
├── .mama/                          # Project identity (SOUL.md, IDENTITY.md, config.json)
├── .sisyphus/                      # Internal planning artifacts (drafts/, plans/)
├── .docs/                          # Development docs (PRDs, tech specs, epics, stories)
├── .claude-plugin/                 # Local dev marketplace config
├── scripts/                        # Build utilities (verify-install.js, sync-check.js)
├── .husky/                         # Git hooks (pre-commit: lint-staged + gitleaks + typecheck + tests)
└── CLAUDE.md                       # Claude Code guidance (CRITICAL: read this before editing)
```

---

## WHERE TO LOOK

| Task                          | Location                                                                                               | Notes                                                              |
| ----------------------------- | ------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------ |
| **Add memory feature**        | `packages/mama-core/src/mama-api.js`                                                                   | High-level API (2,615 lines — SPLIT CANDIDATE)                     |
| **Modify context compile**    | `packages/mama-core/src/context-compile/` + `packages/standalone/src/agent/context-compile-service.ts` | Scoped evidence packets and trusted `context_packet_id` provenance |
| **Add MCP tool**              | `packages/mcp-server/src/tools/`                                                                       | All tools use `mama-core/mama-api`                                 |
| **Modify embeddings**         | `packages/mama-core/src/embeddings.js`                                                                 | HTTP client + local Transformers.js fallback                       |
| **Modify database**           | `packages/mama-core/src/db-manager.js` + `src/db/migrations/`                                          | SQLite + pure-TS cosine similarity, migrations required            |
| **Add Claude Code command**   | `packages/claude-code-plugin/commands/*.md`                                                            | Markdown-based command definitions                                 |
| **Modify hooks**              | `packages/claude-code-plugin/scripts/*.js`                                                             | Hook scripts (must complete <1800ms)                               |
| **Add gateway integration**   | `packages/standalone/src/gateways/*.ts`                                                                | Discord, Slack, Telegram handlers                                  |
| **Modify multi-agent**        | `packages/standalone/src/multi-agent/swarm/`                                                           | Wave-based orchestration (5 waves, tier-based access)              |
| **Fix reuse-first violation** | Check `packages/mcp-server/src/mama/` FIRST                                                            | CRITICAL: 70% of features already exist here                       |
| **Run all tests**             | `pnpm test` (root)                                                                                     | Single-fork pool required (ONNX/V8 locking)                        |
| **Build all packages**        | `pnpm build` (root)                                                                                    | TypeScript compile for mama-core and standalone                    |
| **Lint + format**             | `pnpm lint:fix && pnpm format`                                                                         | ESLint + Prettier auto-fix                                         |

---

## CONVENTIONS

### **Language Split**

- **JavaScript:** mcp-server, claude-code-plugin, and legacy mama-core modules
- **TypeScript:** standalone and newer mama-core surfaces such as `context-compile`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jungjaehoon-lifegamez/MAMA](https://github.com/jungjaehoon-lifegamez/MAMA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
