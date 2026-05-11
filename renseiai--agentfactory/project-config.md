---
trigger: always_on
description: Multi-agent fleet management for coding agents. This is a pnpm monorepo using Turborepo.
---

# AgentFactory Monorepo

Multi-agent fleet management for coding agents. This is a pnpm monorepo using Turborepo.

> **Migrating from the legacy Node CLI?** See [`docs/migration-from-legacy-cli.md`](./docs/migration-from-legacy-cli.md) for the full `@renseiai/agentfactory-cli` → Go `af` binary mapping.

## Architecture

Authoritative architecture lives in `../rensei-architecture/` (remote: https://github.com/RenseiAI/rensei-architecture, private). Read in this order:

1. `001-layered-execution-model.md` — canonical synthesis. Always first.
2. The reference doc(s) for whichever layer you are working on (`002`–`008`, `011`, `013`–`016`).
3. Any open ADRs that touch your work (`ADR-*.md`).

If this project's docs conflict with `../rensei-architecture/`, the corpus wins. Either update this project's docs to align, or open an ADR to amend the corpus (do NOT amend the corpus during implementation — post a `migration:needs-spec-decision` comment on the Linear issue and continue with adjacent work).

## Project Structure

| Package | Path | Purpose |
|---------|------|---------|
| `@renseiai/agentfactory` | `packages/core` | Orchestrator, providers, crash recovery, deployment checker |
| `@renseiai/plugin-linear` | `packages/linear` | Linear SDK client, agent sessions, webhook types |
| `@renseiai/agentfactory-server` | `packages/server` | Work queue server for webhook-driven execution |
| `@renseiai/agentfactory-nextjs` | `packages/nextjs` | Next.js webhook handlers and middleware |
| `@renseiai/agentfactory-dashboard` | `packages/dashboard` | Fleet management dashboard UI |
| `@renseiai/agentfactory-mcp-server` | `packages/mcp-server` | MCP server exposing fleet capabilities to external clients |
| `@renseiai/agentfactory-code-intelligence` | `packages/code-intelligence` | Tree-sitter AST parsing, BM25 search, incremental indexing |
| `@renseiai/create-agentfactory-app` | `packages/create-app` | Project scaffolding CLI |
| `@renseiai/agentfactory-cli` | `packages/cli` | Orchestrator, worker, Linear CLI, and admin entry points |

## Code Intelligence (Optional)

`@renseiai/agentfactory-code-intelligence` is an **optional dependency** of the CLI. When installed, agents running via the Claude provider receive 6 in-process MCP tools:

| Tool | Purpose |
|------|---------|
| `af_code_get_repo_map` | PageRank-ranked map of the most important files |
| `af_code_search_symbols` | Find function/class/type definitions by name |
| `af_code_search_code` | BM25 keyword search with code-aware tokenization |
| `af_code_check_duplicate` | Exact + near-duplicate detection before writing code |
| `af_code_find_type_usages` | Find all switch/case, mapping objects, and usage sites for a type |
| `af_code_validate_cross_deps` | Check cross-package imports have package.json dependency entries |

**Graceful degradation:** If the package is not installed, the CLI starts normally — agents simply won't have `af_code_*` tools available. The `{{> partials/code-intelligence-instructions}}` partial provides in-process tool instructions when `useToolPlugins` is true, and CLI fallback instructions otherwise.

### Code Intelligence CLI

The same functionality is also available as CLI commands via `pnpm af-code`, enabling Task sub-agents and non-MCP contexts to use code intelligence:

```bash
# Get PageRank-ranked repository map
pnpm af-code get-repo-map [--max-files 50] [--file-patterns "*.ts,src/**"]

# Search for symbol definitions (functions, classes, types)
pnpm af-code search-symbols "<query>" [--max-results 20] [--kinds "function,class"] [--file-pattern "*.ts"]

# BM25 keyword search with code-aware tokenization
pnpm af-code search-code "<query>" [--max-results 20] [--language typescript]

# Check for duplicate code before writing
pnpm af-code check-duplicate --content "<code>"
pnpm af-code check-duplicate --content-file /tmp/snippet.ts

# Find all switch/case, mapping objects, and usage sites for a type
# Use before adding new members to a union type to identify all files needing updates
pnpm af-code find-type-usages "AgentWorkType" [--max-results 50]

# Validate cross-package imports have package.json dependency declarations
pnpm af-code validate-cross-deps [packages/linear]
```

All commands output JSON. First invocation builds the index (~5-10s); subsequent calls reuse the persisted index.

**Optional env vars for enhanced search:**
- `VOYAGE_AI_API_KEY` — Enables semantic vector embeddings (hybrid BM25 + vector mode)
- `COHERE_API_KEY` — Enables cross-encoder reranking for more precise result ordering

Without these keys, agents still get full BM25 keyword search, symbol search, repo maps, and dedup.

**Index cache:** The incremental indexer persists to `.agentfactory/code-index/` (add to `.gitignore`). First run builds the index; subsequent runs re-index only changed files via Merkle tree diffing.

## Linear CLI (CRITICAL)

**Use `pnpm af-linear` for ALL Linear operations. Do NOT use Linear MCP tools.**

The Linear CLI wraps the `@renseiai/plugin-linear` SDK and outputs JSON to stdout. All agents must use this CLI instead of MCP tools for Linear interactions.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RenseiAI/agentfactory](https://github.com/RenseiAI/agentfactory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
