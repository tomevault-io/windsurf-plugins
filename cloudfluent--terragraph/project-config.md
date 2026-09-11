---
trigger: always_on
description: Conventions for AI agents and reviewers working on terragraph. Process rules (PR title, squash-merge, release-please, review requirements) live in [`CONTRIBUTING.md`](CONTRIBUTING.md) and are not repeated here.
---

# AGENTS.md

Conventions for AI agents and reviewers working on terragraph. Process rules (PR title, squash-merge, release-please, review requirements) live in [`CONTRIBUTING.md`](CONTRIBUTING.md) and are not repeated here.

terragraph is a Go CLI that orchestrates independent Terraform/OpenTofu root modules as a graph. It shells out to `terraform`/`tofu`; it never generates or edits `.tf` files.

# Verifying changes

`make check` is the gate. It runs exactly what CI runs, in the same order: `fmt-check`, `lint`, `docs-check`, `build`, `test`, `vscode-check`. Run it before claiming a change is done.

Narrower loops while iterating:

- `make test`: full suite with `-race` (the race detector is always on; don't propose disabling it)
- `go test ./internal/graph -run TestBuild_UseVars`: one package or one test
- `make fmt`: reformat in place
- `make docs`: regenerate `docs/cli/*.md`

Go version is pinned to the `go.mod` value and matched in CI. Don't bump it as a side effect of another change.

# Layout

`cmd/terragraph` is the entrypoint; implementation lives under `internal/`, one domain per package, with the public SDK under `plugin/`:

| Package | Owns |
|---|---|
| `blueprint` | Parsing `blueprint.hcl` / `group.hcl` into typed values |
| `graph` | Building and validating the node graph, cycle detection, group expansion |
| `plugins` | Executable plugin host: package installation, locks, sessions, and diagnostics |
| `plugin` (public) | SDK descriptors, typed values, logging, and RPC protocol; no engine orchestration |
| `engine` | Execution: levels, plan/apply/destroy, input resolution, approval |
| `exec` | The terraform/tofu subprocess wrapper and the ephemeral tfvars file |
| `vendor` | Fetching and rewriting module sources |
| `cli` | Cobra commands, flag wiring, output formatting |
| `language`, `lsp` | Editor intelligence |
| `runlock` | Cross-process advisory lock on a blueprint directory |
| `module` | Terraform module introspection |
| `pathidentity` | Read-only filesystem path equality, including known directory case rules |

Dependencies flow one way: `cli` → `engine` → `graph` → `blueprint`, `cli` → `plugins` and `engine` → `plugins` → `blueprint`, and `lsp` → `language` → `blueprint`. The host imports the public `plugin` SDK; the SDK never imports `internal/` packages. `blueprint`, `exec`, `module`, `runlock`, and `pathidentity` are leaves: they import nothing else under `internal/`, and keeping them that way is what makes them testable in isolation. Don't add an import that reverses the direction or gives a leaf a dependency, and don't reach into another package to do work it should expose.

Before adding a package, function, or type, check whether an existing one already covers it. Extend the existing implementation rather than building a parallel one.

# Execution invariants

These are the ones that fail silently, so they are worth stating even though the code enforces them today.

- **Every node always gets its own isolated `TF_DATA_DIR`** via `exec.Runner.DataDir`, not just nodes that share a `source`. Terraform caches which backend it was last configured with inside `.terraform/`, keyed by working directory, so two nodes sharing one data dir break each other's backend configuration rather than failing loudly. Never make this conditional.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cloudfluent/terragraph](https://github.com/cloudfluent/terragraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
