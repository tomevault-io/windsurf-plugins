---
trigger: always_on
description: Behavioural guidance for AI agents working in this repository. Reference
---

# AGENTS.md

Behavioural guidance for AI agents working in this repository. Reference
material for complex procedures lives next to the code — integration
testing is documented in [`cmd/hi/README.md`](cmd/hi/README.md) and
[`integration/README.md`](integration/README.md). Read those files
before running tests or writing new ones.

Headscale is an open-source implementation of the Tailscale control server
written in Go. It manages node registration, IP allocation, policy
enforcement, and DERP routing for self-hosted tailnets.

## Interaction Rules

These rules govern how you work in this repo. They are listed first
because they shape every other decision.

### Ask with comprehensive multiple-choice options

When you need to clarify intent, scope, or approach, use the
`AskUserQuestion` tool (or a numbered list fallback) and present the user
with a comprehensive set of options. Cover the likely branches explicitly
and include an "other — please describe" escape.

- Bad: _"How should I handle expired nodes?"_
- Good: _"How should expired nodes be handled? (a) Remain visible to peers
  but marked expired (current behaviour); (b) Hidden from peers entirely;
  (c) Hidden from peers but visible in admin API; (d) Other."_

This matters more than you think — open-ended questions waste a round
trip and often produce a misaligned answer.

### Read the documented procedure before running complex commands

Before invoking any `hi` command, integration test, generator, or
migration tool, read the referenced README in full —
`cmd/hi/README.md` for running tests, `integration/README.md` for
writing them. Never guess flags. If the procedure is not documented
anywhere, ask the user rather than inventing one.

### Map once, then act

Use `Glob` / `Grep` to understand file structure, then execute. Do not
re-explore the same area to "double-check" once you have a plan. Do not
re-read files you edited in this session — the harness tracks state for
you.

### Fail fast, report up

If a command fails twice with the same error, stop and report the exact
error to the user with context. Do not loop through variants or
"try one more thing". A repeated failure means your model of the problem
is wrong.

### Confirm scope for multi-file changes

Before touching more than three files, show the user which files will
change and why. Use plan mode (`ExitPlanMode`) for non-trivial work.

### Prefer editing existing files

Do not create new files unless strictly necessary. Do not generate helper
abstractions, wrapper utilities, or "just in case" configuration. Three
similar lines of code is better than a premature abstraction.

## Quick Start

```bash
# Enter the nix dev shell (Go 1.26.1, buf, golangci-lint, prek)
nix develop

# Full development workflow: fmt + lint + test + build
make dev

# Individual targets
make build           # build the headscale binary
make test            # go test ./...
make fmt             # format Go, docs, proto
make lint            # lint Go, proto
make generate        # regenerate protobuf code (after changes to proto/)
make clean           # remove build artefacts

# Direct go test invocations
go test ./...
go test -race ./...

# Integration tests — read cmd/hi/README.md first
go run ./cmd/hi doctor
go run ./cmd/hi run "TestName"
```

Go 1.26.1 minimum (per `go.mod:3`). `nix develop` pins the exact toolchain
used in CI.

## Pre-Commit with prek

`prek` installs git hooks that run the same checks as CI.

```bash
nix develop
prek install            # one-time setup
prek run                # run hooks on staged files
prek run --all-files    # run hooks on the full tree
```

Hooks cover: file hygiene (trailing whitespace, line endings, BOM),
syntax validation (JSON/YAML/TOML/XML), merge-conflict markers, private
key detection, nixpkgs-fmt, prettier, and `golangci-lint` via
`--new-from-rev=HEAD~1` (see `.pre-commit-config.yaml:59`). A manual
invocation with an `upstream/main` remote is equivalent:

```bash
golangci-lint run --new-from-rev=upstream/main --timeout=5m --fix
```

`git commit --no-verify` is acceptable only for WIP commits on feature
branches — never on `main`.

## Project Layout

```
headscale/
├── cmd/
│   ├── headscale/    # Main headscale server binary
│   └── hi/           # Integration test runner (see cmd/hi/README.md)
├── hscontrol/        # Core control plane
├── integration/      # End-to-end Docker-based tests (see integration/README.md)
├── proto/            # Protocol buffer definitions
├── gen/              # Generated code (buf output — do not edit)
├── docs/             # User and ACL reference documentation
└── packaging/        # Distribution packaging
```

### `hscontrol/` packages

- `app.go`, `handlers.go`, `grpcv1.go`, `noise.go`, `auth.go`, `oidc.go`,
  `poll.go`, `metrics.go`, `debug.go`, `tailsql.go`, `platform_config.go`
  — top-level server files
- `state/` — central coordinator (`state.go`) and the copy-on-write
  `NodeStore` (`node_store.go`). All cross-subsystem operations go
  through `State`.
- `db/` — GORM layer, migrations, schema. `node.go`, `users.go`,
  `api_key.go`, `preauth_keys.go`, `ip.go`, `policy.go`.
- `mapper/` — streaming batcher that distributes MapResponses to
  clients: `batcher.go`, `node_conn.go`, `builder.go`, `mapper.go`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [juanfont/headscale](https://github.com/juanfont/headscale) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
