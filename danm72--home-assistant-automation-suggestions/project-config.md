---
trigger: always_on
description: A Home Assistant custom integration that analyzes user behavior patterns from history and suggests automations.
---

# Automation Suggestions - Home Assistant Custom Integration

A Home Assistant custom integration that analyzes user behavior patterns from history and suggests automations.

## Build/Test/Lint Commands

```bash
# Run all tests
pytest

# Run unit tests only
pytest custom_components/automation_suggestions/tests/unit/

# Run integration tests only
pytest custom_components/automation_suggestions/tests/integration/

# Run specific test file
pytest custom_components/automation_suggestions/tests/unit/test_analyzer.py

# Run with verbose output
pytest -v

# Lint with ruff
ruff check .

# Format with ruff
ruff format .

# Check formatting without changes
ruff format --check .

# Ensure you're using Python 3.13+ virtual environment
.venv/bin/pytest --ignore=tests/e2e/
```

### Test Configuration
- Requires Python 3.13+ and Home Assistant 2026.1+
- Uses `pytest-homeassistant-custom-component` plugin
- Root `conftest.py` registers the plugin: `pytest_plugins = ["pytest_homeassistant_custom_component"]`
- Tests located in: `custom_components/automation_suggestions/tests/`

### Linting Configuration (pyproject.toml)
- Target: Python 3.13
- Line length: 100
- Rules: F (Pyflakes), E (pycodestyle errors), W (warnings), I (isort), UP (pyupgrade)

## E2E Tests (Docker)

End-to-end tests using real Home Assistant Docker containers via `testcontainers`.

### Prerequisites

```bash
pip install testcontainers requests
```

### Running E2E Tests

```bash
# Run e2e tests (MUST use -c flag for isolated config)
pytest tests/e2e/ -c tests/e2e/pytest.ini

# Run all tests except e2e
pytest --ignore=tests/e2e/

# Run e2e in live mode (against real HA instance)
pytest tests/e2e/ -c tests/e2e/pytest.ini --live
```

### Socket Blocking Fix

`pytest-homeassistant-custom-component` blocks sockets via `pytest-socket`, breaking Docker/testcontainers.

**Key discovery**: The plugin registers as `homeassistant` (not `homeassistant-custom-component`):
```
# From entry_points.txt:
[pytest11]
homeassistant = pytest_homeassistant_custom_component.plugins
```

**Solution**: `tests/e2e/pytest.ini` uses `-p no:homeassistant` to disable the plugin.

See: `docs/solutions/testing/pytest-homeassistant-socket-blocking.md`

### How It Works

The `HomeAssistantTestContainer` class in `tests/e2e/conftest.py`:
1. Creates a temp config directory (copies from `initial_test_state/` if exists)
2. Starts `homeassistant/home-assistant:stable` Docker container
3. Binds port 8123 and mounts config volume
4. Waits for HA to be ready (up to 120s timeout)
5. Provides `ha_url` fixture for tests
6. Cleans up container and temp files on teardown

### Test Files

```
tests/e2e/
├── __init__.py
├── conftest.py           # HomeAssistantTestContainer, fixtures
├── README.md             # Setup instructions
└── test_recorder_api.py  # API compatibility tests
```

### What E2E Tests Catch

- API compatibility issues (e.g., `session_scope` deprecation)
- Real Home Assistant behavior vs mocked behavior
- Integration with actual recorder database

### Setting Up Authenticated Tests

For tests requiring auth:
1. Start a fresh HA container manually
2. Complete onboarding (user: `test` / password: `test`)
3. Generate a long-lived access token
4. Copy `.storage/` directory to `tests/e2e/initial_test_state/`
5. Update `TEST_TOKEN` in `conftest.py`

## Visual E2E Tests (CRITICAL QA)

**Visual E2E testing with Chrome MCP is a CRITICAL part of QA.** Unit and integration tests verify logic, but visual tests verify the actual user experience in a real browser against a real Home Assistant instance.

### Why Visual E2E is Critical

- Catches UI rendering issues that mocked tests miss
- Verifies Lovelace card JavaScript works in real HA environment
- Tests WebSocket subscriptions with real coordinator updates
- Validates CSS styling, tab switching, button interactions
- Screenshots provide evidence that features actually work

### Running Visual E2E Tests

```bash
# 1. Start the Docker container with custom component
python tests/e2e/start_visual_test.py

# 2. Use Chrome MCP tools to run visual tests:
#    - Navigate to the HA URL printed by the script
#    - Complete onboarding if fresh container
#    - Configure the integration
#    - Add the Lovelace card
#    - Take screenshots and verify UI elements
```

### Visual Test Specifications

Visual test procedures are documented in:
- `tests/e2e/visual_test_stale_automations.md` - Stale automation card tests

### Chrome MCP Tools for Visual Testing

Key tools for visual E2E:
- `mcp__claude-in-chrome__tabs_context_mcp` - Get browser tab context
- `mcp__claude-in-chrome__navigate` - Navigate to URLs
- `mcp__claude-in-chrome__computer` (action: screenshot) - Take screenshots
- `mcp__claude-in-chrome__read_page` - Read DOM/accessibility tree
- `mcp__claude-in-chrome__computer` (action: left_click) - Click elements

### QA Checklist

Before merging any Lovelace card changes:
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] E2E API tests pass
- [ ] **Visual E2E tests pass with screenshots** ← CRITICAL

## Architecture

### Key Components

1. **coordinator.py** - `AutomationSuggestionsCoordinator`
   - Extends `DataUpdateCoordinator[list[Suggestion]]`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Danm72/home-assistant-automation-suggestions](https://github.com/Danm72/home-assistant-automation-suggestions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
