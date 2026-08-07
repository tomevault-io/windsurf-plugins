---
trigger: always_on
description: > Last updated by big-pickle on 2026-06-10.
---

# Context for AI Coding Agents

> Last updated by big-pickle on 2026-06-10.

## V2 Foundation (in progress — branch `feat/v2-foundation`)

**Goal:** cut the dominant cost of long single sessions — Anthropic **prompt-cache write/read**, not model output. Evidence: a single call charged $2.95 where $2.84 was a 5-min cache *write* of ~455k tokens; output was only ~3.9k. The MCP cannot control *when/how* the client caches (breakpoints/TTL are client-owned); its only lever is **reducing how many tokens ever enter the resident context window** (the thing re-cached every turn). Every V2 piece serves that one goal.

Three steps:
1. **DONE — compact `repo_summary`.** `BuildRepositorySummaryCompact(ks, budget, module)` in `internal/retrieval/repository_summary.go`: token-budgeted (reuses `parseBudget`), top modules by symbol count + `+N more` rollup, dropped the unbounded `Layers` chain, optional `module` drill-down. Tool gained `budget`+`module` params. Legacy `BuildRepositorySummary`/`Format()` untouched. (50-module synthetic: 1192→216 tokens; capped regardless of repo size.)
2. **DONE (code) — 3 cache-reducing tools** in `internal/mcpserver/tools.go`: `remember(repo_path,key,fact)` (durable fact → `kmemory` `UserNote` + per-repo `session_facts.json`), `stash_context(repo_path,content,label?)` (park large blob out of window → tiny handle; file-backed `internal/stash` at `<repoRoot>/.mycli-fts/stash/`), `recall(repo_path,query,source?,budget?)` (query-scoped read of a stash by handle, or facts; hard-capped via step-1 budgeting). Stores wired into `RepoSession` (`Stash`, `FactsPath`, `RememberFact`, `RecallFacts`). Chosen over compact/summarize/forget because the user requires **no context drop** — stash is lossless (relocates tokens, re-fetchable). All tool outputs stay tiny. New names added to `claude.go` allow-list. Tests: `internal/stash`, `internal/session/repo_session_v2_test.go`.
3. **DONE — `costwise-session` skill (session-awareness).** Teaches the model to keep the session lean (route large content through stash/recall, remember durable facts, prefer narrow retrieval). Single embedded source of truth: `internal/skill/policy.md` (`go:embed`, ~275 tok). Delivered two ways: (a) **automatic/cross-IDE** via `server.WithInstructions(skill.Instructions())` in `internal/mcpserver/server.go` — every MCP client auto-loads it, zero install; (b) **native Claude Code SKILL.md** via `costwise skill {install,uninstall,print}` (`cmd/skill.go`) writing `~/.claude/skills/costwise-session/SKILL.md` (or `.claude/...` with `--local`). `install` writes the skill by default (opt out `--no-skill`); `uninstall` removes it. Other IDEs rely on the instructions field + `skill print` for manual placement. `internal/skill` is standalone (NOT in the Target interface). Tests in `internal/skill/skill_test.go`.

## V3 Enterprise Capabilities (Completed)

1. **Semantic Search via Bluge**: Replaced SQLite FTS with Bluge inverted index for `search_code`, enabling fuzzy matching and BM25 scoring natively.
2. **LSIF Ingestion**: Added `.lsif` support for compiler-verified reference tracing via `find_references`.
3. **Zero-Latency CI/CD Caching**: GitHub Action integration (`costwise-action`) to build and fetch `cache.db` from remote artifacts.
4. **Shared Team Cache**: Remote Postgres adapter supporting `COSTWISE_PG_URL` for shared stash and facts memory.
5. **Policy Engine**: Centralized AST-based architecture checking (`costwise-architecture.yaml`) enforced by the `costwise validate` command and `validate_architecture` MCP tool.

**LANDMINE:** `repo_memory`/`discovery_memory` Init with shared `os.TempDir()` paths (NOT per-repo) — a clobber risk (same class as the shared-index bug). New V2 stores MUST be per-repo (derive from `repoRoot` like `treesitter.NewSymbolDB`/`cache.NewCache`).

**Honest limit:** these tools can't evict content the client already placed in context; they only help when the model *routes new large content through them* — which is what the step-3 skill enforces. MCP server-side state persists across tool calls via a process-global per-repo `SessionCache` (`internal/mcpserver/session_cache.go`).

## CGO Requirement

**CGO is mandatory.** The project depends on:
- `github.com/mattn/go-sqlite3` — `//go:build cgo` constraint
- `github.com/smacker/go-tree-sitter` — C bindings via cgo

Builds with `CGO_ENABLED=0` will fail. Always ensure `CGO_ENABLED=1` (which is the default on most systems with a C compiler).

On Ubuntu/Debian: `sudo apt install gcc libsqlite3-dev`
On macOS: Xcode Command Line Tools (`xcode-select --install`)
On Windows: MinGW-w64 (`mingw-w64`)

## Build & Test
```bash
# Build all packages (CGO must be enabled)
CGO_ENABLED=1 go build ./...

# Run all tests
CGO_ENABLED=1 go test ./...

# Build binary
CGO_ENABLED=1 go build -o costwise ./cmd/costwise/

# Build with version injection (for releases)
go build -ldflags="\
  -X github.com/okyashgajjar/costwise-mcp/cmd.version=v1.0.0 \
  -X github.com/okyashgajjar/costwise-mcp/cmd.commit=$(git rev-parse --short HEAD) \
  -X github.com/okyashgajjar/costwise-mcp/cmd.date=$(date -u +%Y-%m-%dT%H:%M:%SZ)" \
  -o costwise ./cmd/costwise/
```

Default version (no ldflags): `dev`
Injected version example: `v1.0.0` with commit hash and build date

## Release Process


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [okyashgajjar/costwise-mcp](https://github.com/okyashgajjar/costwise-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
