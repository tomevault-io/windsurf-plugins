---
trigger: always_on
description: Instructions for AI coding agents (Claude Code, Cursor, Codex, Aider, etc.) working in this repo.
---

# AGENTS.md

Instructions for AI coding agents (Claude Code, Cursor, Codex, Aider, etc.) working in this repo.

Humans: this is a quick map of where things live, what to run, and what trips agents up. The full prose is in `README.md` and `CONTRIBUTING.md`.

---

## What this repo is

Rookery is a Go monorepo that produces **`cilock`** — a witness-compatible CLI for emitting and verifying SLSA / in-toto attestations — plus the underlying library and 50+ attestor plugins. Read `README.md` first if you have no context.

The CLI binary entrypoint is `cilock/cmd/cilock/main.go`. Library entrypoint is `attestation/`. Plugins live in `plugins/attestors/<name>/` and `plugins/signers/<name>/`, each as its own Go module.

---

## Repo layout (memorise this)

```
attestation/         # Core library. AttestationContext, Attestor interface, DSSE.
cilock/              # The CLI. cmd/cilock/main.go blank-imports plugins.
plugins/attestors/   # 50+ attestors. Each has its own go.mod.
plugins/signers/     # signer plugins — canonical list in docs/signers.md.
presets/             # all/, cicd/, minimal/ — curated blank-import sets.
builder/             # Generates custom cilock binaries with chosen plugins.
compat/              # Import shims so legacy witness.dev paths still resolve.
docs/                # attestor-catalog.md is the canonical name reference.
scripts/             # CI scripts (check-dep-budget.sh, check-provenance.sh, gen-attestor-catalog.sh).
.provenance/         # JSON entries for code inlined from upstream — never edit by hand without updating the SHA.
```

`go.work` defines the workspace. The list of members is also what `MODULES` expands to in the `Makefile`.

---

## Commands you'll actually need

```bash
make build           # build every module via go.work
make test            # run all tests (no race)
make test-race       # race detector
make lint            # golangci-lint v2
make vet
make vulncheck       # govulncheck
make verify-isolated # build each module with GOWORK=off (catches accidental workspace coupling)
make docs            # regenerate docs/attestor-catalog.md from source
make tidy            # go mod tidy across every module
make help            # list everything
```

**Single-module workflow** (avoid the `for dir in` loop when iterating):
```bash
cd plugins/attestors/git
go test ./...               # uses go.work
GOWORK=off go test ./...    # isolation check
```

---

## Conventions (don't fight these)

### Commits
- **Signed commits required** on `main`. CI rejects unsigned.
- **Conventional Commits format**: `type(scope): description`. Types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`. CI rejects non-conventional commits on PRs.
- **Never use `--no-verify`** — pre-commit hooks exist for a reason. If a hook fails, fix the cause.
- **Never use `git rebase -i` or `git add -i`** — agents can't drive interactive prompts.

### Code
- **`os.Exit`** is allowed only in `main.go`. CI's `forbidden-patterns` job scans diffs.
- **`panic`** is allowed only in tests and `init()`. Same scanner.
- **`//nolint`** without a specific linter (e.g., `//nolint:gosec`) is rejected.
- Tests use **testify** (`require` + `assert`). Don't introduce other test frameworks.
- Comments that explain WHAT the code does are noise — well-named identifiers do that. Only write a comment when the WHY is non-obvious (hidden constraint, workaround for a specific bug, behavior that would surprise a reader).
- Don't add references to "the current task" / "added for X" / "TODO: cleanup after Y" — those belong in PR descriptions.

### Go version
All modules pin the version in `.go-version`. CI's `go-version-check` job rejects mismatches. If you bump, bump every `go.mod` + `go.work` in the same PR.

### Dependencies
- New direct deps in `cilock` are budgeted in **`.dep-budget.yaml`**. CI's `dep-budget` job blocks PRs that exceed the ceiling. If you legitimately need to raise it, bump the ceiling **in the same PR** so the new deps surface in review.
- Anything inlined from another project needs a matching entry in **`.provenance/<name>.json`** with the upstream commit SHA (40 chars, never a tag), license SPDX, and SHA256 of both upstream and local. Licenses must be on the allowlist in `NOTICE.md` (Apache-2.0, MIT, BSD-2-Clause, BSD-3-Clause, ISC — **not** MPL/GPL/LGPL).

### Predicate types
Aflock uses `https://aflock.ai/attestations/<name>/v<n>`. Legacy `witness.dev/attestations/<name>/v<n>` types are registered as aliases at CLI startup — both consume each other's chains. If you add a new attestor, register both.

---

## Attestor naming gotcha

The **directory name is not always the canonical attestor name.** The canonical name is what registers via `attestation.RegisterAttestation(Name, ...)` in each attestor's `*.go` file. Examples that bite:

| Directory | Canonical name (use this) |
|---|---|
| `plugins/attestors/commandrun` | `command-run` |
| `plugins/attestors/aws-iid` | `aws` |
| `plugins/attestors/githubaction` | `github-action` |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aflock-ai/rookery](https://github.com/aflock-ai/rookery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
