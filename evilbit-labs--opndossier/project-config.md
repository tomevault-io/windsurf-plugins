---
trigger: always_on
description: This file provides guidance to GitHub Copilot when working with code in this repository.
---

# GitHub Copilot Instructions for opnDossier

This file provides guidance to GitHub Copilot when working with code in this repository.

**AGENTS.md is the single source of truth** for AI assistant behavior, development workflows, and architecture. This file provides Copilot-specific guidance derived from AGENTS.md.

## Rule Precedence (CRITICAL)

**Rules are applied in the following order of precedence:**

1. **Project-specific rules** (from AGENTS.md or .cursor/rules/)
2. **General development standards**
3. **Language-specific style guides** (Go conventions, etc.)

When rules conflict, always follow the rule with higher precedence.

## Project Overview

opnDossier is a tool for auditing and reporting on OPNsense configurations, with the primary goal of generating markdown views derived from OPNsense config.xml files. This project follows EvilBit Labs standards and is built for operators, by operators.

### Core Philosophy

- **Operator-Focused**: Build intuitive, efficient tools for end users
- **Offline-First**: Must operate in fully offline or airgapped environments with no external dependencies
- **Structured Data**: Data should be structured, versioned, and portable for auditable, actionable systems
- **Framework-First**: Leverage built-in functionality of established frameworks; avoid custom solutions

### EvilBit Labs Brand Principles

- **Trust the Operator**: Full control, no black boxes
- **Polish Over Scale**: Quality over feature-bloat
- **Offline First**: Built for where the internet isn't
- **Sane Defaults**: Clean outputs, CLI help that's actually helpful
- **Ethical Constraints**: No dark patterns, spyware, or telemetry

## Project Architecture & Data Flow

- **Monolithic Go CLI**: Converts OPNsense `config.xml` to Markdown, JSON, or YAML. No external network calls - offline-first.

- **Major Components**:

  - `cmd/`: CLI entrypoints (`convert`, `display`, `validate`). See `cmd/root.go` for command registration.

  - `internal/cfgparser/`: XML parsing to `schema.OpnSenseDocument` (XML DTO), converted to `common.CommonDevice` via `pkg/parser/opnsense/`.

  - `pkg/model/`: Platform-agnostic CommonDevice domain model.

  - `pkg/parser/`: Factory + DeviceParser interface (`pkg/parser/opnsense/` for OPNsense-specific converter).

  - `pkg/schema/opnsense/`: Canonical OPNsense XML data model structs.

  - `internal/processor/`: Normalization, validation, analysis, and transformation pipeline.

  - `internal/converter/`: Multi-format export (Markdown, JSON, YAML) using templates and options.

  - `internal/audit/`, `internal/compliance/`, `internal/plugins/`: Compliance audit engine and plugin system (STIG, SANS, firewall).

  - `internal/display/`, `internal/logging/`: Terminal output and structured logging.

  - **Data Flow**: `cfgparser` → `pkg/parser/opnsense` → `processor` → `converter` → `export`

- **Audit overlays**: `processor` → `audit` → `plugins`

## Technology Stack

| Layer             | Technology                                     |
| ----------------- | ---------------------------------------------- |
| **CLI Framework** | `cobra` v1.8.0 for command organization        |
| **Configuration** | `charmbracelet/fang` + `spf13/viper`           |
| **Styling**       | `charmbracelet/lipgloss` for terminal output   |
| **Markdown**      | `charmbracelet/glamour` for rendering          |
| **XML Parsing**   | `encoding/xml` for OPNsense config files       |
| **Logging**       | `charmbracelet/log` for structured logging     |
| **Data Formats**  | Support for XML, JSON, and YAML export formats |
| **Go Version**    | 1.26+                                          |

## Critical Workflows

- **All development tasks use `just`** (see `justfile`):
  - `just install` - install dependencies
  - `just build` - build binary
  - `just test` - run all tests
  - `just lint` - run golangci-lint
  - `just ci-check` - run full CI-equivalent checks (must pass before reporting success)
  - `just format` - format code and documentation
  - `just dev` - run in development mode
- **Pre-commit hooks may modify files**: It is normal for hooks to rewrite files during commit attempts, especially for formatting and other automatic fixes. Run `just ci-check` immediately before committing to reduce churn. If hooks still modify files, restage the changes and retry the commit.
- **Read `GOTCHAS.md` before substantial changes**: It captures non-obvious behaviors and repository footguns that frequently affect parser work, tests, and CLI behavior.
- **Markdown validation**: Never run `mdformat` directly. Use `pre-commit run -a` so the repository's configured plugins and formatting pipeline are applied correctly.
- **`//nolint:` placement**: Put `//nolint:` directives on their own line immediately above the relevant call or declaration. Inline directives may be stripped or broken by formatting tools.
- **No external dependencies**: All code must run fully offline.

## Go Coding Standards

### Code Style and Formatting

- **Tools**: `gofmt` (with tabs), `golangci-lint`, `go vet`, `go test -race`
- **Naming Conventions**:
  - **Packages**: `snake_case` or single word, lowercase
  - **Variables/functions**: `camelCase` for private, `PascalCase` for exported

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EvilBit-Labs/opnDossier](https://github.com/EvilBit-Labs/opnDossier) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
