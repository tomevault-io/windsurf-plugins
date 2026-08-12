---
trigger: always_on
description: This skill ships **only** in the binary. The Claude Code plugin (`plugin/`,
---

# Rekal CLI

## Soul

Before making any design decision, read `SOUL.md`. It defines the two problems Rekal exists to solve and the seven beliefs that guide every choice. If a decision conflicts with the soul, the decision is wrong.

When working on a problem, consult Rekal's own memories first:

```bash
rekal "<describe the problem>"
```

The prior context for what you're working on may already exist.

## Standing Rules

- Keep this file up to date. Any change to commands, packages, files, or behavior must be reflected here. Update `--help` text when command behavior changes. Update `docs/spec/command/` when a command spec changes. Stale docs are worse than no docs.
- Consult `SOUL.md` before design decisions. Consult `rekal` before starting work on a problem.

## Architecture

Single binary. Everything embedded — CLI, database engine, embedding model, compression dictionary.

- CLI: Cobra (`github.com/spf13/cobra`)
- Storage: DuckDB via `github.com/marcboeker/go-duckdb` (`database/sql` interface)
- Compression: zstd via `github.com/klauspost/compress` with preset dictionary
- IDs: ULID via `github.com/oklog/ulid/v2`
- Embeddings: LSA (gonum) + Nomic (platform-specific builds)
- Build: mise, go modules
- Lint: golangci-lint v2 (2.8.0)
- Language: Go 1.25.6

Two databases in `.rekal/`:
- `data.db` — immutable source of truth. Append-only. Pushed to git.
- `index.db` — local derived index. Rebuilt from data.db. Never pushed.

This split is a direct consequence of the soul: thin on the wire, rich on the machine.

The `.rekal/` store lives in the repository's **main worktree**; every linked
git worktree resolves to that one shared store via `gitx.MainWorktreeRoot`
(a no-op for non-worktree repos, so existing installs need no migration). All
store-path helpers (`db.StoreDir`, `cli.RekalDir`, and the `Open*`/`*Path`
functions) funnel through it; git-state helpers (`HeadSHA`/`CurrentBranch`) and
session discovery keep using the invoking worktree. The no-op is enforced by
identity, not by string: git reports worktree paths with symlinks resolved, so
a repo reached through a symlinked directory (macOS `/var` → `/private/var`,
or any symlinked project dir) would otherwise come back under a different name
for the same place and move `.rekal/` off the store that already exists —
`preferCallerPath` stats both and hands the caller's own spelling back.

## Key Directories

### Commands (`cmd/rekal/`)

- `main.go`: Entry point

### Core CLI (`cmd/rekal/cli/`)

- `root.go`: Root command (recall is the default) + command registration.
  A query whose first word matches a registered command name (`log`, `push`,
  `sync`, …) is ordinary cobra dispatch, not search — `rekal -- <query>`
  forces root-level recall past the collision (`--` stops cobra resolving
  the next word as a subcommand), documented in `--help` and the `Example`
  block rather than left for the user to discover. Zero-arg subcommands
  (`log`, `version`, `sync`, `push`, `embed`, `index`, `checkpoint`, `clean`,
  `init`) carry `Args: rejectExtraArgs` (preconditions.go) so a query that
  silently collided with one of them — `rekal log recent commits about the
  ledger` used to dump the plain `log` output at exit 0, discarding
  everything after the matched word — now errors and names the `--` escape
  hatch instead of answering a different question with no sign anything was
  wrong. `-A/--actor` is validated (`validateActorFilter`): an unrecognized
  value used to silently match zero sessions, indistinguishable from a
  query that legitimately has no hits for a real `human`/`agent` value.
  `-e/--explain` without `-j/--json` now warns to stderr — the enrichment
  only ever reached the JSON payload, so passing `-e` alone silently
  computed and discarded it.
- `recall.go`: Recall command orchestration — open/migrate/auto-rebuild the
  index DB, refresh the knowledge layer (watermark-gated), call the `search`
  package. Two self-healing paths, both best-effort: an **empty** index is
  rebuilt inline (`IsIndexPopulated`), and an index whose vectors are **behind
  this binary** spawns the same background `rekal embed` that index/sync use,
  rather than only printing a warning — otherwise an upgrade leaves the semantic
  layer dark on every repo the user owns until they happen to read a warning.
  Staleness is a comparison, not a list: `index_state.embed_backend` records
  which backend produced the vectors (written beside `embed_model` by
  `recordEmbedProvenance`), and `staleEmbeddedVectors` treats an
  **embedded**-backend index whose model id is not `nomic.ModelName` as an
  upgrade to finish. So a future id bump needs no registration — the previous
  hand-maintained set could be forgotten, and forgetting it told the user their
  embedding config was missing. `legacyEmbeddedModels` is a **closed** set for
  stores predating that column; do not add to it. An **http**-backend mismatch
  is deliberately *not* stale: it means the embedding config was removed, and
  re-embedding locally would silently replace the store's vectors with a
  different model's. `rekal embed` holds its own lock, so concurrent recalls
  converge on one worker. Skipped under `session.BenchEnv`. First runs `maybeRefreshStaleSkill` (init.go): the agent enters here

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rekal-dev/rekal-cli](https://github.com/rekal-dev/rekal-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
