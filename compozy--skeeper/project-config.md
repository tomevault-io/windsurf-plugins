---
trigger: always_on
description: This file provides project guidance for coding agents working in this repository.
---

# CLAUDE.md

This file provides project guidance for coding agents working in this repository.

## HIGH PRIORITY

- **IF YOU DON'T CHECK SKILLS** your task will be invalidated and we will generate rework
- **YOU CAN ONLY** finish a task if `make verify` passes at 100% (runs `fmt + lint + test + build`). No exceptions — failing any of these commands means the task is **NOT COMPLETE**
- **`make lint` has zero tolerance**. **Zero issues allowed** — any golangci-lint issue is a blocking failure
- **ALWAYS** check dependent package APIs before writing integration code or tests to avoid writing wrong code
- **NEVER** use workarounds — always use the `no-workarounds` skill for any fix/debug task + `testing-anti-patterns` for tests
- **ALWAYS** use the `no-workarounds` and `systematic-debugging` skills when fixing bugs or complex issues
- **NEVER** use web search tools to search local project code — for local code, use Grep/Glob instead
- **YOU SHOULD NEVER** add dependencies by hand in `go.mod` — always use `go get` instead

## MANDATORY REQUIREMENTS

- **MUST** run `make verify` before completing ANY subtask
- **ALWAYS USE** the `golang-pro` skill before writing any Go code
- **ALWAYS USE** the `systematic-debugging` + `no-workarounds` skills before fixing any bug
- **ALWAYS USE** the `testing-anti-patterns` skill before writing or modifying tests
- **ALWAYS USE** the `verification-before-completion` skill before claiming any task is done
- **Skipping any verification check will result in IMMEDIATE TASK REJECTION**

## Project Overview

**skeeper** is a single-binary Go command-line interface built with production-grade patterns and agent-ready tooling. The CLI is structured around `cobra` and exposes subcommands wired through `internal/cli`.

**Toolchain pin:** Go **1.26.2** (managed via `mise.toml` + `.go-version`), Bun **1.3.4** (`.bun-version`), `golangci-lint v2.11.4`, `gotestsum v1.13.0`, `gopls/modernize v0.21.1`, `goreleaser v2.6.1`. Run `mise install` to provision everything.

## Package Layout

| Path               | Responsibility                                               |
| ------------------ | ------------------------------------------------------------ |
| `cmd/skeeper`      | CLI entry point — thin shim that delegates to `internal/cli` |
| `internal/cli`     | Cobra root command, subcommand wiring, exit-code mapping     |
| `internal/config`  | Config loading and validation                                |
| `internal/version` | Build metadata (Version, Commit, BuildDate via -X)           |
| `internal/logger`  | Structured logging setup                                     |

## Build & Development Commands

```bash
# Full verification pipeline (BLOCKING GATE for any change)
make verify              # Serial: fmt -> lint -> test -> build

# Go pipeline
make fmt                 # gofmt every .go file
make lint                # golangci-lint v2 + gopls modernize (zero-tolerance)
make modernize           # gopls modernize idioms only
make test                # gotestsum + -race -parallel=4
make test-integration    # tests with `-tags integration`
make cover               # coverage.out + coverage.html
make build               # bin/skeeper with version ldflags

# Dependencies and tools
make deps                # go mod tidy + verify
make tools               # install gotestsum, golangci-lint, modernize, goreleaser

# JS/TS toolchain (for hooks + non-Go files)
make bun-lint            # oxfmt + oxlint
make bun-fmt             # apply oxfmt
make bun-fmt-check       # check oxfmt without writing

# Hooks and release
make hooks-install       # husky install (requires `bun install` first)
make release-snapshot    # local goreleaser snapshot under dist/
make docker-build        # build distroless container
```

## Hooks & Conventional Commits

- `pre-commit`: runs `lint-staged` — `make fmt` for `*.go`, `oxfmt`+`oxlint` for JS/TS, `oxfmt` for CSS/HTML/JSON/YAML/MD.
- `commit-msg`: enforced by `commitlint` with `@commitlint/config-conventional`.
- Allowed types: `build, chore, ci, docs, feat, fix, perf, refactor, test`. Scope is optional.
- Bootstrap: `bun install && make hooks-install` after cloning.

## Release Flow

Releases are tag-driven. Push a `v*` tag to trigger `.github/workflows/release.yml`:

```bash
git tag v0.1.0 && git push origin v0.1.0
```

Goreleaser (community v2) produces multi-OS/arch binaries (linux/darwin/windows × amd64/arm64), tarballs/zip archives, checksums, source tarball, and an auto-generated changelog. Optional channels (Homebrew tap, deb/rpm via nfpm, cosign signing, SBOMs) are scaffolded as commented blocks in `.goreleaser.yml`.

## CRITICAL: Git Commands Restriction

- **ABSOLUTELY FORBIDDEN**: **NEVER** run `git restore`, `git checkout`, `git reset`, `git clean`, `git rm`, or any other git commands that modify or discard working directory changes **WITHOUT EXPLICIT USER PERMISSION**
- **DATA LOSS RISK**: These commands can **PERMANENTLY LOSE CODE CHANGES** and cannot be easily recovered
- **REQUIRED ACTION**: If you need to revert or discard changes, **YOU MUST ASK THE USER FIRST**
- If the worktree contains unexpected edits, read them and work around them; do not revert them

## Code Search and Discovery


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [compozy/skeeper](https://github.com/compozy/skeeper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
