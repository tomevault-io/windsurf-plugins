---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
just install        # Update lock file and sync all extras + lint group
just lint           # Format and lint (eof-fixer, ruff format, ruff check --fix, ty check)
just lint-ci        # CI lint in check-only mode (no auto-fix)
just test           # Run pytest with coverage
just test -- -k "test_name"  # Run a single test
just test-branch    # Run tests with branch coverage
```

All commands use `uv run` — do not invoke tools directly (e.g., use `uv run pytest`, not `pytest`).

## Architecture

**lite-bootstrap** bootstraps Python microservices with pre-configured observability instruments.

### Core pattern

```
BaseConfig (frozen dataclass, kw_only)
    └── Framework configs compose multiple instrument configs via multiple inheritance

BaseInstrument[ConfigT] (generic, non-frozen dataclass with slots)
    └── Instrument subclasses: lifecycle via bootstrap() / teardown(); skip check via is_configured() classmethod

BaseBootstrapper (abc.ABC)
    ├── FastAPIBootstrapper
    ├── LitestarBootstrapper
    ├── FastStreamBootstrapper
    ├── FastMcpBootstrapper
    └── FreeBootstrapper
```

### Key design decisions

Recent design context, bugs, and convention rationale: see `planning/specs/2026-*-bug-audit-*.md` (audits + retros).

- **Optional dependencies**: Each instrument checks for its optional package via `import_checker.py` (`importlib.util.find_spec`). Instruments are skipped silently if the package is absent. Optional packages are imported inside `if import_checker.is_X_installed:` blocks; static analyzers that don't model this guard will report spurious "possibly unbound" diagnostics — the project uses `ty` which handles the pattern correctly.
- **Instrument skip ordering**: `BaseBootstrapper.__init__` runs `instrument_type.is_configured(config)` first (silent skip if the user's config indicates the instrument shouldn't run — populates `bootstrapper.skipped_instruments: list[tuple[type, str]]`); then `check_dependencies()` (emits `InstrumentDependencyMissingWarning` only for configured-but-dep-missing — the genuine deployment surprise); then instantiates. One `logger.info` summary line at the end lists configured + skipped instruments via `BaseBootstrapper.build_summary()`; that method is also publicly callable for post-construction debugging. Uses stdlib `logging` so it composes cleanly with the user's logging setup and with pytest's `caplog`.
- **Frozen configs, non-frozen instruments**: All `*Config` classes are `@dataclasses.dataclass(kw_only=True, frozen=True)`. All `*Instrument` classes lose `frozen=True` because two instruments (`LoggingInstrument`, `OpenTelemetryInstrument`) cache mutable runtime state (`_logger_factory`, `_tracer_provider`); Python's dataclass rules require the whole hierarchy to be non-frozen. `from_dict()` and `from_object()` filter unknown keys before constructing.
- **`FastAPIConfig.application` uses an `UnsetType` sentinel**: shared in `lite_bootstrap/types.py` as `UnsetType` + `UNSET` (singleton). `FastAPIConfig.__post_init__` checks `isinstance(self.application, UnsetType)` and replaces with a constructed `FastAPI()` via `object.__setattr__` (config stays frozen for user-facing immutability). A one-line comment in `__post_init__` documents the freeze bypass.
- **Instrument registry**: `BaseBootstrapper` holds a list of instrument instances; it calls `bootstrap()` on each in order and `teardown()` in reverse during shutdown.
- **Idempotent teardown**: `BaseBootstrapper.teardown()` returns immediately if `not self.is_bootstrapped`. Cached runtime state in `LoggingInstrument` and `OpenTelemetryInstrument` is reset inside `try/finally` so a raised shutdown leaves no stale references.
- **Logging ↔ Sentry integration**: `logging_instrument.py` injects structlog context into Sentry events. `sentry_instrument.py` chains `before_send` callbacks via `wrap_before_send_callbacks()`. The `skip_sentry` flag in log context suppresses events; the flag is also stripped from the Sentry context payload (added to `IGNORED_STRUCTLOG_ATTRIBUTES`).
- **OTel ↔ Logging integration**: The logging instrument injects span/trace IDs from the active OpenTelemetry context into every log record.
- **`OpenTelemetryInstrument` is single-instance per process**: `bootstrap()` calls `opentelemetry.trace.set_tracer_provider(...)`, which the OTel SDK enforces as set-once via `_TRACER_PROVIDER_SET_ONCE.do_once(...)` (subsequent calls log `"Overriding of current TracerProvider is not allowed"` and have no effect). `teardown()` calls `shutdown()` on the provider (flushes batched spans, closes exporters) but cannot reset the process-global pointer. Construct one `OpenTelemetryInstrument` per process; do not bootstrap a second instance. Verified against `opentelemetry/trace/__init__.py:548-556`.

### Module layout

One file per instrument under `lite_bootstrap/instruments/`, one per framework under `lite_bootstrap/bootstrappers/`. Non-obvious files worth knowing:

- `lite_bootstrap/types.py` — `UnsetType` + `UNSET` singleton used as the "user did not supply this" sentinel (notably for `FastAPIConfig.application`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [modern-python/lite-bootstrap](https://github.com/modern-python/lite-bootstrap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
