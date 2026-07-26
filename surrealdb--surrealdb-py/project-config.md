---
trigger: always_on
description: Authoritative instructions for agentic coding work in this repository. Scope: entire repo unless a deeper AGENTS.md overrides it.
---

# AGENTS

Authoritative instructions for agentic coding work in this repository. Scope: entire repo unless a deeper AGENTS.md overrides it.

## Golden rules
- Default to `uv` for everything (install, run, build).
- Prefer smallest-scope checks: format → lint → type check → targeted tests → full suite.
- Keep changes minimal and idiomatic to existing patterns; avoid drive-by refactors.
- Do not add inline comments unless a test/CI rule requires them.
- Preserve public API surface and error semantics unless explicitly requested.
- When unsure about SurrealDB version behavior, prefer configurable or version-agnostic solutions.

## Repo map
- Library source: `src/surrealdb/` (Python, Rust extension bindings under `surrealdb_ext`).
- Tests: `tests/` (pytest, pytest-asyncio, hypothesis); see `tool.pytest.ini_options` in `pyproject.toml`.
- Scripts: `scripts/` (`build.sh`, `checks.sh`, `tests.sh`).
- Examples: `examples/` directories for frameworks; keep untouched unless asked.
- Rust extension: `surrealdb_ext/` (maturin build target `surrealdb._surrealdb_ext`).

## Environment setup
- Install uv: `curl -LsSf https://astral.sh/uv/install.sh | sh`.
- Install runtime deps: `uv sync`.
- Install with dev tools: `uv sync --group dev` (required before running checks).
- Virtualenv activation is handled by uv; do not add venv folders to repo.

## Build commands
- Python wheel: `uv build`.
- Full wheel via maturin: `uv run maturin build --release --out dist/` (see `scripts/build.sh`).
- Dev install of extension: `uv run maturin develop --release`.

## Test commands (pytest)
- Run all: `uv run pytest`.
- Run all with coverage: `uv run pytest --cov=src/surrealdb --cov-report=term-missing --cov-report=html`.
- Single file: `uv run pytest tests/unit_tests/data_types/test_geometry.py`.
- Single test node: `uv run pytest tests/unit_tests/data_types/test_geometry.py::test_point`.
- Filter by keyword: `uv run pytest -k "keyword"`.

## Test strategy expectations
- Prefer smallest relevant scope (node/file) before whole suite.
- Respect `tool.pytest.ini_options` markers/strictness; avoid `xfail`/`skip` unless necessary and justified.

## Linting and formatting (ruff)
- Check lint: `uv run ruff check src/` (configured with `I`, `UP`).
- Apply safe lint fixes: `uv run ruff check src/ --fix` (configured with `I`, `UP`).
- Check formatting: `uv run ruff format --check --diff src/`.
- Apply formatting: `uv run ruff format src/`.
- Keep imports auto-sorted; avoid manual reordering unless required.

## Type checking
- mypy (library): `uv run mypy --explicit-package-bases src/`.
- mypy (tests relaxed overrides): `uv run mypy --explicit-package-bases tests/`.
- pyright strict: `uv run pyright src/`.
- Honor strict settings (no implicit optional, no untyped defs, no Any generics) unless explicitly suppressed by config.

## Combined check scripts
- Quick all-checks: `./scripts/checks.sh` (uv sync + ruff check/format check + mypy src/tests + pyright).

## SurrealDB version testing
- Start local server for integration: `docker-compose up -d`.
- Default version is latest stable; to target specific version: `SURREALDB_VERSION=v2.1.8 uv run scripts/run_tests.sh`.
- For many versions: `./scripts/test-versions.sh v2.1.8` or set `SURREALDB_VERSION` env.

## Code style – Python
- Use type hints everywhere (functions, methods, module-level constants when helpful).
- Prefer explicit `ValueError`/`TypeError` for invalid inputs; use `SurrealDBMethodError` where appropriate.
- Keep functions small and composable; avoid side effects in helpers.
- Favor early returns over deep nesting.
- Preserve synchronous vs asynchronous separation (blocking vs async connection classes). Do not mix event loops in blocking code.
- Respect public API surface exported in `src/surrealdb/__init__.py`; avoid new exports without justification.
- Imports: standard library → third-party → local; rely on ruff isort.
- String formatting: prefer f-strings; avoid `%` formatting.
- Constants and enums live in `surrealdb.data.types.constants`; reuse them instead of duplicating literals.
- Serialization/deserialization lives under `surrealdb.cbor` and `surrealdb.request_message`; follow existing patterns.
- Maintain immutability where present (e.g., data type objects); avoid mutating input parameters.
- Keep exception messages actionable and user-facing (state what to supply instead of only saying "invalid").
- Avoid adding global state; prefer dependency injection through parameters.
- Follow pytest naming (`test_*` functions, `Test*` classes) per config.

## Code style – Rust extension (`surrealdb_ext`)
- Build with `cargo fmt` defaults; do not change edition.
- Follow existing module layout (`async_db.rs`, `sync_db.rs`, `lib.rs`).
- Keep public surface in sync with Python bindings; avoid breaking signatures.
- Prefer `Result<T, surrealdb::Error>` patterns and early `?` propagation; keep error messages concise.
- When touching pyo3 code, ensure `pyo3/extension-module` feature remains enabled as in `pyproject.toml`.

## Naming conventions
- Modules and functions: snake_case.
- Classes and exceptions: PascalCase.
- Constants: UPPER_SNAKE (see `constants.py`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [surrealdb/surrealdb.py](https://github.com/surrealdb/surrealdb.py) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
