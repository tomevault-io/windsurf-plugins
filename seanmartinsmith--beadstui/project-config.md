---
trigger: always_on
description: Project-canonical guidelines for AI agents working in this Go codebase.
---

# AGENTS.md - beadstui

Project-canonical guidelines for AI agents working in this Go codebase.
Root `CLAUDE.md` imports this file via `@AGENTS.md`; it loads on every
session, so keep it tight.

## Core Rules

1. **No file deletion** without explicit written permission. No exceptions.
2. **No destructive git** (`git reset --hard`, `git clean -fd`, `rm -rf`) without explicit approval. Use `git stash`/`git diff` first.
3. **No script-based code changes.** Make all edits manually or via parallel subagents.
4. **No file proliferation.** Edit existing files in place. No `_v2.go` / `_improved.go` variants. New files only for genuinely new functionality.
5. **Default branch is `main`.** Never reference `master` in code or docs.
6. **No backwards compatibility shims.** Early development, no users. Fix code directly.
7. **Verify after changes:** `go build ./...` and `go vet ./...` after any code change.
8. **bd writes via bash command line corrupt non-ASCII** (Windows bash routes through cp1252). For ANY non-ASCII content (em-dashes, smart quotes, Unicode), use `bd close --reason-file <path>`, `bd update --body-file <path>`, `bd create --body-file <path>`, `bd comments add -f <path>`. Inline strings via `--reason=`/`-d` corrupt em-dashes to `â€"`. Default to ASCII (hyphens, straight quotes) when writing inline; switch to file-based for anything richer.

## Project Identity

- **Project**: beadstui (fork of beads_viewer, retargeted to upstream beads/Dolt)
- **Binary**: `bt`
- **Module**: `github.com/seanmartinsmith/beadstui`
- **Language**: Go 1.25+ (check `go.mod`)
- **TUI framework**: Charm Bracelet v2 (Bubble Tea, Lipgloss, Bubbles, Glamour)
- **Data backend**: Dolt (MySQL protocol). See [docs/design/beads-data-layer.md](docs/design/beads-data-layer.md) before touching the data layer, correlations, sprints, session columns, or git-history-derived features.

## Key Directories

```
cmd/bt/              # CLI entry point (cobra)
pkg/ui/              # Bubble Tea model, update loop, views
pkg/analysis/        # Graph metrics, triage, planning
pkg/search/          # Hybrid search: hash-based semantic embeddings + lexical boost; custom .bvvi vector index under .bt/semantic/
pkg/model/           # Core data types
pkg/view/            # CompactIssue projections (robot output)
pkg/loader/          # JSONL parsing, bead loading
pkg/export/          # Static site export
pkg/agents/          # Agent detection (filename pinned by agents.AgentsFileName constant)
pkg/correlation/     # Bead-to-commit correlation
pkg/watcher/         # Filesystem watching, daemon mode
internal/datasource/ # Data loading (Dolt, JSONL fallback)
internal/doltctl/    # Dolt server lifecycle
docs/                # See docs/README.md for layout map and decision tree
tests/               # Cross-package integration / E2E tests
```

## Build & Test

```bash
go build ./...          # Build all
go test ./...           # All tests
go test ./... -race     # Race detector
go vet ./...            # Static analysis
go install ./cmd/bt/    # Install binary (after a change lands; bt is invoked from PATH)
```

`go install` is a full rebuild that pegs every core (plus Defender scanning the
output) on a machine that usually has several sessions running. Before
installing, compare `go version -m ~/go/bin/bt.exe | grep vcs.revision` with
`git rev-parse HEAD` and skip it if the binary is already current. With other
sessions active, ask sms or wait for a quiet window. Subagents never run
`go install`; the dispatcher installs once, after integrating their work.

## Scratch Conventions

| What | Where | Tracked? |
|---|---|---|
| Beads-context scratch (descriptions, close reasons, comments, audit findings) | `.beads/tmp/` | gitignored |
| General temp / non-beads scratch | `_tmp/` | gitignored |
| bt runtime caches (semantic index, baselines) | `.bt/` | partially gitignored |
| Per-agent worktrees | `.claude/worktrees/` | gitignored |

`.beads/tmp/` is for content that becomes part of bead state (drafts loaded with `--body-file`, comment text via `bd comments add -f`). `_tmp/` is everything else.

## Key Design Constraints

- **Two-phase analysis**: Phase 1 (degree, topo sort, density) is instant. Phase 2 (PageRank, betweenness, HITS, eigenvector, cycles) runs async with 500ms timeout - check `status` flags in output.
- **Robot-first API**: All `bt robot <subcmd>` invocations emit deterministic JSON to stdout. Human TUI is secondary.
- **Elm architecture TUI** via bubbletea - all state transitions are message-based.
- **No raw prints in production** - TUI through lipgloss; robot mode outputs JSON to stdout; errors to stderr.
- **Error wrapping**: `fmt.Errorf("context: %w", err)` always.
- **Division safety**: Guard against divide-by-zero before computing averages/ratios.
- **Nil checks**: Check before dereferencing pointers, especially in graph traversal.
- **Browser safety**: All browser-opening functions gated by `BT_NO_BROWSER` / `BT_TEST_MODE` env vars.
- **Concurrency**: `sync.RWMutex` for shared state; capture channels before unlock to avoid races.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [seanmartinsmith/beadstui](https://github.com/seanmartinsmith/beadstui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
