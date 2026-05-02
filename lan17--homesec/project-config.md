---
trigger: always_on
description: **Last reviewed:** 2026-04-25
---

# HomeSec Development Guidelines

**Last reviewed:** 2026-04-25
**Purpose:** Critical patterns to prevent runtime bugs when extending HomeSec. For architecture overview, see `DESIGN.md`.

---

## Absolute Rules

- **Strict type checking required**: Run `make typecheck` before committing. Error-as-value pattern requires explicit type narrowing via match/isinstance.
- **Program to interfaces**: Use factory/registry helpers (e.g., `load_filter_plugin()`). Avoid direct instantiation of plugins.
- **Architecture constraints**: See `DESIGN.md` → “Architecture Constraints”. Boundary violations (core ↔ concrete plugins, backend-specific config in core) are bugs.
- **Repository pattern**: Use `ClipRepository` for all state/event writes. Never touch `StateStore`/`EventStore` directly.
- **Preserve stack traces**: Custom errors must set `self.__cause__ = cause` to preserve original exception.
- **Tests must use Given/When/Then comments**: Every test must include these comments and follow behavioral testing principles (see `TESTING.md`).
- **Run `make check` before pushing code or opening/updating a PR**: Targeted checks are not enough for publish flow unless the user explicitly asks for a narrower validation scope.
- **PR titles must satisfy repo CI**: Use semantic PR titles such as `fix: ...`, `refactor: ...`, or `docs: ...`. Do not use `[codex] ...` in this repository because CI rejects non-semantic PR titles.
- **Respect `uv.lock`**: Validation commands run with `--locked`. If `uv` reports the lockfile is stale, run `uv lock`, inspect the diff, and commit the lockfile sync intentionally. Do not revert `uv.lock` churn without checking whether `pyproject.toml` or dependency metadata changed.
- **Import from canonical source**: Import types from their defining module, not re-exports. For example, import `RiskLevel` from `models.enums`, not from `models.vlm`. Avoid creating re-exports in `__all__`.
- **Postgres for state**: Use `clip_states` table with `clip_id` (primary key) + `data` (jsonb) for evolvable schema.
- **Pydantic everywhere**: Validate config, DB payloads, VLM outputs, and MQTT payloads with Pydantic models.
- **Clarify before complexity**: Ask user for clarification when simpler design may exist. Don't proceed with complex workarounds.
- **Product priorities**: Recording + uploading (P0) must work even if Postgres is down. Analysis/notifications are best-effort (P1).

---

## Logging Guidance

**Rule:** Use regular Python logging for operational logs, and structured extras for queryable telemetry. Logging is best-effort observability; use `ClipRepository` for workflow state and durable events.

**Structured logging conventions:**
- Use normal log levels (`debug`, `info`, `warning`, `error`) for severity.
- Put queryable context in `extra={...}` instead of packing metadata into the message string.
- For telemetry events, include a stable low-cardinality `event_type` such as `vlm_usage`, `recording_started`, or `upload_failed`.
- Treat `event_type` as the canonical event marker. DB filtering and payload shaping classify any truthy `event_type` as `kind="event"`.
- `kind="event"` is optional when `event_type` is present. Never set `kind="log"` on a record that has `event_type`.
- Do not manually add standard `LogRecord` fields such as `message`, `asctime`, `levelname`, `pathname`, or `lineno` in `extra`.
- Prefer the existing logging context injection for `camera_name` and `recording_id`; only pass them explicitly when the caller is outside that context and has accurate values.
- Never log secrets, tokens, credentials, or full DSNs. Log secret source names such as env var names when needed.

**Template: Telemetry Event Log**

```python
logger.info(
    "VLM usage recorded",
    extra={
        "event_type": "vlm_usage",
        "camera_name": camera_name,
        "recording_id": clip_id,
        "input_tokens": usage.input_tokens,
        "output_tokens": usage.output_tokens,
        "total_tokens": usage.total_tokens,
    },
)
```

**Template: Operational Log**

```python
logger.warning(
    "Upload failed; continuing without storage URI",
    extra={"recording_id": clip_id, "storage_backend": backend},
    exc_info=exc,
)
```

---

## Core Pattern Templates

### 1. Error-as-Value + Type Narrowing

**Rule:** For partial failures, return `Result | ErrorType` instead of raising exceptions. Always narrow types with match or isinstance.

**✅ Template: Error-as-Value with Type Narrowing**

```python
# Define custom error that preserves stack traces
class FilterError(PipelineError):
    def __init__(self, clip_id: str, plugin_name: str, cause: Exception):
        super().__init__(f"Filter failed for {clip_id}", stage="filter", clip_id=clip_id)
        self.plugin_name = plugin_name
        self.__cause__ = cause  # ← Preserves full stack trace

# Stage returns error as value, not raise
async def _filter_stage(self, clip: Clip) -> FilterResult | FilterError:
    try:
        async with self._sem_filter:
            return await self._filter.detect(clip.local_path)
    except Exception as e:
        return FilterError(clip.clip_id, self._config.filter.backend, cause=e)

# Caller uses match for type narrowing
filter_result = await self._filter_stage(clip)

match filter_result:
    case FilterError() as err:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lan17/homesec](https://github.com/lan17/homesec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
