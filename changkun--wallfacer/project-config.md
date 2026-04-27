---
trigger: always_on
description: This file provides guidance when working with code in this repository.
---

This file provides guidance when working with code in this repository.

## Project Overview

Wallfacer is a task-board runner for AI agents. It provides a web UI where tasks are created as cards, dragged to "In Progress" to trigger AI agent execution in an isolated sandbox container, and results are inspected when done.

**Architecture:** Browser → Go server (:8080) → per-task directory storage (`data/<uuid>/`). The server runs natively on the host and launches ephemeral sandbox containers via `os/exec` (podman/docker). Each task gets its own git worktree for isolation.

For detailed documentation see `docs/`. The user manual is at `docs/guide/usage.md` and the technical internals index is at `docs/internals/internals.md`.

## Specs & Roadmap

Design specs live in `specs/`, organized by track:
- `specs/foundations/` — completed abstraction interfaces (sandbox, storage, container reuse, file explorer, terminal)
- `specs/local/` — local product and UX (spec coordination, desktop app, attachments, terminal extensions, enhancements)
- `specs/cloud/` — cloud platform (tenant filesystem, K8s sandbox, multi-tenant, tenant API)
- `specs/shared/` — cross-track specs (authentication, agent abstraction, native sandboxes, overlay snapshots)

The roadmap and dependency graph are in [`specs/README.md`](specs/README.md). When creating or modifying a spec:

- **Read `specs/README.md` first** to understand the track organization and dependency graph.
- **Place new specs** in the appropriate track directory. Use descriptive filenames without numeric prefixes (e.g., `specs/local/live-serve.md`).
- **Every spec MUST have valid YAML frontmatter** matching the spec document model (see `specs/local/spec-coordination/spec-document-model.md`). Required fields:
  ```yaml
  ---
  title: Human-readable title
  status: drafted          # vague | drafted | validated | complete | stale | archived
  depends_on:              # list of spec paths this one requires (DAG edges)
    - specs/shared/agent-abstraction.md
  affects:                 # packages and files this spec describes
    - internal/runner/
  effort: large            # small | medium | large | xlarge
  created: 2026-04-01      # ISO date
  updated: 2026-04-01      # ISO date, must be >= created
  author: changkun
  dispatched_task_id: null  # null or UUID (leaf specs only)
  ---
  ```
- **Assess cross-impacts** with existing specs. If a new spec modifies interfaces defined in foundations (`SandboxBackend`, `StorageBackend`), it must declare the dependency.
- **Update `specs/README.md`** when adding a spec — add it to the appropriate track table, the status overview, and the dependency graph.

## Build & Run Commands

```bash
make build          # Full gate: fmt + lint (Go + JS) + binary + pull sandbox images
make build-host     # Host-mode build: full gate, no image pull (for `wallfacer run --backend host`)
make build-binary   # Build just the Go binary (no fmt/lint/pull)
make pull-images    # Pull Claude and Codex sandbox images
make install-wails  # Install the Wails CLI (tracked as tool in go.mod)
make build-desktop  # Build native desktop app for current platform (uses go tool wails)
make build-desktop-darwin   # Build macOS .app bundle
make build-desktop-windows  # Build Windows .exe
make build-desktop-linux    # Build Linux desktop binary
make server         # Build and run the Go server natively
make shell          # Open bash shell in sandbox container for debugging
make clean          # Remove all sandbox images
make run PROMPT="…" # Headless one-shot Claude execution with a prompt
make test           # Run all tests (backend + frontend)
make test-backend   # Run Go unit tests (go test ./...)
make test-frontend  # Run frontend JS unit tests (cd ui && npx vitest@2 run)
make ui-css         # Regenerate Tailwind CSS from UI sources
make api-contract   # Regenerate API route artifacts from apicontract/routes.go
make e2e-lifecycle              # E2E: task lifecycle for both sandboxes (requires running server)
make e2e-lifecycle SANDBOX=claude  # E2E: task lifecycle for Claude only
make e2e-lifecycle BACKEND=host    # E2E against `wallfacer run --backend host`
make e2e-dependency-dag WORKSPACE=/path/to/repo  # E2E: dependency DAG with conflict resolution
```

CLI usage (after `make build-binary`, or `make build` for the full fmt/lint-gated build):

```bash
wallfacer                                    # Print help
wallfacer run                                # Start server, restore last workspace group
wallfacer run -addr :9090 -no-browser        # Custom port, no browser
wallfacer run --backend host                 # Host mode: exec claude/codex directly (no container)
wallfacer doctor                             # Check prerequisites and config
wallfacer doctor --backend host              # Readiness check for host-mode deployments
wallfacer status                             # Print board state to terminal
wallfacer status -watch                      # Live-updating board state
wallfacer exec <task-id-prefix>              # Attach to running task container
wallfacer exec --sandbox claude              # Open shell in a new sandbox
wallfacer spec validate                      # Validate the entire specs/ tree

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [changkun/wallfacer](https://github.com/changkun/wallfacer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
