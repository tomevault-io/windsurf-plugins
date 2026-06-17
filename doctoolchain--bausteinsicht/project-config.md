---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Bausteinsicht

Architecture-as-code tool with draw.io as visual frontend and bidirectional synchronization.

## Project Conventions

### Documentation
- All documentation in **English**
- Documentation format: **AsciiDoc (.adoc)**
- ADR path: `src/docs/arc42/ADRs/`
- ADR filename: `ADR-NNN-Name.adoc` (e.g., `ADR-001-DSL-Format.adoc`)
- ADR format: Nygard with Weighted Pugh Matrix (-1/0/1 scale)
- PRD path: `src/docs/PRD/`
- Spec path: `src/docs/spec/`
- Security reports: `src/docs/security/` (fortlaufend, mit Changelog)

### Technology Stack
- Implementation language: **Go** (ADR-002)
- DSL format: **JSONC with JSON Schema** (ADR-001)
- CLI framework: Cobra
- XML processing: beevik/etree
- No JavaScript/Node.js for the product itself (security concerns with npm supply chain)

### Quality Goals (Top 3)
1. **Learnability** — new users productive within 30 minutes
2. **IDE Support** — autocompletion/validation via JSON Schema, no plugin needed
3. **LLM Friendliness** — JSON model readable/writable by AI agents, CLI for automation

### Key Design Decisions
- Flexible element hierarchy (not limited to 4 C4 levels)
- Unique variable names as element IDs for synchronization
- Template-based styling (templates are draw.io files)
- Page-based drill-down navigation (one draw.io page per view, cross-page links + back button; see ADR-009)
- CLI + watch mode; CLI commands for LLM-driven workflows

## Development Environment

### Devcontainer (recommended)
A `.devcontainer/` configuration provides a fully reproducible dev environment with all tools pre-installed. Use with VS Code Dev Containers, GitHub Codespaces, or the `devcontainer` CLI.

Start the container and run Claude Code autonomously:
```bash
devcontainer up --workspace-folder .
devcontainer exec --workspace-folder . claude --dangerously-skip-permissions -p "your prompt"
```

Key details:
- Claude Code is installed via **native installer** (not npm) — no Node.js dependency
- draw.io runs headless via `xvfb-run` — use `drawio-export` wrapper for exports
- `COLORTERM=truecolor` is set for correct terminal color rendering

### Headless draw.io Export

The `bausteinsicht export` and `drawio-export` commands require:
1. **`dbus` daemon running** — Electron needs D-Bus for IPC. If export fails with "Export failed" or "input file/directory not found", start dbus:
   ```bash
   sudo mkdir -p /run/dbus && sudo dbus-daemon --system --fork
   ```
2. **`xvfb-run -a`** — the `-a` flag auto-picks a free display (avoids conflicts with existing X servers)
3. **`--no-sandbox`** — required in containers without user namespaces

The devcontainer `postStartCommand` starts dbus automatically. The `drawio-export` wrapper handles xvfb and `--no-sandbox`.

GPU errors in stderr (`"Exiting GPU process due to errors during initialization"`) are **harmless** — draw.io falls back to software rendering.

### Makefile
All build, test, and analysis commands are available via `make`:
- `make build` — build the CLI binary
- `make test` / `make test-race` — run tests (with race detector)
- `make check` — run all analysis tools + race-detected tests
- `make vet` / `make staticcheck` / `make gosec` / `make nilaway` / `make govulncheck` — individual analysis tools
- `make gitleaks` — scan for secrets
- `make golangci-lint` — meta-linter
- `make install-tools` — install Go-based tools

### Installed Tools
- `go vet`, `staticcheck` — static analysis
- `gosec` — security scanner
- `nilaway` — nil pointer analysis
- `govulncheck` — vulnerability scanner
- `golangci-lint` — meta-linter
- `gitleaks` — secret scanner
- `draw.io` CLI (headless via xvfb in devcontainer)
- `claude` (Claude Code CLI)
- `human` (gethuman.sh — AI agent issue tracker integration)

## Code Architecture

### Package Structure

```
cmd/bausteinsicht/     # CLI entry point — Cobra commands, one file per command
internal/model/        # DSL types, loader (JSONC→struct), validation, patch, resolve
internal/drawio/       # draw.io XML document/element/connector/label/template wrappers (beevik/etree)
internal/sync/         # Bidirectional sync engine: diff, forward/reverse apply, conflict resolution, state
internal/diagram/      # Export to C4-PlantUML / Mermaid text formats
internal/watcher/      # File-system watcher (fsnotify) for --watch mode
```

### Data Flow

1. **Model** — JSONC file parsed by `internal/model.Load()` into `BausteinsichtModel` (elements keyed by dot-path variable names, e.g. `system.backend.api`)
2. **Sync cycle** (`internal/sync.Run`) — pure function; no I/O:
   - `DetectChanges` diffs model+drawio against stored `SyncState` (`.bausteinsicht-sync` JSON file)
   - Conflict resolution: model always wins
   - `ApplyForward` writes model changes → draw.io XML
   - `ApplyReverse` writes draw.io label edits → model struct
3. **State** persisted atomically to `.bausteinsicht-sync` (SHA-256 checksummed JSON) so next sync can detect what changed on either side
4. **Export** — `export diagram` renders views to PlantUML/Mermaid; `export table` produces CSV/Markdown; `export` calls headless draw.io for PNG/PDF

### Key Conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [docToolchain/Bausteinsicht](https://github.com/docToolchain/Bausteinsicht) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
