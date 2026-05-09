---
trigger: always_on
description: Agent-agnostic Docker sandbox for AI coding agents (Claude Code, Gemini CLI, Codex). Single Go binary, cross-compiled for ARM and x86.
---

# Asylum

Agent-agnostic Docker sandbox for AI coding agents (Claude Code, Gemini CLI, Codex). Single Go binary, cross-compiled for ARM and x86.

## Change Management

This project uses [OpenSpec](https://openspec.dev) for structured change management. Use the `/opsx:propose` skill to start a new change, `/opsx:apply` to implement, and `/opsx:archive` to archive completed changes. See `openspec/` for specs and change history.

## Architecture

- **Go** (latest stable) — single binary, no runtime dependencies beyond Docker
- Cross-compiled for `linux/amd64`, `linux/arm64`, `darwin/amd64`, `darwin/arm64`
- Shells out to Docker CLI via `os/exec` and `syscall.Exec` (process replacement)
- Layered YAML config: `~/.asylum/config.yaml` → `$project/.asylum` → `$project/.asylum.local` → CLI flags
- Embedded assets (Dockerfile.core/tail, entrypoint.core/tail) via `go:embed`
- Manual CLI argument parsing with passthrough semantics (unknown flags forwarded to agents)
- Kit system: modular, composable tooling profiles (languages, tools, services) that inject Dockerfile/entrypoint/config/rules snippets
- TUI wizard (bubbletea) for first-run setup and kit selection

### Project Structure

```
cmd/asylum/main.go          CLI entry point, argument parsing, dispatch
internal/
  agent/                    Agent interface + implementations (Claude/Gemini/Codex/OpenCode/Echo)
                            Agent install system for Dockerfile snippets
  config/                   Layered YAML config loading, merging, volume parsing
                            Config migration (v1→v2), kit sync, state tracking, defaults
  container/                Docker run arg assembly, volume/env/port orchestration
  docker/                   Thin Docker CLI wrapper (build, inspect, prune)
  firstrun/                 First-run wizard (kit selection, agent config seeding)
  image/                    Two-tier image management with hash-based rebuild detection
  kit/                      Kit system: modular tooling profiles (java, node, python, docker, etc.)
                            Each kit contributes Dockerfile/entrypoint/config/rules snippets
  log/                      Colored terminal output (info/success/warn/error/build)
  onboarding/               In-container post-start tasks (e.g. npm install) with state tracking
  ports/                    Host port allocation registry (file-locked, per-project)
  selfupdate/               Self-update from GitHub releases (stable + dev channels)
  ssh/                      SSH directory setup and key generation
  term/                     Terminal detection and shell quoting
  tui/                      Terminal UI components (wizard, select, multiselect, confirm)
assets/
  Dockerfile.core           Base Dockerfile template (embedded via go:embed)
  Dockerfile.tail           Dockerfile suffix appended after kit snippets
  entrypoint.core           Base entrypoint script template
  entrypoint.tail           Entrypoint suffix appended after kit snippets
  asylum-reference.md       In-container reference documentation
  assets.go                 go:embed declarations
e2e/                        End-to-end tests (Docker-based, separate from integration/)
```

### Key Behaviors

- **First-run wizard** (`firstrun/`) guides kit selection and agent config seeding on first use. Agent config is seeded from host (`~/.claude` → `~/.asylum/agents/claude/`), but resume is skipped for that first session since seeded data doesn't represent a container session.
- **Two-tier images**: a base image (shared across projects, kit-driven) and per-project images (project-specific packages, kits). Base image rebuild invalidates all project images (`baseRebuilt` flag cascades to `EnsureProject`).
- **Kit-driven image assembly**: Dockerfile and entrypoint are assembled from core templates + kit snippets + tail. Each kit registers Dockerfile, entrypoint, config, and rules snippets. Kits have tiers (global vs project-level).
- **Config migration**: v1→v2 migration (`config/migrate.go`) handles schema evolution. New kits are detected and offered via `config/kitsync.go`.
- **Port allocation**: `ports/` maintains a file-locked registry so each project gets non-overlapping host port ranges.
- **Self-update**: checks GitHub releases for updates (stable and dev channels).
- **Onboarding**: in-container tasks (e.g. `npm install`) run post-start with hash-based change detection to avoid redundant work.
- Container names are deterministic: `asylum-<sha256(project_dir)[:12]>`.
- Project directory is mounted at its real host path (not `/workspace`), preserving absolute paths.
- **The entrypoint script must never install anything.** It configures the environment (PATH, mise, fnm, git, direnv) but all tool/package installation belongs in the Dockerfile (base or project image). Installing in the entrypoint adds latency to every container start and is not cached.

## Code Style

### General

- **Less code is better.** Every line must earn its place. Avoid defensive boilerplate, speculative abstractions, and "just in case" code paths.
- Use modern Go: generics where they reduce duplication, errors as values, `slices`/`maps` packages.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [inventage-ai/asylum](https://github.com/inventage-ai/asylum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
