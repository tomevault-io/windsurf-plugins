---
trigger: always_on
description: AIPerf project rules and coding standards
---


<!--
SPDX-FileCopyrightText: Copyright (c) 2025-2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
SPDX-License-Identifier: Apache-2.0
-->

# AIPerf

Python 3.10+ async AI benchmarking tool for measuring LLM inference server performance. 10 services communicate via ZMQ message bus.

**Reference documentation:**
- [`docs/architecture.md`](docs/architecture.md) - Three-plane architecture, core components, credit system, data flow, communication patterns
- [`docs/dev/patterns.md`](docs/dev/patterns.md) - Code examples for CLI commands, services, models, messages, plugins, error handling, logging, testing
- [`docs/cli-options.md`](docs/cli-options.md) - Complete CLI command and option reference
- [`docs/environment-variables.md`](docs/environment-variables.md) - All `AIPERF_*` environment variables by subsystem
- [`docs/metrics-reference.md`](docs/metrics-reference.md) - Metric definitions, formulas, and requirements
- [`docs/plugins/plugin-system.md`](docs/plugins/plugin-system.md) - Plugin architecture, categories, creation guide
- [`CONTRIBUTING.md`](CONTRIBUTING.md) - Development setup, available commands, pre-commit hooks, DCO

## Coding Standards

- async/await for ALL I/O - no `time.sleep`, no blocking calls.
- `Field(description="...")` on EVERY Pydantic field. Docstrings on dataclass fields.
- Type hints on ALL functions (params and return).
- KISS + DRY: minimal code, optimize for reader.
- `AIPerfBaseModel` for data, `BaseConfig` for configuration. `@dataclass(slots=True)` for hot-path inner models created at high volume (e.g. SSE chunks, parsed responses) where Pydantic overhead matters. Use `__pydantic_config__ = ConfigDict(extra="forbid")` on dataclasses that participate in Pydantic union discrimination.
- `BaseComponentService` for services, `BaseService` for SystemController only.
- Message bus for inter-service communication - no shared mutable state.
- CLI commands: one file per command in `cli_commands/`, lazily loaded via import strings in `cli.py`. See `docs/dev/patterns.md`.
- YAML plugin registry for extensible features (`src/aiperf/plugin/plugins.yaml`).
- Lambda for expensive logs: `self.debug(lambda: f"{self._x()}")`. Direct string for cheap ones.
- Always `orjson.loads(s)`, `orjson.dumps(d)` for JSON.
- No `Optional[X]` or `Union[X, Y]` - use `X | Y`.
- Comments only for "why?" not "what".
- Enums are string-based - use `MessageType.X` directly, never `.value`.
- Dependencies: always use `uv` (never pip) - `uv add package`, `uv run pytest`.
- Use mermaid diagrams instead of ASCII art in markdown files.
- Do not create markdown files to document code changes or decisions.
- Do not over-comment code. Removing code is fine without adding comments to explain why.
- No emojis in code or comments.
- Hide a metric from the console table with `console_group = MetricConsoleGroup.NONE`; group it into a separate section with `MetricConsoleGroup.{USAGE,CACHE,PREDICTION,AUDIO,REASONING}`. Default is `DEFAULT`. See `docs/metrics-reference.md` "Metric Console Group Reference".

## NaN/Inf Discipline

Numeric metric values crossing a serialization boundary or feeding a numerical algorithm must be finite or explicitly `None`. Use `aiperf.common.finite` (`FiniteFloat`, `scrub_non_finite`, `nan_safe_mean`/`std`, `is_finite_value`). Mechanical CI invariants in `tests/unit/property/test_finite_invariants.py` reject new violations and ratchet existing debt to zero via baseline files. See [`docs/dev/patterns.md`](docs/dev/patterns.md) § "NaN/Inf Discipline Pattern" and [`docs/dev/global-invariants.md`](docs/dev/global-invariants.md) for the full contract.

## Build and Test Commands

```bash
make first-time-setup                                      # Initial environment setup
make install                                               # Install project + mock server
uv run pytest tests/unit/ -n auto                          # Unit tests (fast, isolated)
uv run pytest -m integration -n auto                       # Integration tests (real services, multiprocess)
uv run pytest -m component_integration -n auto             # Component integration tests (single process)
ruff format . && ruff check --fix .                        # Format and lint
make validate-plugin-schemas                               # Validate plugin registry
pre-commit run                                             # Pre-commit on staged files
pre-commit run --all-files                                 # Pre-commit on all files
make generate-all-docs                                     # Regenerate CLI + env var docs
make generate-all-plugin-files                             # Regenerate plugin enums, overloads, schemas
```

## Pre-Commit Hooks

Run pre-commit after every code change, even before creating commits:

```bash
pre-commit run              # Staged files only
pre-commit run --all-files  # All files (recommended after significant changes)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ai-dynamo/aiperf](https://github.com/ai-dynamo/aiperf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
