---
trigger: always_on
description: <!-- Copyright 2026 Phillip Cloud -->
---

<!-- Copyright 2026 Phillip Cloud -->
<!-- Licensed under the Apache License, Version 2.0 -->

# Copilot Coding Agent Instructions

## Project Overview

**micasa** is a terminal UI (TUI) application for managing home projects,
maintenance schedules, appliances, vendors, incidents, and expenses. It uses a
single SQLite database file with no cloud dependencies.

- **Language**: Go 1.25.5
- **TUI framework**: Charmbracelet Bubble Tea + lipgloss + huh forms
- **ORM**: GORM with pure-Go SQLite (`modernc.org/sqlite`)
- **LLM**: Optional integration via `any-llm-go` (Ollama, OpenAI-compatible)
- **License**: Apache-2.0

## Repository Layout

```
cmd/micasa/          CLI entry point (Kong parser: run, backup, config)
internal/
  app/               TUI layer (~30K lines): model, tabs, tables, forms,
                     overlays, styles, key dispatch, mouse zones
  data/              Persistence: GORM models (14 entities), CRUD, soft-delete,
                     schema introspection, seeding, backup, full-text search
  config/            TOML config with env var overrides (MICASA_*)
  extract/           Document extraction pipeline (text -> OCR -> LLM)
  llm/               LLM client wrapper and SQL prompts
  locale/            Currency formatting
  safeconv/          Safe int64->int narrowing (overflow check)
  fake/              Demo data generation
  ollama/            Ollama model pull API
docs/                Hugo static site (micasa.dev)
nix/                 NixOS module
plans/               Committed design documents
.claude/codebase/    Codebase map (structure.md, types.md, patterns.md)
```

## Building and Testing

### Build

```bash
go build ./cmd/micasa
```

Or with Nix:

```bash
nix build '.#micasa'
```

### Run Tests

```bash
go test -race -shuffle on -timeout 5m ./...
```

- Tests require extraction tools on some platforms (poppler-utils,
  tesseract-ocr, imagemagick). Tests in `internal/extract/ci_test.go` are
  skipped locally if these tools are missing.
- Do not use `-v` flag for normal test runs.
- Always use `-shuffle on` to catch order-dependent bugs.

### Lint

The project uses `golangci-lint` with strict settings (`.golangci.yml`):
exhaustive enum checks, wrapcheck, goconst (min 5 occurrences), type safety
linters, and more.

```bash
golangci-lint run ./...
```

Pre-commit hooks run automatically on `git commit` and include formatting,
import ordering, and license header checks.

### CI

GitHub Actions workflows:

- **ci.yml**: Tests on 5 OS/arch combinations (Ubuntu x86/ARM, macOS, Windows
  x86/ARM) with race detector. Includes benchmarks, Nix build, and docs build.
- **lint.yml**: golangci-lint, deadcode detection, pre-commit hooks.
- **security.yml**: govulncheck, osv-scanner, CodeQL, TruffleHog secrets scan.
- **release.yml**: goreleaser for binary/container releases on GitHub Release.
- **pages.yml**: Hugo site deploy to GitHub Pages.

CI uses change detection (`.github/detect-ci-changes.bash`) to skip jobs
when only non-Go or metadata-only files changed.

## Coding Conventions

### Go Style

- **Unexported by default**: Only export types/functions/fields when another
  package needs access.
- **Error handling**: No broad catches or silent defaults. Propagate errors
  explicitly. Use `testify/assert` and `testify/require` in tests (`require`
  for preconditions, `assert` for assertions). Prefer `require`/`assert` over
  bare `t.Fatal`/`t.Error` except for truly unreachable branches or
  specialized test harness helpers.
- **Type safety**: Never cast `int64` to `int` directly. Use
  `safeconv.Int()` from `internal/safeconv` which returns an error on overflow.
- **Enum switches**: Define typed `iota` constants. The `exhaustive` linter
  catches missing cases. Never switch on bare integers.
- **Constants over magic numbers**: Use `math.MaxInt64` or codebase constants.
- **DRY**: Search for existing helpers before creating new ones.

### Architecture

- **Bubble Tea Model-Update-View**: Central `Model` struct dispatches messages
  by type, then by mode. Each entity tab implements the `TabHandler` interface.
- **Rendering pipeline**: `View() -> buildView() -> baseView + overlay stack`
  (dashboard, calendar, notes, finder, extraction, chat, help).
- **Soft delete**: GORM `Delete()` sets `deleted_at` and creates a
  `DeletionRecord` for audit. `checkDependencies()` prevents orphans.
- **Single-file backup**: `micasa backup backup.db` must produce a complete
  backup. Never store app state outside SQLite.
- **LLM is opt-in**: Every feature must work fully without the LLM.
- **Deterministic ordering**: Every `ORDER BY` clause must include a tiebreaker
  (typically `id DESC`).

### UI/UX

- **Styles singleton**: All styles live as private fields on the `Styles`
  struct in `styles.go` with public accessor methods, referenced via the
  package-level `appStyles` singleton (e.g., `appStyles.Money()`). Never
  inline `lipgloss.NewStyle()` in rendering functions.
- **Wong palette**: Colorblind-safe colors using
  `lipgloss.AdaptiveColor{Light, Dark}`. See `styles.go`.
- **Key constants**: All keyboard key strings used in dispatch, `key.WithKeys`,
  `SetKeys`, `helpItem`, and display hints must use constants defined in
  `internal/app/model.go`. No bare key string literals.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [micasa-dev/micasa](https://github.com/micasa-dev/micasa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
