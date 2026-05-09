---
trigger: always_on
description: Guidelines for agent-assisted development in Python projects managed with `uv`.
---

# AGENTS Guidelines (Python)

Guidelines for agent-assisted development in Python projects managed with `uv`.

This file defines project defaults for repositories created from this template. Follow the required rules for all work in this repo; treat stack sections as preferred choices only when the project needs that capability.

## Instruction Priority

When instructions conflict, follow this order:

1. User request
2. Safety and security constraints
3. Existing repo patterns
4. This file

## Process & Security

- **Scope**: Work only in this repo.
- **Approvals**: **ALWAYS** ask before adding dependencies, installing packages, fetching external resources, or calling external services. Routine `uv sync` is allowed when it only installs dependencies already declared in this repo.
- **Secrets**: Store in `.env`, load with `python-dotenv`. Never hardcode or commit secrets.
- **Assumptions**: Surface inconsistencies, unclear intent, and ambiguous requirements; ask for clarification before proceeding. Push back on bad ideas and present trade-offs when relevant.
- **Simplicity**: Favor simple, explicit, maintainable solutions that meet requirements without over-engineering.
- **Scope Discipline**: Only modify code directly related to the current task. Never change, move, or remove unrelated code, comments, or logic, even if it seems wrong or unclear. Flag unrelated issues separately.
- **Cleanup**: Remove dead code, temporary files, and dev artifacts after each step. Release resources (files, connections, temp dirs) with `with` context managers or `atexit` for global cleanup.
- **Self-Check Before Finishing**: Before presenting a solution, verify: (1) Did I make assumptions I should have clarified? (2) Is this the simplest solution? (3) Did I change unrelated code? (4) What alternatives or trade-offs should I mention?
- **Save durable findings in NOTES.md**: Capture non-obvious insights that should not be rediscovered later: root causes, failed approaches, confirmed constraints, refactoring risks, and important decisions. Do not add noisy step-by-step logs.
- **TODOs go in PLAN.md**: Keep actionable next steps, open tasks, follow-up ideas, deferred work, and open questions in this file. Treat `PLAN.md` as the working store for what should happen next, not `NOTES.md`. If either file is needed and does not exist yet, create it.

## Security First

- Treat security as a design constraint, not a final checklist. Prefer the smallest safe data access, network access, file permissions, and dependency surface that meet the task.
- Never commit secrets, tokens, credentials, private keys, raw personal data, or sensitive operational data. Redact them from logs, errors, fixtures, docs, and examples.
- Validate and constrain untrusted inputs at boundaries: CLI arguments, config files, uploaded files, HTTP responses, model outputs, scraped content, and user-provided paths.
- Avoid unsafe execution patterns: no `eval`, `exec`, shell injection, unsafe deserialization, path traversal, unchecked downloads, or dynamic imports from untrusted input.
- Before adding dependencies or external services, consider supply-chain risk, maintenance status, license fit, and whether the standard library or an existing dependency is enough.
- If you find a security issue, stop broad changes, document the risk, and propose the smallest focused fix. Do not bury security-relevant changes inside unrelated refactors.

## Environment and Execution (`uv` Only)

Use `uv` for all Python environment management and local Python execution. Do not run or manage Python code with Pylance tools, manually created virtual environments, `venv`, `virtualenv`, Conda, Poetry, Pipenv, direct `python`, direct `pip`, direct `pytest`, direct `ruff`, or any other Python runner or package manager.

Use the latest stable Python version that package compatibility allows. Set the supported version range in `pyproject.toml`; this template starts at Python 3.12+.

Allowed patterns:

```bash
uv sync                 # Lock/Sync
uv add [--dev] <pkg>    # Add dependency
uv run <cmd>            # Run in env
```

Required examples:

```bash
uv run python -m ai_project_template.main
uv run pytest
uv run ruff check .
uv run ruff format .
```

After dependency or lockfile changes, run `uv sync`. If a tool or assistant offers Python execution outside `uv`, translate it into an equivalent `uv run ...` command instead.

## Dependency Policy

- Do not import a package unless it is already declared in `pyproject.toml` or was first added with `uv add` / `uv add --dev` after approval.
- Keep runtime dependencies in `[project].dependencies` and development-only tools in `[dependency-groups].dev`.
- Keep `[tool.uv] exclude-newer = "7 days"` so dependency resolution uses packages at least seven days old, reducing supply-chain risk.
- If a package appears in the preferred stack below but is not declared yet, treat it as a recommendation, not as available code.

## Python Conventions

- **Types**: Modern syntax (`list[str]`, `X | None`, `Self`). No `typing.List`.
- **Data**: Use `dataclasses` or `TypedDict`.
- **Paths**: `pathlib.Path` only.
- **Errors**: Specific exceptions with messages. No bare `except:`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [machinelearningZH/hybrid-search-eval](https://github.com/machinelearningZH/hybrid-search-eval) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
