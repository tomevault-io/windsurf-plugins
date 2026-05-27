---
trigger: always_on
description: This repository contains the NHS Wales Architecture documentation site. These instructions will help you work efficiently with this codebase.
---

# GitHub Copilot Coding Agent Instructions

This repository contains the NHS Wales Architecture documentation site. These instructions will help you work efficiently with this codebase.

## Repository Overview

This is an **internal** architecture documentation repository for NHS Wales (GIG Cymru). It has:

- A companion **public** repository at <https://github.com/GIGCymru/architecture> (read-only, auto-synced)
- Documentation published to: <https://gigcymru.github.io/architecture/>
- Built using [Zensical](https://zensical.org/), a modern static site generator
- Markdown-based documentation under the `doc/` directory
- Architecture Decision Records (ADRs), principles, and design authority content

## Quick Start

### Environment Setup (Already Done)

The `.github/workflows/copilot-setup-steps.yml` workflow has already installed:

- Python 3.13
- uv package manager
- Just command runner
- Node.js and npm
- All project dependencies

You can immediately start working with the project using the commands below.

### Essential Commands

All commands use `just` (a modern command runner). View all commands with:

```bash
just --list
```

**Most Important Commands:**

```bash
just build          # Build the documentation site
just run            # Start local dev server (http://127.0.0.1:8000)
just qa             # Run ALL quality checks (MUST pass before committing)
just lint           # Lint markdown files only
just check-links    # Check internal links only
```

**Full Workflow:**

```bash
just                # Runs: install → sync → build → run
```

## Development Workflow

### Before Making Changes

1. **Always run quality checks first** to understand the baseline:
   ```bash
   just qa
   ```

2. **Test the build** to ensure no existing issues:
   ```bash
   just build
   ```

### Making Changes

1. **All documentation lives in `doc/` directory**
2. **Markdown files must follow:**
   - 120 character line limit (except URLs/links)
   - markdownlint-cli2 rules (see `.markdownlint-cli2.jsonc`)
   - Internal links must resolve correctly

3. **After changes, ALWAYS run:**
   ```bash
   just build          # Check for build warnings
   just qa             # Run all quality checks
   ```

4. **Test locally before committing:**
   ```bash
   just run            # Start dev server, verify changes render correctly
   ```

5. **Cross-reference related documents:**
   - When making changes to documents under `doc/`, check for relevant related content
   - Add links between related documents to help users navigate
   - Update both documents with reciprocal links where appropriate
   - This improves discoverability and helps users find related information

### Quality Assurance (CRITICAL)

**ALWAYS run `just qa` before committing.** This runs:

1. **Markdown linting** - Validates markdown syntax and style
2. **Internal link checking** - Ensures all internal links resolve
3. **Sync manifest verification** - Validates public sync configuration

These same checks run automatically on PRs and will block merging if they fail.

## Documentation Structure

### Architecture Decision Records (ADRs)

- **Location**: `doc/decisions/`
- **Template**: `doc/design-authority/dhcw/architecture-decision-record-template.md`
- **Naming conventions**: See `doc/decisions/meta-decisions/simplify-architecture-decision-records-structure.md`
- **Structure**: Each ADR is a single Markdown file (not a directory) using kebab-case naming
- **Important**: Always check if an ADR is marked as "Deprecated" or "Superseded" in its status - use the replacement ADR instead

### Architecture Principles

- **Location**: `doc/principles/`
- **Examples**: `doc/principles/architecture-principles.md`, `doc/principles/digital-products-and-software-engineering.md`

### Design Authority

- **Location**: `doc/design-authority/`
- **Templates**: Decision record and design overview templates
- **Meetings**: Meeting notes in `doc/design-authority/dhcw/meetings/`

### Diagrams

- **Use Mermaid** for diagrams in markdown files
- **Reference**: `doc/decisions/meta-decisions/use-mermaid-for-documenting-diagrams.md`

## Configuration Files

### Key Configuration

- **`zensical.toml`** - Site configuration (name, URL, navigation, theme)
- **`justfile`** - Command definitions
- **`pyproject.toml`** - Python dependencies
- **`package.json`** - Node.js dependencies (for linting)
- **`.markdownlint-cli2.jsonc`** - Markdown linting rules
- **`sync-public.toml`** - Public repository sync manifest

### Navigation

Navigation is **explicitly defined** in `zensical.toml` under the `nav` array. Files are not auto-discovered, so:

- **When adding new docs**, update `zensical.toml` navigation
- **When removing docs**, remove from navigation and consider sync manifest

## Public Repository Synchronization

This internal repository syncs selected files to the public repository:

- **Manifest**: `sync-public.toml` - Lists files to sync
- **Workflow**: `.github/workflows/sync-public.yml` - Runs on push to `main`
- **Command**: `just sync-public` - Test locally (dry-run by default)
- **Verification**: `just verify-sync-manifest` - Validates manifest

**Important:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GIGCymru/architecture](https://github.com/GIGCymru/architecture) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
