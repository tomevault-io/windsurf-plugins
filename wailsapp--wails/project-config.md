---
trigger: always_on
description: Wake is a Go-native Taskfile executor embedded in `wailsapp/wails/v3/internal/wake/`. It replaces the external `task` CLI dependency, executing `Taskfile.yml` files directly in Go. Enabled via `WAILS_USE_WAKE=true` env var; falls back to `task` CLI when unset or unsupported features are used.
---

# Wake - Go-native Build System for Wails v3

## Overview

Wake is a Go-native Taskfile executor embedded in `wailsapp/wails/v3/internal/wake/`. It replaces the external `task` CLI dependency, executing `Taskfile.yml` files directly in Go. Enabled via `WAILS_USE_WAKE=true` env var; falls back to `task` CLI when unset or unsupported features are used.

## Architecture

```
wake.go              Entry point: Parse -> Resolve -> DAG -> Execute (serial or parallel)
ast/                 Taskfile AST types + deep Clone() for include isolation
parse/               YAML parsing, include resolution, var/shell expansion, template expansion
resolve/             DAG builder (topological sort, cycle detection), platform filtering
exec/                Task executor, cache (SHA-256 hash + mtime), glob matching with ** support
fallback/            Falls back to external `task` CLI when WAKE disabled or unsupported
override/            Taskfile.local.yml / Taskfile.override.yml merge support
cmds/                Command routing: shell, native Go (go build, npm, etc), frontend ops
platform/            OS/arch detection
```

Build output is rendered through `internal/report` (a leaf contract) and
`internal/report/termui` (the lipgloss renderer). See "Build Reporting" below.

## Execution Flow

1. **Discover** `Taskfile.yml` / `Taskfile.yaml` / `build/Taskfile.yml`
2. **Parse** YAML into AST, recursively resolve includes with cycle detection
3. **Clone** included tasks (`task.Clone()`) to prevent pointer aliasing across namespaces
4. **Populate** built-in vars (`OS`, `ARCH`, `ROOT_DIR`, `TASKFILE_DIR`, etc.)
5. **Layer** local overrides (`override.LoadLocal` + `resolve.MergeTaskfile`): two stacked layers in increasing precedence — `Taskfile.override.*` (committed team layer) then `Taskfile.local.*` (personal layer, wins last); within a layer the first existing extension wins. Local-wins precedence — a same-named task replaces the base task (list fields replace, `env`/`vars` maps merge per-key); local-only tasks are added; top-level vars layer over base.
6. **Filter** platforms (remove non-matching `darwin:`/`linux:`/`windows:` namespaces)
7. **Resolve** vars (shell execution, ref chains, cycle detection)
8. **Expand** templates (`{{.VAR}}`) in task names, dirs, deps, cmds, env
9. **Resolve** dep namespaces (short deps -> fully qualified `prefix:task`)
10. **Build DAG** from target task's dependency tree
11. **Execute** serially (`ex.Execute`) or parallel (`executeParallel` via DAG in-degree)
12. **Cache** results in `.wake/cache.json` (hash + last_run timestamp)

## Parallel Execution

DAG-based: tasks with in-degree 0 grouped into waves, each wave executed via goroutines with `sync.WaitGroup`. Deadlock detection if no ready tasks remain but not all completed. Enabled via `ExecuteOptions.Parallel` or `WAKE_PARALLEL` env var.

## Cache System

- **Location**: `.wake/cache.json` per project
- **Key**: Fully namespaced task name (e.g., `darwin:common:generate:icons`)
- **Hash**: SHA-256 of task name + cmds + env + sources + generates patterns
- **Skip logic** (`ShouldSkip`):
  - Returns `false` if no sources/generates/status (task always runs)
  - Status commands: skip if all exit 0
  - Generates: skip if all files exist
  - Hash: skip if matches cache entry
  - Sources: skip if no source file mtime > cache `last_run`
- **Task Dir**: Sources/generates resolved relative to `task.Dir` (joined with `baseDir`), not `baseDir` directly
- **Glob**: `**` patterns handled via `recursiveMatch`; `exclude:` prefix filters sources

## Native Go Command Auto-Cache (wake-exclusive)

`exec/gocache.go` gives wake caching the external `task` CLI cannot match. For a single-command task that declares no explicit sources/generates/status and whose command is `go build` or `go mod tidy` (post template-expansion), wake derives the cache inputs itself and skips the subprocess when nothing changed:

- **`go mod tidy`**: inputs = `go.mod`, `go.sum`.
- **`go build`**: inputs = module-local non-test `.go` files + `go.mod`/`go.sum` + the resolved `generates` of the task's transitive dependency closure (this captures embedded assets like `frontend/dist`, so a frontend change still forces a re-link). Output existence (`-o` path) is also required for a hit.

Cache entries use the `#go` key suffix so they don't collide with `RecordTask`. The entry hash is the SHA-256 of the *expanded* command, so changing build flags (e.g. dev vs `-tags production`) invalidates correctly. Set `WAKE_DEBUG=1` to log cache misses.

Result (badge example, no-op rebuild): wake ~20ms vs `task` ~316ms (~94% faster) — wake skips both `go build` and `go mod tidy`; `task` always runs them.

**Limitation**: a file embedded directly by the main package that is neither a `.go` file nor produced by a dependency's `generates` won't be tracked. The Wails template routes embedded assets through `frontend/dist` (a dep generate), so the common case is covered.

## Known Gotchas


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wailsapp/wails](https://github.com/wailsapp/wails) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
