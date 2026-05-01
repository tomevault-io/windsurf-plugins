---
trigger: always_on
description: This file is the source of truth for AI coding assistants working on this repository. Follow it before generating code, tests, or documentation. If a request conflicts with this file, pause and ask.
---

# cavemem — Agent Playbook

This file is the source of truth for AI coding assistants working on this repository. Follow it before generating code, tests, or documentation. If a request conflicts with this file, pause and ask.

## Project identity

cavemem is a cross-agent persistent memory system for coding assistants. It captures observations from editor sessions, compresses prose using the project's deterministic caveman grammar, stores entries in a local SQLite + vector index, and exposes them to agents through a Model Context Protocol (MCP) server and a local web viewer.

The signature property of the project is that **memory is stored compressed**. Every write path runs text through `@cavemem/compress`. Every human-facing read path runs it back through `@cavemem/compress#expand`. Model-facing reads may keep content compressed when the caller requests it.

## Non-negotiable rules

1. **All persisted prose must pass through `packages/compress` before hitting storage.** Writing raw prose to SQLite is a defect. If you add a new write path, it must use `MemoryStore`, which enforces this.
2. **Never compress technical tokens.** Code blocks, inline code, URLs, file paths, shell commands, version numbers, dates, numeric literals, and quoted identifiers are preserved byte-for-byte. The tokenizer in `packages/compress/src/tokenize.ts` is the single authority.
3. **Round-trip tests must pass.** Any change to the compressor, the lexicon, or the tokenizer requires `pnpm --filter @cavemem/compress test` green, including the technical-token preservation suite.
4. **Progressive disclosure in MCP.** `search` and `timeline` return compact results (IDs + snippets). Full observation bodies are only returned by `get_observations(ids[])`. Do not bloat the compact shapes.
5. **Hot-path hooks are fast.** Hook handlers in `packages/hooks` must complete under 150 ms p95. Summarization, embedding, and indexing are handed off to the worker. No network calls in hooks.
6. **Privacy is enforced at the write boundary.** Content inside `<private>…</private>` tags is stripped. Paths matching `settings.excludePatterns` are never read. Neither appears in logs.
7. **Local by default.** Default embedding provider is local (Transformers.js). Remote providers are opt-in via settings. Do not add default network calls.
8. **No silent failures.** Hook and worker errors are logged as structured JSON; user-visible commands surface failures with a non-zero exit code and a short message.
9. **No daemon on the write path.** Hooks write observations synchronously through `MemoryStore.addObservation` — never across a network or HTTP boundary. Hooks may *detach-spawn* the worker to kick off background embedding, but they must never wait on it. If the worker is down, writes still succeed; only the semantic-search side is degraded (BM25 keeps working).

## Architectural rules

- Monorepo with pnpm workspaces. Dependency direction is strictly downward: `apps/*` may depend on `packages/*`; `packages/*` may depend on each other only in the order `config → compress → storage → { core, embedding } → hooks → installers`. (`core` and `embedding` are siblings — both consume `config` and `storage`, neither depends on the other.) No upward or sideways imports that break this order.
- All database I/O goes through `@cavemem/storage`. No other package opens the DB directly.
- Settings access goes through `@cavemem/config`. No direct reads from `~/.cavemem/settings.json` elsewhere.
- All user-visible strings default to the caveman intensity from settings (default `full`).
- Public package exports are listed in each package's `package.json#exports`. Internal files are not imported across package boundaries.

## Layout

```
apps/cli          user-facing binary
apps/worker       local HTTP daemon: read-only viewer + embedding backfill loop
apps/mcp-server   stdio MCP server
packages/config   settings schema, loader, defaults, settingsDocs()
packages/compress compression engine + lexicon
packages/storage  SQLite + FTS5 + vector adapter
packages/core     domain models, MemoryStore facade, Embedder interface
packages/embedding provider factory (local / ollama / openai / none)
packages/hooks    lifecycle hook handlers + worker auto-spawn
packages/installers per-IDE integration modules
viewer            Vite + React read-only UI
hooks-scripts     portable shell stubs that invoke node handlers
docs              architecture + user docs
evals             token-savings and round-trip harness
```

## Development workflow

- `pnpm install` once. Node ≥ 20.
- `pnpm dev` runs the CLI and worker in watch mode against `.cavemem-dev/` in the repo root (isolated data dir).
- The four required gates before merging:
  - `pnpm typecheck`
  - `pnpm lint`
  - `pnpm test`
  - `pnpm build`
- New features require unit tests. Any change that affects MCP contracts requires an integration test via the MCP inspector.
- Every PR touching a package under `packages/*` or `apps/*` needs a changeset entry (`pnpm changeset`).

## End-to-end publish test


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JuliusBrussee/cavemem](https://github.com/JuliusBrussee/cavemem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
