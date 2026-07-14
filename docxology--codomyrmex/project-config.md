---
trigger: always_on
description: **Version**: v1.3.0 | **Status**: Active | **Last Updated**: June 2026
---

# Codomyrmex Agents — Repository Root

**Version**: v1.3.0 | **Status**: Active | **Last Updated**: June 2026

## Purpose

This is the root coordination document for all AI agents operating within the Codomyrmex repository. It defines the top-level structure, surfaces, and operating contracts that govern agent interactions across the entire project.

Codomyrmex is a modular coding workspace enabling AI development workflows with **130** top-level modules under `src/codomyrmex/`, plus project workspaces under `projects/` for integration builds such as Paperclip adapters. This document serves as the central navigation hub for agents working with any part of the system. Repo metrics: [docs/reference/inventory.md](docs/reference/inventory.md).

## Repository Structure

### Primary Surfaces

The repository is organized into distinct surfaces, each with specific responsibilities:

| Surface | Purpose | Documentation |
| :--- | :--- | :--- |
| **src/** | Core source modules implementing functionality | [src/README.md](src/README.md) |
| **scripts/** | Maintenance and automation utilities | [scripts/README.md](scripts/README.md) |
| **docs/** | Project documentation (about Codomyrmex) | [docs/README.md](docs/README.md) |
| **tests/** | Test suites (unit and integration) | [tests/README.md](tests/README.md) |
| **config/** | Configuration templates and examples | [config/README.md](config/README.md) |
| **projects/** | Project workspace, templates, and external adapter integrations (`daf-consulting`, `hermes-paperclip-adapter`) | [projects/README.md](projects/README.md) |
| **src/codomyrmex/examples/** | Executable examples and demos | [src/codomyrmex/examples/README.md](src/codomyrmex/examples/README.md) |
| **scripts/sair/** | SAIR Mathematics Distillation submodule | [scripts/sair/README.md](scripts/sair/README.md) |

## Key Files

- `README.md` - Primary entry point for users and contributors
- `AGENTS.md` - This file: agent coordination and navigation
- `scripts/rasp_gap_report.py` — regenerates [docs/plans/agents-readme-gap-report.md](docs/plans/agents-readme-gap-report.md) (scoped `AGENTS.md` / `README.md` presence under `src/codomyrmex/`, `docs/`, `projects/`, `scripts/`, `config/`, `.github/`; see script docstring for excludes)
- `scripts/doc_inventory.py` — prints repo doc metrics (module counts, workflows, optional pytest collect); output summarized in [docs/reference/inventory.md](docs/reference/inventory.md)
- `LICENSE` - MIT License
- `SECURITY.md` - Security policies and vulnerability reporting
- `pyproject.toml` - Python package configuration
- `pyproject.toml` (`[tool.pytest.ini_options]`, `[tool.coverage.*]`) — pytest and coverage configuration
- `Makefile` - Primary build and automation tasks (`make test` applies the 40% coverage gate)
- `justfile` - Optional [just](https://github.com/casey/just) recipes mirroring common Makefile targets
- `index.html` - Root redirect to `/output/website/index.html` for static hosting entry
- `uv.lock` - Python dependency lock file
- `start_here.sh` - Interactive entry point for exploration
- `package.json` - Root Node.js config (Playwright, docs scripts); Bun lockfiles live next to Bun projects (e.g. `src/codomyrmex/pai_pm/server/bun.lock`)

## Dependencies

- All dependencies are managed via `uv` (for Python) and `npm`/`yarn` (for JS/TS).
- See `pyproject.toml` and `package.json` for explicit version constraints.
- No direct dependencies between modular layers are permitted without interface contracts.

## Development Guidelines

- **Zero-Mock Policy:** All tests must use real components. No mocks. Narrow `monkeypatch.setenv`/`delenv`/`chdir` and `tmp_path` fixtures are permitted for test-input isolation — see [docs/development/testing-strategy.md § Zero-Mock Policy (clarified)](docs/development/testing-strategy.md#zero-mock-policy-clarified) and [issue #175](https://github.com/docxology/codomyrmex/issues/175).
- **Coverage Gate:** **40%** line coverage (`[tool.coverage.report] fail_under` in `pyproject.toml`). CI and `make test` pass `--cov-fail-under=40`; plain `uv run pytest` skips coverage for speed. Experimental `meme/` is omitted from `[tool.coverage.run]` (see `pyproject.toml`). New work must not drop below the floor when measured with `--cov`.
- **Documentation:** Maintain `AGENTS.md`, `README.md`, and `SPEC.md` parity on structural changes.
- **Generated leaf docs:** Thousands of per-folder `AGENTS.md` / `README.md` files are produced by tooling, not by hand.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [docxology/codomyrmex](https://github.com/docxology/codomyrmex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
