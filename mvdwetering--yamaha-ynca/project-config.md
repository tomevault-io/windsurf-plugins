---
trigger: always_on
description: Guidelines for AI agents working on this repository.
---

# AGENTS.md

Guidelines for AI agents working on this repository.

## Project Overview

**Yamaha YNCA** is a custom Home Assistant integration for Yamaha AV receivers using the YNCA protocol over serial or network connections. It supports Yamaha RX-A, RX-V, AVENTAGE, TSR, and HTR receivers from 2010 onwards.

- **Language**: Python 3.13+
- **Framework**: Home Assistant Custom Component
- **Core dependency**: [`ynca`](https://github.com/mvdwetering/ynca) — the YNCA protocol library

---

## Repository Structure

```
custom_components/yamaha_ynca/   # Integration source code
  __init__.py                    # Entry setup, platform registration
  config_flow.py                 # Config wizard (serial / network)
  options_flow.py                # Options/settings UI
  const.py                       # Constants and LOGGER
  entity.py                      # Shared base entity mixin
  helpers.py                     # Utility functions (scale, version parsing)
  input_helpers.py               # ynca.Input → HA source mapping
  services.py                    # Custom HA services/actions
  diagnostics.py                 # Diagnostic data
  media_player.py                # Main media player platform
  button.py / switch.py / select.py / number.py / sensor.py / remote.py
  translations/en.json           # UI strings (entity names, etc.)

tests/                           # pytest test suite
  conftest.py                    # Fixtures: create_mock_zone, setup_integration
  mock_yncaconnection.py         # Fake YNCA connection for tests
  test_*.py                      # Per-platform and per-feature tests

docs/DEVELOPING.md               # Extended developer guide
pyproject.toml                   # Project config, dependencies, tool config
coverage.sh                      # Run pytest + mypy (CI equivalent)
bump_ynca_version.sh             # Update ynca package version everywhere
```

---

## Dev Environment Setup

Requires **Python 3.13**.

```bash
python3.13 -m venv venv
source ./venv/bin/activate
pip install -e .[dev]
```

---

## Running Tests & Checks

```bash
# Full check — same as CI (pytest + mypy)
./coverage.sh

# Tests only
pytest tests

# Type checking only
mypy custom_components --check-untyped-defs

# Format code
ruff format

# Lint with auto-fix
ruff check --fix
```

pytest is configured in `pyproject.toml`: async mode is `auto`, coverage is collected for `custom_components/yamaha_ynca`, and HTML + terminal reports are generated.

Always run `./coverage.sh` (or at minimum `pytest tests`) before considering a change complete.

**Test coverage must remain at 100%.** Every new code path requires a corresponding test. If coverage drops, add tests — do not add `# pragma: no cover` exclusions unless the code is genuinely unreachable (e.g., inside a `TYPE_CHECKING` block).

---

## Code Conventions

### Linting & Formatting

- **Ruff** handles both formatting and linting. The ruleset is `ALL` with specific ignores defined in `pyproject.toml`.
- Docstrings are intentionally omitted (D100–D107 are ignored).
- Use the import aliases configured in `pyproject.toml`:

  ```python
  import homeassistant.helpers.area_registry as ar
  import homeassistant.helpers.config_validation as cv
  import homeassistant.helpers.device_registry as dr
  import homeassistant.helpers.entity_registry as er
  import homeassistant.helpers.issue_registry as ir
  import voluptuous as vol
  ```

### Type Hints

- All code is type-annotated; mypy runs with `--check-untyped-defs`.
- Use `TYPE_CHECKING` guards for imports used only in annotations to avoid circular imports:

  ```python
  from typing import TYPE_CHECKING
  if TYPE_CHECKING:  # pragma: no cover
      from homeassistant.core import HomeAssistant
  ```

### Logging

- Import and use the shared logger from `const.py`:

  ```python
  from .const import LOGGER
  LOGGER.debug("message %s", value)
  ```

### Entity Description Pattern

All platforms define entities declaratively using `EntityDescription` dataclasses:

```python
@dataclass(frozen=True, kw_only=True)
class YncaSwitchEntityDescription(SwitchEntityDescription):
    on: Enum | None = None
    off: Enum | None = None
    function_names: list[str] | None = None
```

Add new entities by appending entries to the appropriate `ZONE_ENTITY_DESCRIPTIONS` list — do not write new entity subclasses unless genuinely necessary.

### Update Callbacks

Entities subscribe to YNCA updates using a consistent pattern:

```python
async def async_added_to_hass(self):
    self._subunit.register_update_callback(self.update_callback)

def update_callback(self, function: str, value: Any):
    if function in self._relevant_updates:
        self.schedule_update_ha_state()
```

### Feature Detection

Use walrus operators to conditionally create entities based on receiver capabilities:

```python
if zone := getattr(api, "zone2"):
    entities.append(YamahaYncaZone(zone))
```

### Async

All setup and I/O functions are `async`. Follow Home Assistant's async patterns throughout (`ConfigEntry`, `async_add_entities`, etc.).

### Home Assistant Best Practices

Follow the [Home Assistant developer documentation](https://developers.home-assistant.io/) for all integration patterns. Key points:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mvdwetering/yamaha_ynca](https://github.com/mvdwetering/yamaha_ynca) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
