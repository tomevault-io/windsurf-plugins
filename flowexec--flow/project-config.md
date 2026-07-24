---
trigger: always_on
description: **flow** is a workflow automation hub for organizing automation across multiple projects (workspaces) with built-in secrets, templates, and cross-workspace composition. Users define workflows in YAML flow files, discover them visually, and run them anywhere.
---

# flow repo — Claude Code Context

## Project Overview

**flow** is a workflow automation hub for organizing automation across multiple projects (workspaces) with built-in secrets, templates, and cross-workspace composition. Users define workflows in YAML flow files, discover them visually, and run them anywhere.

This repo contains the flow CLI (Go). flow itself is used for all dev automation — build, test, generate, lint, release — via `.execs/*.flow` files.

---

## Critical Rules

Read these before touching any code:

1. **NEVER edit generated files** — `types/**/*.go`, `docs/cli/*.md`, `docs/types/*.md` are all auto-generated. Edit the schema source, not the output.
2. **Run `flow generate` after any schema change** in `types/*/schema.yaml` — CI will reject uncommitted generated diffs.
3. **Remove test focus markers before committing** — `FDescribe`, `FIt`, `FEntry` are temporary debugging tools, never ship them.
4. **Never use `logger.Log().FatalErr()` in `cmd/`** — use `errhandler.HandleFatal(ctx, cmd, err)` instead.
5. **`pkg/` is the stable API surface; `internal/` is unexported** — packages in `pkg/` may be imported outside the binary; `internal/` may not.

---

## Repository Structure

```
flow/
├── cmd/                 # Cobra CLI entry points and command handlers
├── pkg/                 # Shared, importable packages
│   ├── cache/           # Workspace and executable cache management
│   ├── cli/             # Shared CLI helpers and flag definitions
│   ├── context/         # Global app context (workspace, config, vault)
│   ├── errors/          # Typed errors with machine-readable codes
│   ├── filesystem/      # Path helpers and workspace file I/O
│   ├── logger/          # Structured logging
│   └── store/           # Persistence layer interfaces and implementations
├── internal/            # App logic NOT exported outside the binary
│   ├── fileparser/      # Flow file YAML parsing and validation
│   ├── io/              # Terminal UI and output rendering (tuikit)
│   ├── mcp/             # MCP server implementation (tools, resources)
│   ├── runner/          # Executable execution engine
│   ├── services/        # Business logic orchestration layer
│   ├── templates/       # Workflow template expansion
│   ├── updater/         # Auto-update logic
│   ├── utils/           # Internal utilities
│   ├── validation/      # Schema and config validation
│   ├── vault/           # Secret management
│   └── version/         # Build version info
├── types/               # Generated Go types from YAML schemas — DO NOT EDIT
├── tests/               # E2E test suite (Ginkgo, -tags=e2e)
├── docs/                # Documentation source (flowexec.io) — CLI/type docs are generated
├── tools/               # Code generation and build tooling
└── .execs/              # flow dev automation executables (build, test, lint, release)
```

---

## Architecture

**CLI execution path:**
```
cmd/ (Cobra command) → pkg/context (workspace + config resolution)
  → internal/services (business logic) → internal/runner (execution engine)
  → type-specific handler in internal/runner/
```

**Type generation pipeline:**
```
types/*/schema.yaml → go-jsonschema → types/*/generated.go (DO NOT EDIT)
  → internal/fileparser (YAML parsing and validation)
```

**MCP server:**
`internal/mcp` exposes the same execution pipeline to AI tools over the Model Context Protocol. The `flow mcp` command starts the server. Claude Code, Cursor, and other MCP clients can call `mcp__flow__*` tools to run executables directly.

---

## Key Technologies

### Go CLI
- **Language**: Go 1.25+ (`go.mod:3`)
- **CLI Framework**: Cobra (`github.com/spf13/cobra`)
- **TUI**: Custom tuikit (`github.com/flowexec/tuikit`) built on Bubble Tea
- **Testing**: Ginkgo v2 BDD framework (`github.com/onsi/ginkgo/v2`)

---

## Development Workflow

The project uses flow itself for dev automation:

```bash
flow build binary ./bin/flow   # Build the CLI binary
flow test                      # Run all tests (unit + e2e) in parallel
flow lint                      # Run golangci-lint
flow generate                  # Run all code generation
flow validate                  # Full check: generate → lint → test → diff validation
flow install tools             # Install/update Go tools
flow mcp                       # Start the MCP server
flow browse                    # TUI explorer for discovering executables
```

### Using flow's MCP Tools in Sessions

During a Claude Code session, prefer MCP tools over raw shell when possible — they respect workspace config and handle environment setup:

```
mcp__flow__list_executables    # Browse all available executables
mcp__flow__execute             # Run an executable (e.g., ref: "test unit", ref: "lint")
mcp__flow__get_executable      # Inspect a specific executable's definition
mcp__flow__get_info            # Get current workspace context
mcp__flow__get_workspace       # Get workspace details
```

---

## Testing

- **Unit tests** (`-tags=unit`): Fast, no binary needed. `go test -race -tags=unit ./...`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flowexec/flow](https://github.com/flowexec/flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
