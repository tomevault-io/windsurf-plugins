---
trigger: always_on
description: Defaults for agent-assisted Python projects managed with `uv`. Required rules apply throughout; Preferred Stack entries apply only when that capability is needed.
---

# AGENTS Guidelines (Python)

Defaults for agent-assisted Python projects managed with `uv`. Required rules apply throughout; Preferred Stack entries apply only when that capability is needed.

## Priority

When instructions conflict, follow:

1. Safety and security constraints
2. User request
3. This file
4. Existing repo patterns

Briefly flag contradictions to the user, naming the conflicting instructions, the priority applied, and any effect on the task.

## Required Process

- **Scope**: Work only inside this repo and change only task-relevant files. Do not alter unrelated code, comments, or logic; report unrelated issues separately.
- **Approvals**: Ask before adding dependencies, installing packages, fetching external resources, or calling external services. `uv sync` is pre-approved only for dependencies already declared here.
- **Assumptions**: Surface inconsistencies, ambiguity, and risky trade-offs; ask when clarification is necessary. Challenge unsafe or unsound requests with concrete alternatives.
- **Simplicity**: Prefer small, explicit, maintainable changes that satisfy the request without over-engineering.
- **Cleanup**: Remove dead code, temporary files, and development artifacts created by the work. Release resources with context managers or appropriate cleanup handlers.
- **Artifacts**: Keep large datasets, generated outputs, and caches out of git unless intentionally versioned.
- **Before finishing**: Confirm assumptions, simplicity, scope, validation, and any trade-offs worth reporting.
- **Durable memory**: Record non-obvious root causes, failed approaches, confirmed constraints, refactoring risks, and decisions in `NOTES.md`. Track open tasks, questions, deferred ideas, and follow-ups in `PLAN.md`. Update either file only when the information is durable and useful; avoid step logs.

## Security

- Treat security as a design constraint. Minimize data, network, permission, and dependency exposure.
- Never hardcode or commit secrets, tokens, credentials, private keys, personal data, or sensitive operational data. Store secrets in `.env`, load them with `python-dotenv`, and redact sensitive values from logs, errors, fixtures, documentation, and examples.
- Validate and constrain untrusted inputs at boundaries: CLI args, config files, uploaded files, HTTP responses, model outputs, scraped content, and user-provided paths.
- Do not use `eval`, `exec`, unsafe deserialization, unchecked downloads, or dynamic imports on untrusted input. Prevent shell injection and path traversal.
- Before adding dependencies or external services, consider supply-chain risk, maintenance status, license fit, and whether the standard library or an existing dependency is enough.
- If you find a security issue, stop broad changes, document the risk, and propose the smallest focused fix. Keep security changes separate from unrelated refactors.

## Python and Dependency Management

- Prefer Python 3.13 for new projects when dependencies permit. Preserve the supported range declared in `pyproject.toml` unless the task explicitly changes it.
- Use `uv` exclusively for Python environments, dependencies, and commands. Do not run `python`, `pip`, `pytest`, or `ruff` directly or create environments with other tools.
- Use `uv sync`, `uv add [--dev] <package>`, `uv sync --upgrade-package <package>`, and `uv run <command>`. Run `uv sync` after dependency or lockfile changes.
- Import only packages declared in `pyproject.toml`. Put runtime dependencies in
  `[project].dependencies` and development tools in `[dependency-groups].dev`.
- Keep `[tool.uv] exclude-newer = "7 days"` so dependency resolution uses packages at least seven days old.
- Preferred-stack packages below are recommendations, not available code, until declared.
- For new projects, prefer a `src/<project_name>/` layout with tests under `tests/`; preserve an established layout unless migration is in scope. Separate unit and integration tests where useful, and define operational scripts in `[project.scripts]`.
- Install repository pre-commit hooks with `uv run pre-commit install` before the first commit.

## Python Standards and Style

- **Types**: Use modern syntax: `list[str]`, `X | None`, `Self`; no `typing.List`.
- **Data**: Use `dataclasses` or `TypedDict`.
- **Paths**: Use `pathlib.Path`.
- **Errors**: Raise/catch specific exceptions with clear messages; no bare `except:`.
- **Formatting**: Use f-strings, including `f"{var=}"` when useful.
- **Public/non-trivial APIs**: Add type hints.

## Configuration

- Put operator-tunable runtime settings in `config.yaml`, loaded with `pyyaml` when needed: model names, temperatures, token limits, timeouts, retries, endpoints, paths, feature flags, and thresholds. Keep implementation constants and invariants in code.
- Keep package, lint, format, and test configuration in `pyproject.toml` or the tool's native config file, not `config.yaml`.

## Logging

Use `logging` with JSON output containing timestamp, level, message, logger/module, and exception details when present.

## Architecture


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [machinelearningZH/simply-simplify-language](https://github.com/machinelearningZH/simply-simplify-language) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
