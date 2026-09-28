---
trigger: always_on
description: Python library for structured LLM prompt engineering. Prompts are declared in YAML
---

# PromptKit

Python library for structured LLM prompt engineering. Prompts are declared in YAML
(Jinja2 template + typed input schema), validated with Pydantic, and executed through
a pluggable engine abstraction. Ships a Typer/Rich CLI.

Distribution name is `promptkit-core`; import name is `promptkit`.
License MIT. Public community project — treat every public symbol as a contract.

**PromptKit is at 1.0. The public API is frozen and semver applies.** Anything in
`promptkit.__all__` plus documented YAML fields is a contract; removing or changing it
needs a deprecation that warns for a full minor release first. Everything else is
internal.

[ARCHITECTURE.md](ARCHITECTURE.md) is the design. [MIGRATION.md](MIGRATION.md) records
how it was reached and what was deliberately deferred — read it before proposing
something that was already considered and rejected (a middleware pipeline, a richer
type-string language, tool calling).

## Layout

| Path | Role |
| --- | --- |
| `promptkit/errors.py` | Exception hierarchy — everything public raises from here |
| `promptkit/types.py` | `Message`, `Usage`, `Completion`, `Chunk`, `Capabilities` |
| `promptkit/events.py` | Lifecycle events and listener registration |
| `promptkit/cache.py` | `Cache` protocol, `MemoryCache`, `DiskCache`, key derivation |
| `promptkit/retry.py` | `RetryPolicy` and retryability rules |
| `promptkit/pricing.py` | Rate lookup over the vendored snapshot in `promptkit/data/` |
| `promptkit/core/` | `Prompt`, messages, YAML loader, sandboxed compiler, schemas, runner |
| `promptkit/core/template.py` | Confined Jinja loader and include-dependency resolution |
| `promptkit/core/jsonschema.py` | JSON Schema to Pydantic compilation |
| `promptkit/schemas/` | Published JSON Schema for prompt files |
| `promptkit/engines/` | `BaseEngine`, provider engines over official SDKs, plugin registry |
| `promptkit/core/structured.py` | JSON extraction, output validation, repair loop |
| `promptkit/utils/` | Token estimation, pricing tables, logging |
| `promptkit/cli/` | Typer app; one module per command under `cli/commands/` |
| `promptkit/evals/` | Eval cases, assertions, runner, reporters, cassettes |
| `promptkit/pytest_plugin.py` | pytest11 plugin collecting `*.evals.yaml` |
| `promptkit/core/registry.py` | Directory-backed, namespaced prompt registry |
| `promptkit/core/lint.py` | Lint rules PK001-PK009 |
| `examples/` | Runnable YAML prompts and a demo script |
| `docs/` | MkDocs Material, deployed to https://promptkit-core.ochotzas.com/ |
| `benchmarks/` | pytest-benchmark suites, run in CI |
| `promptkit/telemetry/` | OpenTelemetry listener behind the `otel` extra |
| `promptkit/codemod.py` | 0.1.x to 1.0 upgrade tool |
| `tests/` | Test suite, top-level and not shipped in the wheel |

## Commands

```
make install-dev     # editable install with dev extras + pre-commit hooks
make test            # pytest
make test-cov        # pytest with coverage, fails under COVERAGE_FLOOR (80)
make lint            # ruff check + ruff format --check
make format          # ruff check --fix + ruff format
make type-check      # mypy (strict = true)
make lock            # refresh uv.lock
make all             # format, lint, type-check, test
make docs            # build the MkDocs site (strict)
make serve-docs      # live-reload docs
make bench           # run benchmarks
```

## Code style

- **No comments. No docstrings.** Names and types carry the meaning. This applies to
  all new and modified code without exception. Do not reintroduce them when editing a
  file that still has them.
- Full type annotations on every function and method; mypy runs with
  `disallow_untyped_defs` and `disallow_incomplete_defs`.
- ruff for linting and formatting, line length 88. black and isort are gone — do not
  reintroduce them. Lint config lives in `[tool.ruff.lint]` in `pyproject.toml`; prefer
  a targeted `per-file-ignores` entry over a `# noqa` comment, since comments are banned.
- Python 3.10+ syntax is available (`X | None`, `list[str]`), but the codebase still
  mixes `typing.Optional` and `|`. Prefer the modern form in new code.
- Pydantic v2 only (`model_config`, `model_dump`, `model_post_init`).
- Errors raised out of the library must be types from `promptkit/errors.py`, never bare
  `ValueError` or third-party exceptions leaking from dependencies. Add to the hierarchy
  rather than raising something generic.
- Typer command help lives in `@app.command(help=..., epilog=...)`, not in docstrings —
  Typer reads docstrings for help text and the project has none.

## Conventions

- Prompt YAML requires `name`, `description`, and either `template` or `messages`.
  `input_schema` takes either type strings or a JSON Schema object; `output_schema`
  takes JSON Schema only.
- **The type-string language is frozen** at six scalars plus `X | None`. Do not extend
  it — richer types belong in JSON Schema. This is a deliberate boundary, and a PR
  adding generics or constrained forms should be turned down.
- `Prompt.template` is deprecated and warns. Internal code must use `joined_template`
  or `messages`; the suite runs `filterwarnings = ["error"]`, so a deprecated call
  inside the library fails tests.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ochotzas/promptkit-core](https://github.com/ochotzas/promptkit-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
