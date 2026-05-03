---
trigger: always_on
description: Rules for writing tests for the BMW Wallbox integration
---


# Rules: Writing Tests

## Documentation References

**MANDATORY - Read these files BEFORE writing tests:**

1. `custom_components/bmw_wallbox/docs/TESTING.md` - Complete testing guide, patterns, best practices
2. `tests/conftest.py` - Available fixtures (mock_coordinator, mock_config_entry, etc.)
3. `custom_components/bmw_wallbox/docs/DATA_SCHEMAS.md` - Expected data structures for mocking
4. `custom_components/bmw_wallbox/docs/ENTITIES.md` - Entity properties to test

**For testing specific components:**
- Sensors: `docs/ENTITIES.md` section on sensor properties
- Controls: `docs/ENTITIES.md` section on buttons/switches/numbers
- Config flow: `docs/DATA_SCHEMAS.md` section on configuration schema

## Running Tests

```bash
# All tests
pytest tests/ -v

# Specific file
pytest tests/test_sensor.py -v

# Specific test
pytest tests/test_sensor.py::test_power_sensor -v

# With coverage
pytest tests/ --cov=custom_components.bmw_wallbox
```

## Available Fixtures

### mock_coordinator

Mock `BMWWallboxCoordinator` with test data.

```python
async def test_something(mock_coordinator):
    # Has all coordinator.data fields populated
    assert mock_coordinator.data["power"] == 7000.0
    assert mock_coordinator.current_transaction_id == "test-transaction-123"
```

### mock_config_entry

Mock `ConfigEntry` with test configuration.

```python
async def test_something(mock_config_entry):
    assert mock_config_entry.entry_id == "test_entry_id"
    assert mock_config_entry.data["port"] == 9000
```

### mock_wallbox_charge_point

Mock `WallboxChargePoint` for OCPP testing.

```python
async def test_something(mock_wallbox_charge_point):
    mock_wallbox_charge_point.call = AsyncMock(return_value=response)
```

---

## Testing Sensors

```python
from custom_components.bmw_wallbox.sensor import BMWWallboxPowerSensor

async def test_power_sensor(hass, mock_coordinator, mock_config_entry):
    """Test power sensor returns correct value."""
    sensor = BMWWallboxPowerSensor(mock_coordinator, mock_config_entry)

    # Test value
    assert sensor.native_value == 7000.0

    # Test unit
    assert sensor.native_unit_of_measurement == "W"

    # Test device class
    assert sensor.device_class == "power"
```

### Test with Data Changes

```python
async def test_sensor_updates(hass, mock_coordinator, mock_config_entry):
    """Test sensor updates when data changes."""
    sensor = BMWWallboxPowerSensor(mock_coordinator, mock_config_entry)

    # Initial value
    assert sensor.native_value == 7000.0

    # Update coordinator data
    mock_coordinator.data["power"] = 5000.0

    # Value should reflect change
    assert sensor.native_value == 5000.0
```

### Test with Extra Attributes

```python
async def test_sensor_attributes(hass, mock_coordinator, mock_config_entry):
    """Test sensor extra attributes."""
    sensor = BMWWallboxStateSensor(mock_coordinator, mock_config_entry)

    attrs = sensor.extra_state_attributes
    assert attrs["evse_id"] == 1
    assert attrs["connector_id"] == 1
```

---

## Testing Buttons

```python
from custom_components.bmw_wallbox.button import BMWWallboxStartButton

async def test_start_button(hass, mock_coordinator, mock_config_entry):
    """Test start charging button."""
    button = BMWWallboxStartButton(mock_coordinator, mock_config_entry, hass)

    # Test properties
    assert button.name == "Start Charging"
    assert button._base_icon == "mdi:play"

    # Test press
    await button.async_press()

    # Verify coordinator method was called
    mock_coordinator.async_start_charging.assert_called_once()
```

---

## Testing Numbers

```python
from custom_components.bmw_wallbox.number import BMWWallboxCurrentLimitNumber

async def test_current_limit(hass, mock_coordinator, mock_config_entry):
    """Test current limit number entity."""
    number = BMWWallboxCurrentLimitNumber(mock_coordinator, mock_config_entry)

    # Test properties
    assert number.native_min_value == 0
    assert number.native_max_value == 32

    # Test set value
    await number.async_set_native_value(16.0)

    # Verify coordinator method was called
    mock_coordinator.async_set_current_limit.assert_called_once_with(16.0)
```

### Test Availability

```python
async def test_requires_transaction(hass, mock_coordinator, mock_config_entry):
    """Test entity is unavailable without transaction."""
    number = BMWWallboxCurrentLimitNumber(mock_coordinator, mock_config_entry)

    # With transaction - available
    mock_coordinator.current_transaction_id = "test-123"
    assert number.available is True

    # Without transaction - unavailable
    mock_coordinator.current_transaction_id = None
    assert number.available is False
```

---

## Testing Config Flow

```python
from homeassistant import config_entries
from custom_components.bmw_wallbox.const import DOMAIN

async def test_config_flow(hass):
    """Test successful config flow."""
    result = await hass.config_entries.flow.async_init(
        DOMAIN, context={"source": config_entries.SOURCE_USER}
    )

    assert result["type"] == "form"
    assert result["step_id"] == "user"

    # Mock file existence
    with patch("os.path.isfile", return_value=True):
        result2 = await hass.config_entries.flow.async_configure(
            result["flow_id"],

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JoaoPedroBelo/bmw-wallbox-ha](https://github.com/JoaoPedroBelo/bmw-wallbox-ha) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
