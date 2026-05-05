---
trigger: always_on
description: Browser-based UI for Jujutsu (jj) version control. See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for system design, [BACKLOG.md](BACKLOG.md) for planned features.
---

# lightjj

Browser-based UI for Jujutsu (jj) version control. See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for system design, [BACKLOG.md](BACKLOG.md) for planned features.

## Build & Test

```bash
go test ./...                                        # Go tests
go vet ./...                                         # static analysis
cd frontend && pnpm install && pnpm run build        # build frontend
go build ./cmd/lightjj                               # build binary (needs frontend build first)

# Dev mode: two terminals
# 1: go run ./cmd/lightjj --addr localhost:3000 --no-browser
# 2: cd frontend && pnpm run dev
# Vite proxies /api/* to localhost:3000
```

## Project Structure

```
cmd/lightjj/main.go       — CLI entry point, flag parsing, embeds frontend-dist/
internal/
  jj/                     — Command builders + data models (PURE — no I/O, no side effects)
    commands.go            — Functions that return []string args for jj subcommands
    commands_test.go       — Command builder tests
    commit.go              — Commit model with ChangePrefix/CommitPrefix, Immutable, Divergent, WorkingCopies, Mine, AuthorEmail, Timestamp
    commit_test.go         — Commit model tests
    bookmark.go            — Bookmark model + output parsers; ParseBookmarkListOutput/ParseRemoteListOutput take defaultRemote param for sort order
    bookmark_test.go       — Bookmark parser tests
    alias.go               — jj config alias parser
    file_change.go         — FileChange model, FilesTemplate (single-call file stats + conflict info), ParseFilesTemplate
    divergence.go          — DivergenceEntry, Divergence() template builder, ParseDivergence. committer_ts DELIBERATELY absent (structurally inverted for --at-op). splitNonEmpty helper.
    selected_revisions.go  — Multi-revision selection helper
    version.go             — Semver type + ParseSemver + named feature gates (WorkspaceRootTmpl etc)
    workspace_store.go     — Protobuf parser for .jj/repo/workspace_store/index (name→path map). <0.40 fallback — ≥0.40 uses WorkspaceRef.root() in the WorkspaceList template.
    workspace_store_test.go — Parser tests with real binary data
  runner/                  — CommandRunner interface + implementations
    runner.go              — Interface definition (Run, RunWithInput, RunForMutation, StreamCombined, RunRaw, WriteFile)
    local.go               — LocalRunner: exec("jj", args) with configurable Binary. WriteFile does symlink-escape hardening (EvalSymlinks on parent + Lstat leaf) before os.WriteFile.
    ssh.go                 — SSHRunner: wraps jj args in ssh command. WriteFile pipes content via `cd <repo> && cat > <path>` over stdin — no symlink check (same trust boundary as remote shell).
    ssh_test.go            — SSH arg escaping tests
  api/                     — HTTP handlers
    server.go              — Route registration, runMutation, op-id caching, workspace store reader, helpers
    handlers.go            — All endpoint implementations, flag validation
    handlers_test.go       — Handler tests with MockRunner
    integration_test.go    — Integration tests (build-tagged)
    watcher.go             — fsnotify on .jj/repo/op_heads/heads/ + SSE push, periodic util snapshot. SSH mode uses `sshPollLoop` (`PollOpId` — implicit snapshot + op-id in one round trip; no remote deps; CAS against pre-poll cachedOp so a concurrent runMutation advance isn't regressed). `interval <= 0` gates both loops (parity). Stale-WC detection: both loops check `isStaleWCError()` → `setStale(bool)` (staleMu-serialized Swap+broadcast — handler clear can't race a loop set and emit sentinels out-of-order; Swap edges don't re-fire so the race has no self-heal) → `evStaleWC`/`evFreshWC` sentinels ("!"-prefixed on the existing `chan string`) → frontend non-dismissable warning with "Update stale" action. Both handleSnapshot + handleWorkspaceUpdateStale call `clearStale()`. `handleEvents` refreshes cachedOp if empty (read-only session path), emits current stale state unconditionally on connect (both branches — false branch covers SSE-dropped-during-staleness → CLI-recovered → reconnect), per-write WriteDeadline extension (not blanket disable — dead TCP surfaces in ≤85s).
    watcher_test.go        — Broadcaster tests (subscribe/broadcast/drop), sshPollLoop tests via seqRunner (hook field for mid-call injection), setStale edge-only invariant, handleEvents refresh-if-empty table, CAS-guards-concurrent-advance
    tabs.go                — TabManager: per-tab Server + Watcher mounted at /tab/{id}/; TabResolve + TabFactory injection (mode-agnostic validation via closure); openTabs persisted to config.json (tab 0 excluded — fights with -R on next launch); startup restore re-runs resolve() so moved repos log-and-skip

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chronologos/lightjj](https://github.com/chronologos/lightjj) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
