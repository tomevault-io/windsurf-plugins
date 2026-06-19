---
trigger: always_on
description: Browser-based UI for Jujutsu (jj) version control. See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for system design, [docs/FILES.md](docs/FILES.md) for the detailed per-file guide, [BACKLOG.md](BACKLOG.md) for planned features.
---

# lightjj

Browser-based UI for Jujutsu (jj) version control. See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for system design, [docs/FILES.md](docs/FILES.md) for the detailed per-file guide, [BACKLOG.md](BACKLOG.md) for planned features.

## Build & Test

```bash
go test ./...                                        # Go tests
go vet ./...                                         # static analysis
cd frontend && pnpm install && pnpm run build        # build frontend
cd frontend && pnpm run bench                        # diff perf benchmarks (see docs/design-notes/diff-perf-benchmarks.md)
go build -tags embed ./cmd/lightjj                   # build binary (needs frontend build first; no tag = stub)

# Dev mode: two terminals
# 1: go run ./cmd/lightjj --addr localhost:3000 --no-browser
# 2: cd frontend && pnpm run dev
# Vite proxies /api/* to localhost:3000
```

## Project Structure

One line per file; routine `*_test.go` / `*.test.ts` files are omitted (they sit alongside their subjects), shared test infrastructure is listed. **Detailed per-file notes and invariants live in [docs/FILES.md](docs/FILES.md)** — read a file's entry there before modifying the file or adding a new caller of its exports; update the entry (plus the one-liner here) when adding, removing, or renaming a file or when its contract changes.

```
cmd/lightjj/main.go       — CLI entry point, flag parsing, embeds frontend-dist/
cmd/lightjj/frontend_embed.go — go:embed frontend-dist/ handler (-tags embed)
cmd/lightjj/frontend_stub.go — No-embed fallback: static "frontend not bundled" help page
cmd/lightjj/session_file.go — Agent port-discovery session files ($XDG_RUNTIME_DIR/lightjj/sessions/<pid>.json)
cmd/lightjj/session_file_unix.go — verifyOwner uid check on the session dir (unix)
cmd/lightjj/session_file_other.go — verifyOwner no-op stub (non-unix)
cmd/lightjj/api_cmd.go    — lightjj api / lightjj sessions subcommands: loopback HTTP client for agent harnesses
cmd/lightjj/skill_cmd.go  — lightjj skill [install]: prints/installs embedded SKILL.md agent guide
cmd/lightjj/apply_hunks.go — --apply-hunks re-entry for `jj split --tool lightjj-hunks` (writes hunk spec into $right)
internal/
  jj/                     — Command builders + data models (PURE — no I/O, no side effects)
    commands.go            — Functions that return []string args for jj subcommands
    commit.go              — Commit model (ChangePrefix/CommitPrefix, Immutable, Divergent, WorkingCopies, …)
    bookmark.go            — Bookmark model + output parsers
    alias.go               — jj config alias parser
    file_change.go         — FileChange model, FilesTemplate, ParseFilesTemplate
    divergence.go          — DivergenceEntry, Divergence() template builder, ParseDivergence
    selected_revisions.go  — Multi-revision selection helper
    version.go             — Semver type + named jj feature gates + FeatureGates wire registry (→ /api/info features)
    workspace_store.go     — Protobuf parser for .jj/repo/workspace_store/index (<0.40 fallback)
  runner/                  — CommandRunner interface + implementations
    runner.go              — Interface (Run, RunWithInput, RunForMutation, StreamCombined, RunRaw, WriteFile)
    local.go               — LocalRunner: exec("jj", args); WriteFile symlink-escape hardening; resolve rejects forgotten workspaces
    ssh.go                 — SSHRunner: wraps jj args in an ssh command
  api/                     — HTTP handlers
    server.go              — Route registration (route lines ARE the pure-mutation handlers), runMutation, op-id cache (getOpId/setOpId/casOpId), helpers
    handlers.go            — Endpoint implementations, generic mutation[Req] factory, flag validation
    watcher.go             — Op-id watcher: fsnotify + SSE push (local), sshPollLoop (SSH), typed sseEvent broadcasts, shared probeTracker, stale-WC detection
    tabs.go                — TabManager: per-tab Server + Watcher mounted at /tab/{id}/
    config.go              — Server-side JSONC config (hujson); mergeAndWriteConfig single write path for human-edited keys
    config_jsonc.go        — hujson helpers: standardizeJSONC, unmarshalJSONC, patchConfigKeys, removeConfigKeys
    config_template.go     — First-run JSONC template constant
    state.go               — Machine-state store (state.json, plain JSON): openTabs + recentActions, GET/POST /api/state/recent-actions, legacy-key migration
    jsonstore.go           — jsonCollection[T]: generic keyed flat-file JSON store (mutex, merge-on-upsert, stamping, cascade delete, batch) + atomicWriteFile primitive
    annotations.go         — Per-changeId review comments: Annotation type + store config + GET/POST/DELETE handlers
    doc_comments.go        — Doc-mode range-anchored comments: DocComment types + store config + GET/POST/DELETE/batch handlers
    agent_docs.go          — GET /api/agent serves embedded agent_api.md (doc/route drift guard test)
    symbol.go              — rg-backed go-to-definition (GET /api/symbol) via RunRaw rg --json

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chronologos/lightjj](https://github.com/chronologos/lightjj) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
