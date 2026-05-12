---
trigger: always_on
description: This file provides guidance to coding agents when working with this repository.
---

# AGENTS.md

This file provides guidance to coding agents when working with this repository.

## Directory Organization Rule - MANDATORY

The root directory must contain **no Go source files**. All code lives in clearly scoped subdirectories:

```text
/ (project root - no .go files here)
  go.mod, go.sum
  Makefile
  AGENTS.md (→ CLAUDE.md symlink)
  README.md, README_zh.md
  contributing.md, contributing_zh.md
  LICENSE, .gitignore, .github/
  changelog/
  chats/
  docs/
    agents/
      skill_directory.md
      memory_directory.md
    architecture/
      README.md
      README_zh.md
      overview.md
      overview_zh.md
      contexts.md
      contexts_zh.md
      layers.md
      layers_zh.md
      use-cases.md
      use-cases_zh.md
      runtime-and-storage.md
      runtime-and-storage_zh.md
    features.md
    features_zh.md
    config.md
    config_zh.md
    plans/
    superpowers/
  core/
    config/
    platform/
    shared/
    orchestration/
    readmodel/
    skillcatalog/
      app/
      domain/
      infra/
    promptcatalog/
      app/
      domain/
      infra/
    agentintegration/
      app/
      domain/
      infra/
    skillsource/
      app/
      domain/
      infra/
    backup/
      app/
      domain/
      infra/
  cmd/
    skillflow/
      main.go
      app.go, app_*.go
      adapters.go, providers.go, events.go, version.go
      process_*.go
      tray_*.go
      window_*.go
      single_instance_*.go
      wails.json
      build/
      frontend/
```

**Rules:**
- Never add `.go` files to the project root.
- New backend business code must go under a bounded context in `core/<context>/app`, `core/<context>/domain`, or `core/<context>/infra`.
- Cross-context write coordination belongs in `core/orchestration/`.
- Cross-context read composition belongs in `core/readmodel/`.
- Pure technical capabilities with no business ownership belong in `core/platform/`.
- `core/config/` is a frontend-facing settings facade and split/merge persistence adapter. Do not treat it as a bounded context.
- `core/shared/` is only for highly stable shared kernel concepts. Do not move context-local IDs or business rules there unless they are genuinely cross-context.
- `cmd/skillflow/` remains the Wails desktop shell, transport adapter layer, process host, and composition root.
- `wails.json` must stay co-located with `frontend/` inside `cmd/skillflow/`.
- The `//go:embed all:frontend/dist` directive in `main.go` works because both are in `cmd/skillflow/`.
- `go test ./core/...` is run from the module root.
- Import paths use the full module path: `github.com/shinerio/skillflow/core/...`.
- **`cmd/skillflow/*.go` files must remain flat.** Wails bindings require a single `package main` directory, so do not create subdirectories under `cmd/skillflow/`.
- Use file-name prefixes inside `cmd/skillflow/` as the organization convention:
  - `app.go`, `app_*.go` for Wails-facing transport methods
  - `events.go` for shell event types and emitters
  - `adapters.go`, `providers.go` for shell-side wiring
  - `process_*.go`, `tray_*.go`, `window_*.go`, `single_instance_*.go` for shell/runtime concerns
- When a concern grows large enough to warrant its own package, extract it to `core/` rather than creating a subdirectory inside `cmd/skillflow/`.

## Documentation Organization Rule - MANDATORY

**Root directory may contain only `README.md`, `README_zh.md`, `contributing.md`, `contributing_zh.md`, `REVIEW.md`, and `AGENTS.md` (with `CLAUDE.md` as symlink) as documentation files.**

All other documentation lives under `docs/`:

| File | Purpose |
|------|---------|
| `docs/agents/skill_directory.md` | Built-in agent scan/push directory reference |
| `docs/agents/memory_directory.md` | Common agent memory-file reference |
| `docs/features.md` | Complete UI/UX feature reference in English |
| `docs/features_zh.md` | Complete UI/UX feature reference in Chinese |
| `docs/architecture/README.md` | Architecture index and reading order (English) |
| `docs/architecture/README_zh.md` | Architecture index and reading order (Chinese) |
| `docs/architecture/overview.md` | High-level architecture overview (English) |
| `docs/architecture/overview_zh.md` | High-level architecture overview (Chinese) |
| `docs/architecture/` | Detailed DDD architecture set: overview, contexts, layers, use cases, and storage |
| `docs/config.md` | Persisted config and metadata file reference (English) |
| `docs/config_zh.md` | Same in Chinese |
| `docs/plans/` | Design and implementation plans |
| `docs/<module>/...` | Module-scoped design notes, plans, and reference docs such as `docs/superpowers/` |

**Rules:**
- `README.md` / `README_zh.md` are user-facing only: features overview, download/install links, skill format, cloud backup config, contributing/build instructions. No internal code snippets, no package tables, no architecture diagrams.
- `contributing.md` / `contributing_zh.md` are contributor-facing only: prerequisites, build/test/generate commands, and pointers to deeper architecture docs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shinerio/SkillFlow](https://github.com/shinerio/SkillFlow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
