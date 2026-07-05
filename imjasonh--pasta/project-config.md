---
trigger: always_on
description: Notes for future coding agents (and humans). The user-facing pitch lives
---

# Working in this repo

Notes for future coding agents (and humans). The user-facing pitch lives
in [README.md](./README.md); this file is the maintenance / navigation
side.

## Workflow rules

NEVER `git commit` or `git push` without Jason's explicit approval in
the current turn. "Make this change", "fix that", or "do these things"
is **not** approval to commit. He will say "commit", "commit and
push", or "ship it" when he wants a commit. Approval given earlier in
the conversation does **not** carry forward — every commit needs its
own go-ahead. When work is in a committable state, stop and report
it; let him decide whether to commit.

## Pre-commit hook

`scripts/pre-commit.sh` runs the same checks as CI (gofmt, go vet,
go build, go test). It MUST be installed as the local pre-commit
hook before any work in this repo. If `.git/hooks/pre-commit` is
missing or doesn't point at the script, install it before doing
anything else:

```
ln -sf ../../scripts/pre-commit.sh .git/hooks/pre-commit
```

Then verify with `ls -l .git/hooks/pre-commit`. The hook runs
automatically on every `git commit`.

NEVER skip the hook. Do not pass `--no-verify` to `git commit`. Do
not set `core.hooksPath` to bypass it. Do not edit / delete /
chmod -x the script to dodge a check. If a check fails, fix the
underlying issue; don't reach for the bypass.

If the hook is somehow not running (the developer cloned without
running setup), reinstall it via the symlink above before
committing — re-installing is always preferable to skipping.

## Running everything

```
go test ./...
```

The top-level `pasta_test.go` walks `analyzers/*/` and `testdata/*/` and
runs each directory via `internal/runner.TestDir`. There are no per-analyzer
Go test files — `go test ./...` is the whole verification surface.

The CLI also has a `test` mode that does the same thing on a single
directory:

```
go run ./cmd/pasta test analyzers/go_iferr
```

Useful when you're iterating on one analyzer and want a tighter loop
than `go test`.

## Layout

| Path | What it is |
|------|------------|
| `internal/dsl/`               | Go structs mirroring the CUE schema. `dsl.Arg` is a sum of `string` and `[]string` for predicate args. |
| `internal/loader/`            | CUE loader. Embeds the built-in `github.com/imjasonh/pasta` module under `internal/loader/cuemod/`, and vendors any remote modules declared in the rule directory's `pasta.cue` into the same overlay. |
| `internal/loader/cuemod/`     | The embedded built-in CUE module: `schema/`, `lang/<name>/`, `patterns/<name>/`. |
| `internal/remote/`            | Remote rule imports: `pasta.cue` manifest + `pasta.lock` lockfile, git-based fetcher, on-disk cache under `$XDG_CACHE_HOME/pasta/modules/`. Flat deps only — a remote module declaring its own remote imports is rejected. |
| `internal/lang/`              | Runtime language registry. `grammars.go` is the only Go-side language code (maps grammar name → tree-sitter `GetLanguage`). |
| `internal/tsutil/`            | gotreesitter `Node` wrapper that carries source bytes + language + file-id, so callers don't have to thread them. |
| `internal/match/`             | Pattern matcher: node unions, fields, adjacent windows, preceding, predicates (positional), checks (named). |
| `internal/factstore/`         | Per-run fact store with dual indexing — by (kind, file-id, byte-range) and by (kind, identifier-text). The by-name index is file-agnostic so facts propagate across files in a multi-file group. |
| `internal/effect/`            | Compiles edits to byte-range ops, handles `@capture` interpolation, comment preservation, and `trim_start`/`trim_end`. |
| `internal/apply/`             | Applies ops to source bytes with conflict detection. |
| `internal/engine/`            | Top-level orchestrator. SCC scheduler with fixpoint groups for cyclic rule deps. `Run` is the single-file entry point; `RunGroup` runs a set of files with a shared fact store. |
| `internal/runner/`            | Programmatic API used by both the CLI and Go tests. `LoadRules`, `RunFile`, `RunGroup`, `TestDir`. |
| `analyzers/<name>/`      | A shipped analyzer: a `<name>.cue` rule + `testdata/` (sources and `.golden` files). |
| `testdata/<name>/`       | Extension/integration demos (e.g. `notgo_alias` showing user-supplied language modules). |
| `cmd/pasta/`             | CLI. |
| `pasta_test.go`          | One root test that exercises every directory under `analyzers/` and `testdata/`. |

## Adding an analyzer

Mechanically: `mkdir analyzers/<name>/` and create:

```
analyzers/<name>/
  <name>.cue            # imports github.com/imjasonh/pasta/{schema,lang/...,patterns/...}
  testdata/
    a.<ext>             # source with `// want "regex"` markers
    a.<ext>.golden      # optional: expected output after -fix
    multi_pkg/          # optional: subdir = multi-file analysis group
      api.<ext>
      caller.<ext>
```

Naming convention:
- Single-language rules: `<lang>_<name>` (e.g. `go_iferr`, `python_taint`).
- Cross-language rules: bare name (e.g. `todo_format`, `hardcoded_credentials`).

Test data:
- Each `// want "regex"` (or `# want`, `-- want`) anchors a diagnostic
  expectation on the same line.
- Use `// want:+1 "regex"` to anchor it on the next line — useful when

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [imjasonh/pasta](https://github.com/imjasonh/pasta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
