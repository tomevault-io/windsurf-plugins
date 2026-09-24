---
trigger: always_on
description: herdr ≥ 0.8.2 plugin. It sequences short linear YAML workflows (`agent` / `run` / `herdr` / `workflow`). herdr owns host panes and lifecycle. This repository owns the picker overlay and CLI, and loads and runs workflow steps. Runtime is Go with Charm TUI adapters.
---

# herdr-workflows

herdr ≥ 0.8.2 plugin. It sequences short linear YAML workflows (`agent` / `run` / `herdr` / `workflow`). herdr owns host panes and lifecycle. This repository owns the picker overlay and CLI, and loads and runs workflow steps. Runtime is Go with Charm TUI adapters.

Workflow format is `version: v1alpha1`. The package stays semver `0.x`. A later incompatible alpha increments `v1alphaN`. Workflow YAML never declares a herdr version. The plugin manifest and CLI own minimum version and protocol enforcement.

Invariants of record are the loader, `docs/workflow.schema.json` and the embed schema, and the tests. Code is current behavior. The user-facing contract lives in `docs/` and `README.md`. herdr runtime behavior comes from `.agents/references/herdr/docs/next/website/src/content/docs/` with the checkout detached at the release tag (currently v0.8.2). Never invent it from memory. Clone and update that checkout with `.agents/references/AGENTS.md`.

Before behavior work, read `CONTRIBUTING.md`.

## Commands

```bash
go tool verify                             # every host-feasible check (same as CI)
go tool verify -fast                       # pre-commit
go run ./scripts/generate-workflow-schema  # regenerate docs/workflow.schema.json
go run ./scripts/gen-herdr-methods         # regenerate internal/host/herdr_methods.gen.go
go run ./scripts/sync-embed                # copy skills/, manifest, logo, schema into embed/
go run ./scripts/install-dev               # compile + herdr plugin link + keybindings + reload
```

- Pre-commit (`.githooks/pre-commit`): `go tool verify -fast`.
- CI (`.github/workflows/verify.yml`): `go tool verify` on Linux and macOS after it installs Node.js, golangci-lint, and GoReleaser. Docs publish (`.github/workflows/docs.yml`) runs `npm ci && npm run build` in `docs/`.
- After `go run ./scripts/install-dev`, the live binary is `bin/herdr-workflows`.
- Remote GitHub install downloads the verified release archive for the cloned tag into `bin/herdr-workflows`, then runs `bin/herdr-workflows setup`. The target does not need Go. Local link/dev compiles with `go build` / `go run ./scripts/install-dev`.

## Layout

Go packages under `internal/` and `embed/` (schema, logo, and skill catalog bytes). Test the Go package whose interface you changed.

| Path                                         | Role                                                                                                          |
| -------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| `main.go`                                    | plugin binary entry                                                                                           |
| `internal/cli/`                              | Cobra commands, terminal I/O, `hwf init` / `setup`                                                            |
| `embed/` + `internal/cli/`                   | embedded skill catalog (`assets`) and `hwf skills` registry/show formatting                                   |
| `internal/update/`                           | GitHub release check, managed-plugin `hwf update`, and distribution artifact names/checksums |
| `internal/picker/`                           | picker TUI, workflow rows, ctrl+p palette, update indicator, Parity Baseline                                  |
| `internal/console/`                          | full-screen console TUI, workflows/runs lists, run debug tabs, Parity Baseline                                |
| `internal/runsbrowser/`                      | runs browser TUI, list/detail, run-history presentation, Parity Baseline                                      |
| `internal/tui/`                              | Charm lipgloss/bubbletea adapter shared by picker, console, and runs browser, Parity Baseline                 |
| `internal/workflow/`                         | Workflow Authoring: Definition, document parse, templates, conditions (`when:`), trust, exchange, inputs      |
| `internal/engine/`                           | Workflow Execution: Run, workflow runner, step runners, pane placement, agent turns, detached launch          |
| `internal/history/`                          | Run Observation: Snapshot, Summary, Detail, project claims, recorder, retention                               |
| `internal/host/`                             | Herdr Adapter: explicit identities, generated params/result validation, denylist rail, socket RPC, CLI        |
| `internal/config/`                           | profile/transcript config layers, repo root, invocation context                                               |
| `internal/caps/`                             | byte caps and their guards                                                                                    |
| `internal/transcript/`                       | transcript extractor table, built-in Claude transcript read                                                   |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aorumbayev/herdr-workflows](https://github.com/aorumbayev/herdr-workflows) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
