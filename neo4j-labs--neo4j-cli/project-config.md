---
trigger: always_on
description: <!-- BEGIN GENERATED: AGENTS-MD -->
---

<!-- BEGIN GENERATED: AGENTS-MD -->

# AGENTS.md

Learnings and patterns for future agents working on this project.

## Feedback Instructions

TEST COMMANDS: [`make test`]
BUILD COMMANDS: [`make build`, `make run-neo4j`]
LINT COMMANDS: [`make lint`]
FORMAT COMMANDS: [`make fmt-check`] — runs `gofmt -l .` and fails on any output. `make fmt` rewrites silently and is NOT a gate; use `make fmt-check` to verify. CI's golangci-lint v2 includes `gofmt` as a formatter and will fail the build on unformatted code.
LICENSE CHECK: [`make license-check`]

**Always run `make test`, `make fmt-check`, AND `make lint` as final gates before marking any task or plan complete.** All tests must pass, no file may need gofmt, and lint must be clean — a build that compiles but has failing tests, unformatted code, or lint errors is not done. `make fmt-check` is the local equivalent of CI's gofmt linter, so drift fails before the push instead of after.

## Cobra Command Layout

The repo follows a strict one-file-per-leaf cobra layout. Every command tree under `neo4j-cli/aura/internal/subcommands/<resource>/` and `common/skill/` follows it. Mirror it for any new command tree:

- **Parent file `<resource>.go`** — defines `NewCmd(cfg, ...) *cobra.Command`, registers persistent flags, calls `cmd.AddCommand(newXxxCmd(cfg, ...))` for each leaf. Keep it small (≤80 lines).
- **One file per leaf action `<action>.go`** — defines a private constructor like `newInstallCmd(cfg, ...) *cobra.Command` containing the leaf's flags + `RunE`. No leaf bodies inlined into the parent.
- **Colocated tests `<action>_test.go`** — tests for each leaf live next to its source.
- **Shared test helpers** in `<resource>_helpers_test.go` (or similar) when needed.

Examples: `neo4j-cli/aura/internal/subcommands/instance/{instance.go, list.go, list_test.go, get.go, delete.go, ...}` and `common/skill/{skill.go, install.go, remove.go, list.go, check.go, helpers.go}`.

Don't inline multiple leaves in the parent. Don't name the parent `command.go` — name it after the resource so `grep <resource>.go` finds it. Adding a new leaf = new `<action>.go` + `<action>_test.go`, plus one `cmd.AddCommand(...)` line in the parent.

See [`.agents/cobra.md`](.agents/cobra.md) for Cobra flag access and flag precedence gotchas.

## Project Overview

PRIMARY LANGUAGES: [Go]

Neo4j CLI (`neo4j-cli`) is a command-line tool for interacting with Neo4j.

## Build System

BUILD SYSTEMS: [Go toolchain, Makefile, golangci-lint, GoReleaser, changie]

See [`.agents/build.md`](.agents/build.md) for full details.

- Local build: `make build` (produces `bin/neo4j-cli`)
- Local run (no build): `make run-neo4j`
- Release build (current platform, ldflags baked in): `make snapshot` (uses goreleaser, outputs to `bin/`)
- npm publish dry-run (template + ordering check): `make npm-publish-dry`. Works against empty `dist/` because `publish.sh --dry-run` stubs missing platform binaries with a 1-byte placeholder; run `make snapshot` first if you want real binaries packed. Real-binary path (CI) still hard-errors on missing binaries — the stub is dry-run-only.
- All `.go` files must start with the Neo4j copyright header (enforced in CI via `addlicense`)
- PRs require a changelog entry via `make changelog` **only for user-facing changes** (new features, bug fixes, behaviour changes visible to CLI users). Internal changes (CI/CD workflow fixes, build scripts, code refactors with no visible effect) do not need changelog entries. Use `changie new --projects neo4j-cli --kind <kind> --body <body>` for non-interactive use.

## Testing Framework

TESTING FRAMEWORKS: [Go testing, testify, afero (in-memory FS)]

See [`.agents/testing.md`](.agents/testing.md) for full details and output testing notes.

- Tests are colocated with source as `*_test.go` files
- Run with `go test ./...`; CI runs on ubuntu, windows, and macos
- Mock HTTP server and filesystem helpers live in `neo4j-cli/aura/internal/test/testutils/`
- `neo4j-cli/` (the super-CLI package) has no test files; this is a pre-existing gap
- **Prefer table-driven tests** (`for _, tc := range []struct{...}{...}`) when writing new tests — they reduce duplication and make it easy to add cases later
- **Name test files per command**, not per package — use `get_test.go`, `set_test.go`, `list_test.go` mirroring the source files; put shared helpers in `helpers_test.go`. Avoid aggregating all tests in a single `config_test.go`.
- **Never use `afero.NewOsFs()` in query package tests** — the dev machine has real credentials at `~/Library/Preferences/neo4j/cli/credentials.json`. Tests using a real FS will fail if any dbms credential is stored. Always use `testfs.GetTestFs(`{"format":"json"}`, "{}")` (empty credentials) even when testing dotenv walk-up; write the dotenv into the memFs at `filepath.Join(t.TempDir(), ".env")` and `t.Chdir(tmp)` so `os.Getwd()`+`cfg.Aura.Fs()` finds it.

## Architecture

ARCHITECTURE PATTERN: Cobra command tree — one file per leaf command, directory structure mirrors command hierarchy

See [`.agents/architecture.md`](.agents/architecture.md) for architecture details and toon-go notes.

One binary is produced:
- **`neo4j-cli`** — single CLI entrypoint (`neo4j-cli/main.go`); the Aura command tree lives under the `aura` subcommand.

```
neo4j-cli/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [neo4j-labs/neo4j-cli](https://github.com/neo4j-labs/neo4j-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
