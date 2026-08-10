---
trigger: always_on
description: <!-- adeptability:begin id=adept-code-style hash=df432118 -->
---

<!-- adeptability:begin id=adept-code-style hash=df432118 -->
## Go code style and conventions for the adept codebase — formatting, linters, error wrapping with sentinels, the composition-root/no-globals rule, and core invariants. Apply when writing or reviewing Go in this repo. (matches: **/*.go)


# adept code style

Conventions for Go in `github.com/itaywol/adeptability`. CI enforces formatting and lint;
the rest is reviewed.

## Formatting & lint (enforced)

- `gofmt` + `goimports` — tabs for indentation, grouped imports (stdlib / third-party /
  local). CI fails on any unformatted file.
- `golangci-lint run` (`.golangci.yml`): `errcheck` (incl. type assertions), `staticcheck`,
  `govet`, `gocritic`, `revive` (**every exported symbol needs a doc comment**), `errorlint`,
  `nilerr`, `bodyclose`, `prealloc`, `unconvert`, `misspell`, `unused`, `ineffassign`.

## Errors

- Always wrap with context and `%w`: `fmt.Errorf("clone %s: %w", url, err)`.
- Compare with `errors.Is` against the sentinels in `pkg/adept/errors.go`
  (`ErrSkillNotFound`, `ErrMergeConflict`, `ErrBudgetOverflow`, …). **Never** match on error
  strings. Need a new category? Add a sentinel there.
- `errcheck` is strict — handle or explicitly `_ =` an ignored error, and say why if it isn't
  obvious. Don't drop an error that loses data.

## Structure

- **Composition root, no globals.** Concrete implementations are wired behind interfaces into
  `*Deps` in `internal/cli/deps.go`. No package-level mutable state, no `init()` side effects.
  Take dependencies as parameters so code is testable with fakes/mocks.
- **`pkg/adept` is types-only.** Keep it dependency-light and stable; behavior lives in
  `internal/`. (Example: `SkillIDPattern` is a string in `pkg/adept`, compiled in
  `internal/canonical` — don't add a `regexp` import to `pkg/adept`.)
- Doc comments are full sentences starting with the symbol name (`// NewRoot builds …`).

## Readability

- Small, single-purpose functions; prefer **early returns** over deep nesting.
- Name things for what they are; avoid one-letter names outside tight loops/receivers.
- No magic numbers/strings — name on-disk paths and limits as constants (see
  `pkg/adept/constants.go`).
- Comments explain **why**, not what. Keep them in sync with the code — a stale comment is a
  bug.

## Invariants you must not break

1. Identity is `(id, content-hash)` — no version numbers as a sync signal.
2. Secrets never written to disk — API keys come from the environment at call time.
3. Harness models differ (per-skill / single-file / aggregator) — renderers and importers
   must respect each; aggregators parse their own section markers and honor byte budgets.
4. Canonical layout `<root>/skills/<id>/SKILL.md`; the directory name is the authoritative id.
<!-- adeptability:end id=adept-code-style -->

<!-- adeptability:begin id=adept-contributing hash=dfff4849 -->
## How to contribute to the adeptability (adept) Go CLI — build, the required pre-PR gates, conventional commits, and where things live. Apply when changing adept's own source, opening a PR, or adding a harness.


# Contributing to adept

You are working on the `adept` CLI itself (Go 1.25, module
`github.com/itaywol/adeptability`). Read [AGENTS.md](../../AGENTS.md) for architecture and
invariants; this skill is the operational checklist.

## Build & run

```bash
go build ./...
go build -o /tmp/adept ./cmd/adept && /tmp/adept --help
```

## Gates you MUST pass before opening a PR (same as CI)

```bash
gofmt -l .            # must print nothing → fix with: gofmt -w .
go vet ./...
golangci-lint run     # config: .golangci.yml
go test -race ./...
go test -run E2E ./cmd/adept   # end-to-end against a freshly built binary
```

If any gate fails, fix it before pushing — CI runs the identical set on
ubuntu/macos/windows.

## Where things live

| You're changing… | Go there |
| --- | --- |
| public types / interfaces / sentinel errors | `pkg/adept/` (types only — no behavior) |
| canonical field shape | `pkg/adept/` struct tags **and** `pkg/adeptschema/*.schema.json` together |
| a command | `internal/cli/commands_*.go`; wire deps in `internal/cli/deps.go` |
| a harness renderer | `internal/render/<id>/` + golden fixtures in its `testdata/` |
| sync / drift logic | `internal/harness/orchestrator.go` |
| 3-way merge | `internal/merge/` |
| safety scanner | `internal/scan/` |

## Conventional commits (release-please reads these)

`feat:` → minor · `fix:`/`perf:` → patch · `feat!:`/`BREAKING CHANGE:` → major ·
`refactor:`/`docs:`/`chore:`/`test:`/`ci:` → no release. The squash-merge title becomes the
changelog entry, so write it well.

## Conventions

- Wrap errors with context: `fmt.Errorf("doing X: %w", err)`. Match with `errors.Is` against
  the sentinels in `pkg/adept/errors.go` — never on error strings. Add new sentinels there.
- No package-level state, no `init()` side effects: dependencies flow through `*Deps`.
- New behavior ships with tests; bug fixes ship with a regression test. Update golden
  fixtures in the same PR when output changes, and say why in the message.
- Don't hand-edit `CHANGELOG.md`, `.release-please-manifest.json`, or version strings —
  release-please owns them.
<!-- adeptability:end id=adept-contributing -->


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [itaywol/adeptability](https://github.com/itaywol/adeptability) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
