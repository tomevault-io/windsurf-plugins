---
trigger: always_on
description: A modern circuit breaker for Python: sync + async in a single class, sliding-window rate,
---

# AGENTS.md

A modern circuit breaker for Python: sync + async in a single class, sliding-window rate,
slow-call detection, a type-safe API and transparent transport-level integrations.
This is a foundational library — people put it on the critical path of their services, so the
bar for simplicity, reliability and dependency hygiene is higher than usual.

## Decision-making principles

- Start from the problem: understand it first, pick tools second.
- Occam's razor: no new entities or abstractions without necessity.
- Never break existing functionality while making a change.
- A solution must be simple, reliable and readable without extra explanation.
- Minimise cognitive load — for the human and for the next agent.
- The upsides of a decision must heavily outweigh the downsides. When combining approaches, take
  the best of each.
- Write docstrings in English.

## Technology

- **Python 3.11+** — the minimum supported version (`asyncio.timeout`, `TaskGroup`, exception
  groups, `Self` are required). Development and CI run on 3.11–3.14. The floor is deliberately
  below the usual 3.12 default: a foundational library values reach.
- **Zero-dependency core** — stdlib only. A fault-tolerance tool must not depend on someone
  else's reliability. Everything external goes through optional extras
  (`interlock-cb[httpx2]`, `[httpx]`, `[aiohttp]`, `[requests]`, `[tenacity]`, `[fastapi]`, `[litestar]`, `[redis]`, `[otel]`).
- **Pydantic ≥ 2.0 — extras only, NEVER in the core.** The core config is a frozen dataclass with
  eager validation. Pydantic is acceptable only where it is optional.
- **uv** — package manager.
- **hatchling** — build backend. The version is **static** in `interlock/version.py`
  (`[tool.hatch.version] path`), bumped manually on release and exposed as `interlock.__version__`.
- **ruff** — formatting and linting.
- **mypy + pyright + pyrefly** in strict mode — static analysis.
- **pytest** + `pytest-asyncio`, `pytest-cov`, `pytest-mock`, `pytest-sugar`, `faker`, `hypothesis`.
- **mutmut** — mutation testing over the state machine and the engine; out of band, never a PR gate.
- **zizmor** — static audit of the GitHub Actions workflows; a PR gate like ruff and mypy.
- **griffe** — public-API breakage detection against the latest release tag; a PR gate,
  overridable with the `breaking-change` label for intentional changes.
- **Zensical** — documentation (plain-Markdown content, portable).

## Environment and commands

- Virtual environment: `uv venv --python 3.12 .venv`
- Run commands: `uv run <command>`
- Dependencies: `uv add <package>` / dev: `uv add --dev <package>`
- Format: `uv run ruff format`
- Lint: `uv run ruff check --fix`
- Types: `uv run mypy`, `uv run pyright` and `uv run pyrefly check`
- Tests: `uv run pytest` / with coverage: `uv run pytest --cov`
- Workflows: `uv run zizmor .github/workflows/` (export `GH_TOKEN` for the online audits)
- Public API: `uv run griffe check interlock --search .` (diffs against the latest release tag)
- Mutants: `uv run mutmut run` (optional, out of band — see `CONTRIBUTING.md`)

ruff is configured in `pyproject.toml` (`line-length = 100`, `target-version = "py311"`), not via
CLI flags.

## Architectural rules

- **The core is an I/O-free state machine.** State, window, thresholds and transitions do no I/O
  and know nothing about sync vs async. Around the await-free critical section sits a single
  `threading.Lock` (correct for both threads and the event loop); the lock is never held across
  the protected call.
- **Extension points are `Protocol`s, not inheritance of internal classes:**
  `Clock`, `SlidingWindow`, `Storage`, `FailureClassifier`, `EventListener`.
- **One public `CircuitBreaker` class for sync and async.** No `Sync*`/`Async*` twins. The class
  detects a coroutine function and dispatches to the right path.
- **Time only through the injected `Clock`.** No direct `time.monotonic()` / `sleep` in logic:
  it breaks test determinism.
- **Group by feature, not by kind** (no `models/`, `services/`, `utils/`). Each concept — window,
  state machine, classifier — gets its own module.
- **The public API goes through the package `__init__.py`.** Helpers are underscore-prefixed and
  hidden.
- **Encapsulate external dependencies** behind wrappers; extras never leak into the core.

## Code style (Python)

### Formatting

- Maximum line length: 100 characters.
- Single quotes for strings.
- f-strings instead of `.format()` and `%`.
- `pathlib.Path` instead of `os.path`.
- Context managers instead of `try/finally`.

### Imports

- Always at the top of the file.
- Absolute imports only.
- Group order: stdlib → third-party → local (blank line between groups).

### Typing

- Annotations on every parameter and return value.
- Modern syntax: `list[str]`, `dict[str, int]`, `str | None`.
- Never `Optional[X]` — only `X | None`.
- No magic constants — use `StrEnum` or module-level constants.

### Calls and constructors

With 3+ arguments, keyword arguments only.

```python
# bad
config = Config(0.5, 10, 60.0)

# good
config = Config(failure_rate_threshold=0.5, minimum_number_of_calls=10, wait_duration_in_open=60.0)
```

### Functions

- One function — one job.
- 20–30 lines at most.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bagowix/interlock](https://github.com/bagowix/interlock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
