---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a **documentation aggregation repository** that combines documentation from multiple Infrahub-related projects into a single documentation website using Docusaurus. The content here is mostly read-only and automatically synced from source repositories via git submodules and GitHub Actions workflows.

**Important**: Most documentation changes should be made in the source repositories, not here. Direct edits to synced content in this repository will be overwritten during the next sync cycle.

### Source Repositories

Documentation is synced from these upstream repositories:

- Core Infrahub docs: `opsmill/infrahub`
- Python SDK docs: `opsmill/infrahub-sdk-python`
- Ansible collection docs: `opsmill/infrahub-ansible`
- Service catalog docs: `opsmill/infrahub-demo-service-catalog`
- Nornir integration docs: `opsmill/nornir-infrahub`
- Schema library docs: `opsmill/schema-library`
- Infrahub Sync docs: `opsmill/infrahub-sync`
- Emma docs: `opsmill/emma`
- Infrahub Skills docs: `opsmill/infrahub-skills`

### Git Submodules

The repository uses a git submodule at `infrahub/` that tracks the `stable` branch from `opsmill/infrahub`. Update submodules with:

```bash
git submodule update --recursive --remote
```

## Development Commands

### Quick Start

All Docusaurus commands run from the `docs/` directory:

```bash
cd docs
npm install        # Install dependencies (first time or after package.json changes)
npm start          # Start local development server with hot reload (http://localhost:3000)
npm run build      # Build static site for production
npm run serve      # Serve built site locally
npm run typecheck  # Run TypeScript type checking
npm run clear      # Clear Docusaurus build cache
```

### Python Task Runner (Invoke)

From the repository root, use Python invoke tasks:

```bash
uv run invoke docs        # Build documentation (runs npm run build in docs/)
uv run invoke lint        # Run all linters (ruff + yamllint)
uv run invoke format      # Format Python files with ruff
```

**Note**: Requires uv installation. Run `uv sync` to set up Python dependencies.

### Linting and Quality Checks

Run from the repository root:

```bash
# Markdown linting
markdownlint --config .markdownlint.yaml docs/**/*.mdx docs/**/*.md

# Documentation style validation (requires Vale installation)
vale --glob='!{docs/docs-schema-library/reference/*}' $(find ./docs -type f \( -name "*.mdx" -o -name "*.md" \) )

# Python linting and formatting
ruff check .
ruff format .                    # Auto-format Python files
ruff format --check --diff       # Check formatting without changes

# YAML linting
yamllint -s .
```

### Linting Configuration Files

- `.markdownlint.yaml`: Markdown linting rules (repository root)
  - Disables line length limits (MD013)
  - Allows inline HTML (MD033)
  - Permits duplicate headings for tabs (MD024)
- `.vale.ini`: Documentation style rules and vocabulary (in submodules)
- `.yamllint.yml`: YAML formatting and style rules
- `pyproject.toml`: Ruff configuration (line-length: 120)

## Architecture

### Multi-Plugin Documentation Structure

This site uses multiple Docusaurus plugin instances to combine documentation from different sources. Each plugin creates an independent documentation section with its own URL path and sidebar.

**Plugin Architecture**:

- Each documentation section is defined as a plugin in `docs/docusaurus.config.ts`
- Each plugin has its own `docs-<name>/` directory and `sidebars-<name>.ts` file
- Plugins are configured with unique IDs, route paths, and edit URLs pointing to source repositories

**Documentation Sections**:

| Section | Directory | Route | Source Repo |
|---------|-----------|-------|-------------|
| Core Infrahub | `docs/docs/` | `/` | `opsmill/infrahub` |
| Python SDK | `docs/docs-python-sdk/python-sdk/` | `/python-sdk` | `opsmill/infrahub-sdk-python` |
| Infrahubctl | `docs/docs-python-sdk/infrahubctl/` | `/infrahubctl` | `opsmill/infrahub-sdk-python` |
| Ansible | `docs/docs-ansible/` | `/ansible` | `opsmill/infrahub-ansible` |
| Infrahub Sync | `docs/docs-sync/` | `/sync` | `opsmill/infrahub-sync` |
| Nornir | `docs/docs-nornir/` | `/nornir` | `opsmill/nornir-infrahub` |
| DC Fabric Demo | `docs/docs-demo/` | `/demo` | Demo content |
| Service Catalog | `docs/docs-service-catalog/` | `/demo-service-catalog` | `opsmill/infrahub-demo-service-catalog` |
| Schema Library | `docs/docs-schema-library/` | `/schema-library` | `opsmill/schema-library` |
| Emma | `docs/docs-emma/` | `/emma` | `opsmill/emma` |
| VS Code Extension | `docs/docs-vscode/` | `/vscode` | VS Code extension docs |
| MCP Server | `docs/docs-mcp/` | `/mcp` | MCP integration docs |
| Skills | `docs/docs-skills/` | `/skills` | `opsmill/infrahub-skills` |
| Integrations | `docs/docs-integrations/` | `/integrations` | Integration guides |
| Exporter | `docs/docs-exporter/` | `/exporter` | Exporter documentation |

### Key Configuration Files

- `docs/docusaurus.config.ts`: Main configuration with all plugin definitions, navbar, theme, redirects, and analytics

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [opsmill/infrahub-docs](https://github.com/opsmill/infrahub-docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
