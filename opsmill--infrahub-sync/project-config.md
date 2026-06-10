---
trigger: always_on
description: `infrahub-sync` synchronizes data between infra sources and destinations (Infrahub, NetBox, Nautobot, etc.). It uses uv for packaging, a Typer CLI, and Invoke tasks for linting and docs. Examples live in `examples/`.
---


# LLM Context Guide for `infrahub-sync`

`infrahub-sync` synchronizes data between infra sources and destinations (Infrahub, NetBox, Nautobot, etc.). It uses uv for packaging, a Typer CLI, and Invoke tasks for linting and docs. Examples live in `examples/`.

## Agent Operating Principles

1. **Plan → Ask → Act → Verify → Record**
   Plan briefly, ask for missing context, act with the smallest change, verify locally, then record with a concise commit or PR note.

2. **Default to read-only and dry runs**
   Prefer `list`, `diff`, and `generate` before `sync`. Write/apply only with explicit instruction and human approval.

3. **Be specific and reversible**
   Use small, scoped commits. Do not mix large refactors with behavior changes in the same PR.

4. **Match existing patterns**
   Keep CLI, adapters, examples, and directory structure consistent with the codebase.

5. **Idempotency and safety**
   Favor operations that are safe to re-run. Use dry runs. Never print or guess secrets. Handle timeouts, auth, and network errors explicitly.

## Quickstart

```bash
# Setup
pyenv local 3.12.x || use system Python 3.10–3.13
uv sync

# Validate dev environment
uv run infrahub-sync --help
uv run infrahub-sync list --directory examples/

# Make a change, then:
uv run invoke format
uv run invoke lint
uv run infrahub-sync list --directory examples/

# If docs/CLI changed:
uv run invoke docs.generate
uv run invoke docs.docusaurus
```

## Required Development Workflow

Run these in order before committing.

```bash
uv sync
uv run invoke format
uv run invoke lint
```

`invoke lint` runs ruff → pylint → yamllint → ty.

**Policy:**

- New or changed code is Ruff-clean and typed where touched (docstrings, specific exceptions).
- The codebase is clean under ty with no `[[tool.ty.overrides]]` blocks. Don't reintroduce overrides to mask type errors — fix the underlying issue, or use a targeted `# ty: ignore[<rule>]` with a short TODO at the call site.
- If you add tests, run `uv run pytest -q`.

**CLI sanity after changes:**

```bash
uv run infrahub-sync --help
uv run infrahub-sync list --directory examples/
uv run infrahub-sync generate --name from-netbox --directory examples/
```

**Docs:** (only if user-facing changes)

```bash
uv run invoke docs.generate
uv run invoke docs.docusaurus
```

## Repository Structure

```text
infrahub-sync/
├─ infrahub_sync/                # Source
│  ├─ cli.py                     # Typer entrypoint
│  ├─ __init__.py                # Public API
│  ├─ utils.py                   # Utilities
│  ├─ potenda/                   # Core sync engine
│  └─ adapters/                  # NetBox/Nautobot/Infrahub adapters
├─ examples/                     # Example sync configs
├─ tasks/                        # Invoke task definitions
├─ docs/                         # Docusaurus (npm project)
├─ tests/                        # Unit and integration tests
├─ pyproject.toml                # uv + tool configs
└─ .github/workflows/            # CI
```

## Core Surfaces

- **Adapters** (`infrahub_sync/adapters/`): per-system connectors. Use existing ones as patterns.
    - Available: `infrahub`, `netbox`, `nautobot`, `aci`, `prometheus`, `peeringmanager`, `ipfabricsync`, `slurpitsync`, `genericrestapi`
- **Engine** (`infrahub_sync/potenda/`): orchestrates `list`, `diff`, `generate`, and `sync`.
- **Examples** (`examples/`): runnable configs and templates.

**CLI commands:**

- `infrahub-sync list` — show available sync projects.
- `infrahub-sync diff` — compute differences (safe).
- `infrahub-sync generate` — generate Python from YAML config (servers required).
- `infrahub-sync sync` — perform synchronization (servers and approval required).

## Configuration and Examples

- YAML config keys: `name`, `source`, `destination`, `order`.
- `source` and `destination` specify adapter names and connection settings.
- `order` defines the sync sequence of object types.
- Defaults often target `localhost`; adjust for real deployments.
- Credentials must come from environment or a secret manager. Never commit secrets.

## Code Standards

### Python (3.10–3.13)

- Prefer explicit types on new or changed code.
- Ruff: formatted and lint-clean. Honor `pyproject.toml`.
- Pylint: fix actionable issues in touched code; some warnings are expected.
- ty: included in `uv run invoke lint`; do not increase the error count. For an ad-hoc check, `uv run ty check .` works too.
- Public functions and classes require concise docstrings.
- Raise specific exceptions; avoid broad `except Exception:`.

### CLI and UX

- Predictable, idempotent commands with clear validation and errors.
- No secrets in logs or tracebacks.
- Prefer explicit flags over implicit behavior.

## Testing

If you introduce features or bug fixes, add targeted tests.

- Unit tests for `utils` and adapter edge cases (timeouts, 401/403, empty pages).
- Parametrized tests for config parsing.
- Mark network or integration tests and keep them opt-in (for example, `-m integration`).
- Keep tests atomic and single-purpose. Use parametrization rather than loops.

Run:

```bash
uv run pytest -q
```

## Documentation


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [opsmill/infrahub-sync](https://github.com/opsmill/infrahub-sync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
