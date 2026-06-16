---
trigger: always_on
description: Git-native notes engine. One substrate, multiple tools. Built on `refs/notes/*` — no shadow branches, no worktrees, no external dependencies beyond git.
---

# AGENTS.md

## What is maitake

Git-native notes engine. One substrate, multiple tools. Built on `refs/notes/*` — no shadow branches, no worktrees, no external dependencies beyond git.

Everything is a note — tickets, reviews, PRs, docs, warnings, ADRs, changelog entries. Domain tools (`mai-pr`, `mai-docs`, `mai-changelog`) consume the substrate through a clean interface. The substrate never imports domain logic.

## Architecture

```
┌─────────────────────────────────────────────────────┐
│  mai CLI (substrate)                                 │
│  create, append, fold, find, search, ls, context     │
│  + zero-logic shortcuts: ticket, warn, adr, artifact │
├─────────────────────────────────────────────────────┤
│  Plugin dispatch (.maitake/plugins.toml)             │
│  mai pr → mai-pr | mai docs → mai-docs | etc.       │
├──────────┬──────────┬──────────┬────────────────────┤
│ mai-pr   │ mai-docs │ mai-     │  your-tool-here    │
│ PRs      │ doc sync │ changelog│  (any binary)      │
│          │ CRDT     │          │                    │
│          │ daemon   │          │                    │
├──────────┴──────────┴──────────┴────────────────────┤
│  pkg/notes   — Engine interface (the substrate)      │
│  pkg/docs    — doc sync, CRDT, frontmatter           │
│  pkg/git     — git plumbing                          │
│  pkg/guard   — PII/secret scanning hooks             │
│  internal/cli — shared CLI helpers                   │
└─────────────────────────────────────────────────────┘
```

### The substrate boundary

`pkg/notes` is the substrate. It exposes the `Engine` interface — 19 methods for creating, appending, querying, and syncing notes. Everything else consumes this interface.

**Enforced rule:** `pkg/notes` must NOT import domain packages (`pkg/crdt`, `pkg/docs`, or any `mai-*` logic). This is checked by:
- `pkg/docs/boundary_test.go` — compile-time test
- `scripts/check-boundary.sh` — CI script

If someone adds `import "pkg/crdt"` to `pkg/notes`, both fail.

### How domain tools connect

The engine has a `PostWriteHook` mechanism. Domain packages register callbacks:

```go
engine, _ := notes.NewEngine(repo)
docs.RegisterAutoSync(engine)  // CRDT init + auto-sync to disk
```

Hooks fire after every `Create` or `Append`. The engine never calls domain code directly.

### Plugin discovery

`mai` dispatches unknown commands to external binaries via `.maitake/plugins.toml`:

```toml
[plugins]
pr = "mai-pr"
docs = "mai-docs"
changelog = "mai-changelog"
```

`mai pr show X` → finds `mai-pr` on PATH → execs `mai-pr show X` with:
- `MAI_REPO_PATH` — repo root
- `MAI_MAITAKE_DIR` — .maitake/ path
- `MAI_JSON` — "1" if --json was passed

No manifest entry = no dispatch. Explicit, not magic.

`mai init` writes the default manifest. Users add third-party tools manually.

### Layer rules

- `pkg/git` talks to git only. No domain concepts.
- `pkg/notes` knows about note format, edges, kinds, event folding, slots, branch-scope. Does NOT know about docs, PRs, CRDTs, or changelogs.
- `pkg/docs` builds on `pkg/notes` + `pkg/crdt`. Owns doc sync, CRDT merge, frontmatter, tombstones.
- `pkg/guard` scans content before any write. Every write path goes through it.
- `cmd/mai` is the substrate CLI — flag parsing, then calls into `pkg/notes`.
- `cmd/mai-pr`, `cmd/mai-docs`, `cmd/mai-changelog` are domain tools — separate binaries, same Go module.
- `internal/cli` has shared helpers (Fatal, PrintJSON, ParseFlags) used by all binaries.

### Hard rules

1. **No god files.** 500 lines is the soft limit. 800 is the hard limit. Split before it grows.
2. **Substrate boundary.** `pkg/notes` never imports domain packages. `scripts/check-boundary.sh` enforces this.
3. **Every public function has a test.** No exceptions.
4. **Errors are values, not panics.** Return `error`. Wrap with context. Never `log.Fatal` in library code.
5. **No `interface{}` or `any` in public APIs.** Type everything.
6. **Guard every write.** All content passes through `pkg/guard` before hitting git. PII/secrets rejected at write time.
7. **Event-sourced.** Immutable creation notes + event stream. No mutable blobs. No merge conflicts.
8. **Notes are append-only.** Changes via events, not mutation.
9. **Performance budget.** Listing/filtering 10,000 notes must complete in <200ms. Build caching/indexing from day 0.
10. **jj support from day 0.** Detect `.jj/`, use change_id edges, handle rewritten commit OIDs.
11. **Domain tools are separate binaries.** New domain logic gets its own `cmd/mai-*` binary, never added to `cmd/mai/`.

## Building a new plugin

To add a new domain tool (e.g. `mai-backlog`):

### 1. Create the binary

```
cmd/mai-backlog/main.go
```

```go
package main

import (
    "github.com/cygnusfear/maitake/internal/cli"
    "github.com/cygnusfear/maitake/pkg/notes"
    "github.com/cygnusfear/maitake/pkg/git"
)

func main() {
    // Read MAI_REPO_PATH from env (set by mai dispatcher)
    dir := os.Getenv("MAI_REPO_PATH")
    repo, _ := git.NewGitRepo(dir)
    engine, _ := notes.NewEngine(repo)

    // Use the Engine interface — never import pkg/git internals
    states, _ := engine.Find(notes.FindOptions{Kind: "ticket", Status: "open"})

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Cygnusfear/maitake](https://github.com/Cygnusfear/maitake) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
