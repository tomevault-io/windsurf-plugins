---
trigger: always_on
description: NOTE: The canonical source for these instructions is .cursorrules in the repo root.
---

# AI Code Style Rules for this repository

NOTE: The canonical source for these instructions is .cursorrules in the repo root.
Any changes should be made there; this file syncs automatically via pre-commit hook.

# Style & Formatting
- Use Black/Ruff exactly as configured in pyproject.toml (line length 100, Python 3.14).
- Follow PEP 8 principles; Black/Ruff are authoritative for enforcement.
- Optimize for clarity and readability; prefer explicit types where helpful.
- Prefer `X | None` in type annotations; prefer union syntax in `isinstance()` checks (e.g., `int | str`) over tuples.
- Type Annotations:
  - Always add type hints to function/method parameters and return types
  - Use modern Python 3.10+ syntax: `X | None` instead of `Optional[X]`
  - In tests, annotate fixtures: `hass: HomeAssistant`, `caplog: pytest.LogCaptureFixture`
  - Add return types even for simple functions: `-> None`, `-> str`, `-> dict[str, Any]`

# Imports
- Organize imports consistent with Ruff isort settings.
  - known-first-party: ["custom_components"]
  - Do not force single-line imports.
  - Group order: standard library, third-party, first-party.
- Import ABCs (Callable, Iterable, etc.) from collections.abc, not typing.
- Use X | None instead of Optional[X] (modern Python 3.10+ syntax).

# Linting
- Enable Ruff rules: [E, F, I, B, UP] as configured in pyproject.toml.
- Respect per-file ignores from pyproject:
  - tests/**/*.py: [E501, F401, I001]

# License Headers
- All Python and YAML files MUST include the repository-standard license header at the top:
  - SPDX identifier and full Apache-2.0 header block used in this repo.
  - Python: hash-prefixed lines (e.g., `# SPDX-License-Identifier: Apache-2.0`, followed by the Apache block lines).
  - YAML: hash-prefixed lines at the top of the file.
  - Markdown: use an HTML comment block with the same content (recommended for docs like tests/README.md).
- Markdown files must use the lowercase `.md` extension (e.g., `README.md`).
- JSON does not support comments; do not attempt to add headers there.
- Tests are code; apply headers to test files (Python) as well.
- When creating or editing files, ensure the header is present and matches the canonical format already used in the repo.

# Comments
- Keep comments concise and only when adding non-obvious context.
- Explain complex logic, non-obvious patterns, or HA-specific requirements.
- Document type: ignore comments with justification for why they're needed.

# Home Assistant Specifics
- Only support HA 2026.3 and newer, no need for backward compatibility to older versions
- Use HA async patterns (`async_*` methods); avoid blocking I/O in the event loop.
  - Use hass.async_add_executor_job for any blocking operations.
- Prefer CoordinatorEntity for entities with push updates.
  - For push-based entities, set update_interval=None and push via coordinator.async_set_updated_data.
  - Use optimistic updates for user commands and reconcile on push ack to avoid UI snap-back.
- Multi-device model: coordinator data is dict[device_id, dict[str, int | str]], one entity per
  device; entity unique_id must use self._device_id consistently.
- Optimistic overlays: apply per-key overlays immediately; guard ~8s; confirm then clear;
  revert only on explicit failure.
- Runtime Data (Modern Pattern - HA 2026.2+):
  - Use ConfigEntry.runtime_data with TypedDict (not hass.data)
  - Define type alias for better IDE support and type checking:
    ```python
    class FanSyncRuntimeData(TypedDict):
        client: FanSyncClient
        coordinator: FanSyncCoordinator
    
    type FanSyncConfigEntry = ConfigEntry[FanSyncRuntimeData]
    
    async def async_setup_entry(hass, entry: FanSyncConfigEntry) -> bool:
        client = entry.runtime_data.client  # Full IDE autocomplete ✅
    ```
  - Pass config_entry to DataUpdateCoordinator.__init__ (recommended pattern)
- Entity Performance:
  - Set `should_poll = False` when using coordinator (inherited from CoordinatorEntity)
  - Use `__slots__` for memory efficiency in high-entity-count scenarios (optional)

# Testing
- Always add new tests, ensure they pass, and requisite code coverage is met when adding new functionality.
- Use pytest; do not make real network calls. Patch httpx and async websockets at module paths:
  - HTTP: Patch `httpx.Client` at `custom_components.fansync.client.*`
  - WebSocket: Patch `websockets.connect` at `custom_components.fansync.client.websockets.connect` with `new_callable=AsyncMock`
- Coverage focus in CI is custom_components/fansync; target ≥ 75%.
- Include tests for: push callback merge, reconnect paths, config flow (happy/error/duplicate),
  optimistic overlay expiry, and multi-device entity isolation.
- Test debug logging with caplog to verify messages format correctly.
- Ensure tests clean up background async tasks (call async_disconnect on FanSyncClient).
  - This cancels the `_recv_task` (async WebSocket receiver) and closes connections.
- Use Home Assistant's config flow test helpers (hass.config_entries.flow.async_init) instead of
  directly instantiating flow classes.
- Type annotations in tests:
  - Always annotate test function parameters: `hass: HomeAssistant`, `caplog: pytest.LogCaptureFixture`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tjbaker/homeassistant-fansync](https://github.com/tjbaker/homeassistant-fansync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
