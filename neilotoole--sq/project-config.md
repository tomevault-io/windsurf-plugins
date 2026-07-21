---
trigger: always_on
description: Guidance for AI coding assistants (Claude Code, Copilot, Cursor, Codex, etc.) and
---

# AGENTS.md

Guidance for AI coding assistants (Claude Code, Copilot, Cursor, Codex, etc.) and
human contributors working in this repo.

[`CLAUDE.md`](./CLAUDE.md) is the Claude Code entry point; it points here for
all shared rules.

> [!IMPORTANT]
> Before making a significant change, review [`docs/README.md`](./docs/README.md)
> and the contributor docs it links to (architecture, drivers, grammar, workflows,
> Sakila, releasing). Read the ones relevant to your task before you start, and
> if your change makes any of them inaccurate, update them in the same change.

## About `sq`

`sq` is a command-line data wrangler providing jq-style access to structured
data sources (SQL databases like Postgres, MySQL, SQLite, SQL Server,
ClickHouse, Oracle, DuckDB; and document formats like CSV, JSON, Excel). User
docs live at
[sq.io](https://sq.io).

## Key documents

Before making non-trivial changes, read the document most relevant to your
task:

- [`README.md`](./README.md): project overview and user-facing intro.
- [`CONTRIBUTING.md`](./CONTRIBUTING.md): full contributor guide (tooling,
  `Makefile` usage, driver implementation patterns, test handles,
  `CHANGELOG.md` format).
- [`docs/README.md`](./docs/README.md): index of the contributor docs under
  [`docs/`](./docs).
- [`docs/ARCHITECTURE.md`](./docs/ARCHITECTURE.md): Mermaid ERD of core types
  (`Source`, `Driver`, `Grip`, `Registry`, `RecordWriter`, etc.).
- [`docs/DRIVERS.md`](./docs/DRIVERS.md): driver development guide (package
  structure, type mapping, the driver ship checklist).
- [`docs/GRAMMAR.md`](./docs/GRAMMAR.md): SLQ query language grammar guide
  (companion to [`grammar/SLQ.g4`](./grammar/SLQ.g4)).
- [`docs/WORKFLOW.md`](./docs/WORKFLOW.md): development and CI workflows (local
  `Makefile` loop and the GitHub Actions pipelines).
- [`docs/SAKILA.md`](./docs/SAKILA.md): the Sakila test dataset (`sakiladb`
  images, embedded vs external sources, engine matrix, repo-wide usage).
- [sq.io](https://sq.io): end-user documentation for commands and query
  syntax.

## Common commands

This project uses a `Makefile` as its canonical developer entry point (see
[`CONTRIBUTING.md`](./CONTRIBUTING.md#general-advice) for why).

`make help` (the default target) lists every target with a one-line
description; each target is documented inline in the `Makefile`.

```bash
make help        # list all targets with descriptions (default target)
make init        # one-time clone setup: install deps + activate git hooks
make deps        # install dev deps (bun packages + Go modules)
make all         # gen + fmt + lint + test + build + install
make test        # run all tests (may require Docker for SQL driver tests)
make test-short  # skip long-running / container-backed tests
make fmt         # goimports-reviser (Go imports) + dprint fmt (everything else)
make fmt-check   # dprint check (read-only; verify formatting)
make lint        # golangci-lint + shellcheck + dprint check + biome (site JS)
make build       # build binary to dist/sq
```

Run `make init` once after cloning: it installs dependencies and activates the
repo's git hooks (`.githooks`), including a `pre-commit` hook that runs
`dprint check` on staged files so a formatting slip is caught locally instead
of failing the `Format` CI job. Bypass the hook for one commit with
`git commit --no-verify`.

Driver integration tests for Postgres, MySQL, SQL Server, and ClickHouse
require the `sakiladb/*` Docker images to be reachable. Use `make test-short`
or `go test -short ./...` to skip them.

## Conventions

### Go linting

Run `make lint` after any change to `*.go` files. Fix all reported issues
before committing. Common lint categories:

- `godot`: comments must end with a period.
- `unused`: unused variables, constants, functions.

Go formatting (gofumpt rules + import ordering) is handled by `make fmt`, not
golangci-lint: `dprint` runs the gofumpt plugin (`modulePath` + `extraRules`)
and `goimports-reviser` orders imports. Run `make fmt` before `make lint`.

Don't wait to be asked; treat `make lint` as part of "done".

### Testing

Prefer `github.com/stretchr/testify` for assertions, and prefer `require`
over `assert`:

- `require.*`: fails fast, stopping the test on first failure. Default
  choice.
- `assert.*`: continues after failure. Use only when you genuinely want to
  report multiple independent failures in one run.

```go
import (
    "testing"

    "github.com/stretchr/testify/require"
)

func TestExample(t *testing.T) {
    result := someFunction()
    require.NotNil(t, result)
    require.Equal(t, expected, result)
}
```

Integration tests that need a real database should call [`tu.SkipShort(t, true)`](./testh/tu/skip.go)
so they're skipped under `go test -short`. See
[`docs/DRIVERS.md`](./docs/DRIVERS.md#test-handles) for driver test handle
conventions.

### Error handling

Use [`libsq/core/errz`](./libsq/core/errz) for every error produced inside
sq. `errz` wrappers attach a stack trace at the call site, which the CLI's
debug output and `%+v` formatting rely on (errz errors implement
`fmt.Formatter`). `errors.Is` / `errors.As` continue to work because `errz`
exposes `Unwrap`.

| Situation                                         | Use                            |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [neilotoole/sq](https://github.com/neilotoole/sq) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
