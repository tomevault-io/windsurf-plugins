---
trigger: always_on
description: Routing map for Claude. Points at the rule, skill, agent, or file that owns the answer — does not restate them.
---

# CLAUDE.md

Routing map for Claude. Points at the rule, skill, agent, or file that owns the answer — does not restate them.

## Project at a glance

`remindb` — token-efficient agentic memory database. Single SQLite file, MCP server on top. Go 1.23+. Pre-implementation stage, solo dev. **`dev` is the integration trunk; `main` is a release-marker branch — one squash commit per published release, signed by GitHub's web-flow key via PR from `dev`.** Topic branches (`feat/`, `fix/`, `chore/`, `docs/`) off `dev`, rc tags on `dev`, stable cuts tag `dev` then squash-PR `dev` → `main`; lazy `release/vX.Y` branches handle patches to non-current minors; subject-only signed commits everywhere (release squash on `main` carries the release notes as its body) — see `.claude/rules/git-versioning.md`.

Pipeline: `parser → transformer → emitter → store`. Read side: `query → mcp/tools`. Background: `temperature` ticker decays/notifies.

## Code map

- `pkg/parser/` — file formats → AST nodes
- `pkg/transformer/` — AST → `ContextNode` tree
- `pkg/emitter/` — node tree → store snapshot
- `pkg/store/` — SQLite layer; `queries.go` holds SQL constants, methods live in `node.go` / `snapshot.go` / `search.go` / `temperature.go`
- `pkg/diff/` — snapshot diffing
- `pkg/query/` — search/fetch engine + result formatters (`Format`, `FormatCompact`)
- `pkg/compiler/` — full-workspace compile pipeline
- `pkg/mcp/` — MCP server; tools in `pkg/mcp/tools/` (one file per tool)
- `pkg/temperature/` — decay/boost/cold-set + cold-node notifier
- `cmd/remindb/` — CLI: `serve`, `compile`, `inspect`, `bench`
- `migrations/` — `0001_init.sql`, `0002_*.sql`, applied via embed.FS in `migrations.go`
- `internal/` — bench, contentid, fileext, mcptest, tempfile, testutil, tokens
- `skills/remind/`, `skills/memoize/` — **public** skills shipped to MCP clients: `remind` is the read path + mental model, `memoize` is the write path + Markdown-shape rules (distinct from `.claude/skills/`)
- `plugins/` — per-agent plugin folders (claude-code, gemini-cli, codex, opencode)
- Top-level: `integration_test.go`, `mcp_integration_test.go`, `bench_test.go`

## Where to read first — don't grep, don't ls

| Question | Source |
|---|---|
| End-to-end product story, architecture, benchmarks | `README.md` |
| How clients call the MCP read tools (the contract) | `skills/remind/SKILL.md` |
| How clients author content for MCP write tools (the contract) | `skills/memoize/SKILL.md` |
| Go style, naming, error/log/concurrency idioms | `.claude/rules/go-concise.md` |
| Commit, sign, branch, tag, push, release rules | `.claude/rules/git-versioning.md` |
| MCP tool contract (signature, locking, returns) | `.claude/rules/mcp-tool-conventions.md` |
| `slog` levels, fields, what never logs | `.claude/rules/logging-conventions.md` |
| User & feedback memory across sessions | `.claude/projects/-home-radimsem-personal-projects-remindb/memory/MEMORY.md` (auto-loaded) |

## Tread carefully — pause before implementing

These zones have either an external contract or a silent-drift hazard. Don't change them blind; the linked skill or rule encodes the discipline that prevents the foot-gun.

### MCP tool surface (`pkg/mcp/tools/`, `pkg/mcp/server.go`)

The eight `Memory*` tools are a contract shipped to clients via two public skills: `skills/remind/SKILL.md` (read tools — `MemoryTree`, `MemorySearch`, `MemoryFetch`, `MemoryDelta`, `MemoryHistory`) and `skills/memoize/SKILL.md` (write tools — `MemoryWrite`, `MemorySummarize`, `MemoryCompile`). Renaming, removing, or changing semantics breaks every client and desyncs the relevant public skill. Use the **`add-mcp-tool` skill** for any new/modified tool, follow `.claude/rules/mcp-tool-conventions.md`, and dispatch the **`mcp-surface-reviewer` agent** before merge.

### SQLite schema & migrations (`migrations/`, `pkg/store/`)

Migrations are forward-only, applied at startup, and FTS5 triggers must stay in sync with the `nodes` table. Schema mistakes ship as broken `.db` files in user repos. Use the **`add-store-query` skill** (covers both query-only and migration-bearing changes) and dispatch the **`migration-safety-reviewer` agent** before merge.

### Temperature policy (`pkg/temperature/Config`)

`DecayRate`, `AccessBoost`, `ColdThreshold`, `NotifyThreshold`, `TickInterval` are documented numerically in `skills/remind/SKILL.md` (mental model) and the summarization workflow they trigger lives in `skills/memoize/SKILL.md`. Changing any one shifts search ranking, the cold-set query, *and* the client notification stream — both public skills drift silently. Use the **`tune-temperature-policy` skill**.

### Snapshot atomicity

Each `MemoryWrite` / `MemorySummarize` / `MemoryCompile` call must produce **exactly one** `emitter.Emit` (one snapshot row). Two snapshots per intent fragments the diff trail clients walk via `MemoryDelta`. Detail in `.claude/rules/mcp-tool-conventions.md` §7.

### Read vs. write tool discipline


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [radimsem/remindb](https://github.com/radimsem/remindb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
