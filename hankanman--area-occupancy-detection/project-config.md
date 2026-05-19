---
trigger: always_on
description: All tests are written using pytest.
---


All tests are written using pytest.

All tests are written in the `tests` directory.

The `conftest.py` file is used to configure the tests and contains the fixtures for the tests.

All fixtures should be defined in the conftest.py file.

Each module has its own test file. e.g. `tests/test_coordinator.py`.

Testing is done by running the command `pytest --cov=custom_components/area_occupancy --cov-report=xml --cov-report=term-missing`.

Test coverage is expected to be over 85%.

# Testing Guide for Area Occupancy Detection

This guide explains how to write and maintain tests for the Area Occupancy Detection component, with a focus on the multi-area architecture.

## Table of Contents

- [Architecture Overview](#architecture-overview)
- [Using pytest-homeassistant-custom-component](#using-pytest-homeassistant-custom-component)
- [Choosing the Right Fixtures](#choosing-the-right-fixtures)
- [Common Patterns](#common-patterns)
- [Area-Based Access](#area-based-access)
- [Database Connection Management](#database-connection-management)
- [Migration Guide](#migration-guide)
- [Examples](#examples)

## Architecture Overview

The component uses a **multi-area architecture** where:

- `AreaOccupancyCoordinator` manages multiple `Area` objects
- Each `Area` has its own `config`, `entities`, `prior`, and `purpose`
- Properties like `probability()`, `threshold()`, `device_info()` now take `area_name` parameter
- Access patterns: `coordinator.get_area_or_default(area_name).entities` instead of `coordinator.entities`

## Using pytest-homeassistant-custom-component

This project uses the [`pytest-homeassistant-custom-component`](https://pypi.org/project/pytest-homeassistant-custom-component/) package to provide proper Home Assistant test infrastructure. This package automatically provides fixtures for testing custom components.

### Key Fixtures from the Package

**`hass`** - Real Home Assistant instance for testing:

```python
def test_with_hass(hass: HomeAssistant):
    """Test using real Home Assistant instance."""
    # hass is a real HomeAssistant instance, not a mock
    assert hass is not None
    assert hass.states is not None
    assert hass.config_entries is not None
```

**`device_registry`** - Device registry fixture:

```python
async def test_device_registry(hass: HomeAssistant, device_registry):
    """Test using device registry."""
    # Create a device in the registry
    device_entry = device_registry.async_get_or_create(
        config_entry_id="test_entry_id",
        identifiers={("domain", "identifier")},
        name="Test Device",
    )
    assert device_entry is not None
```

**`entity_registry`** - Entity registry fixture:

```python
async def test_entity_registry(hass: HomeAssistant, entity_registry):
    """Test using entity registry."""
    # Access entity registry
    entities = entity_registry.entities
    assert entities is not None
```

**`enable_custom_integrations`** - Automatically enables custom integrations (autouse fixture):

This fixture is automatically applied to all tests, so you don't need to explicitly request it. It ensures that custom components can be loaded during tests.

### Import Requirements

When using the `hass` fixture, import `HomeAssistant` from `homeassistant.core`:

```python
from homeassistant.core import HomeAssistant

def test_something(hass: HomeAssistant):
    """Test using hass fixture."""
    # Use hass here
```

### Benefits of Using the Package

1. **Real Home Assistant Infrastructure** - Tests run against real Home Assistant components, not mocks
2. **Automatic Setup** - The package handles all the complex setup required for testing custom components
3. **Standard Fixtures** - Uses the same fixtures that Home Assistant core uses for testing
4. **Better Integration Testing** - Tests catch real-world issues that mocks might miss
5. **Device and Entity Registry Support** - Provides real registry fixtures for testing device/entity interactions

### Migration from Custom mock_hass

**We no longer use a custom `mock_hass` fixture.** All tests should use the `hass` fixture from `pytest-homeassistant-custom-component`:

```python
# OLD - Don't use this anymore
def test_something(mock_hass: Mock):
    mock_hass.states = Mock()
    # ...

# NEW - Use this instead
def test_something(hass: HomeAssistant):
    # hass already has states, config_entries, etc.
    # No need to mock them
```

## Choosing the Right Fixtures

### Real Coordinators vs Mock Coordinators

**We now prefer using real coordinators and databases in tests.** This provides better integration testing and catches real-world issues.

**Use `coordinator` when:**

- Integration-style tests that need real coordinator behavior
- Testing area loading and initialization
- You need areas to be automatically loaded from config
- Testing coordinator methods that interact with areas
- **This is the recommended default for most tests**

**Use `test_db` when:**

- Testing database operations directly
- Testing data persistence and loading
- Testing migration logic
- Testing database schema and models
- **This is the recommended primary fixture for database testing**

**Use `coordinator_with_db` when:**

- Testing database operations that need coordinator access
- Testing integration between coordinator and database

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Hankanman/Area-Occupancy-Detection](https://github.com/Hankanman/Area-Occupancy-Detection) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
