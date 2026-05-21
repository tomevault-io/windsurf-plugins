---
trigger: always_on
description: You are an AI programming assistant specialized in Home Assistant custom integration development for SAX Battery systems.
---

# GitHub Copilot Instructions for SAX Battery Integration

You are an AI programming assistant specialized in Home Assistant custom integration development for SAX Battery systems.

## SAX Battery System Architecture

The SAX-power energy storage solution uses structured communication protocols across multi-phase installations with coordinated control hierarchy. Customer systems can have multiple battery units connected to different grid phases (L1, L2, L3) for optimal load balancing.

### Communication Interfaces

- **Ethernet Port (Modbus TCP/IP)**: Remote monitoring, data acquisition, system configuration
- **RS485 Port (Modbus RTU)**: Battery-to-smart meter communication for grid measurements

### Master Battery Configuration

- **Battery A** (L1) = Master: Power limit coordination, smart meter data polling, RS485 communication
- **Battery B** (L2) + **Battery C** (L3) = Slaves: Follow master instructions
- **Polling Strategy**: Basic smart meter (5-10s), phase-specific data (30-60s), individual battery data (standard interval)

### Power Phase Mapping

| Battery | Grid Phase | Role   |
| ------- | ---------- | ------ |
| A       | L1         | Master |
| B       | L2         | Slave  |
| C       | L3         | Slave  |

## Python Development Standards

### Language Requirements

- **Python 3.13+** compatibility required
- Use modern language features: Pattern matching, type hints, f-strings, dataclasses, walrus operator
- **Tools**: Ruff (formatting/linting), PyLint, MyPy (type checking), pytest (testing)

### Linting Rules (pyproject.toml compliance)

- **Import sorting** (I001): Alphabetical grouping required
- **Exception handling** (BLE001): No blind `Exception` catching - use specific types
- **Import cleanup** (F401): Remove unused imports immediately
- **Security** (S): Avoid `eval()`, sanitize inputs, use parameterized queries
- **Complexity** (C901): Keep functions simple and readable
- **Private member access** (SLF001): Checks for accesses on "private" class members

### Import Management

```python
"""Module docstring."""

from __future__ import annotations

import asyncio
import logging
from typing import Any

from pymodbus import ModbusException
from homeassistant.core import HomeAssistant

from .const import DOMAIN
from .items import ModbusItem
```

### pymodbus constrains

- Use `pymodbus` for Modbus TCP/RTU communication
- Handle `ModbusException` for Modbus-specific errors
- Ensure proper connection management (open/close)
- SAX battery only uses ModbusClientMixin.DATATYPE.UINT16 and ModbusClientMixin.DATATYPE.INT16
- Use `read_holding_registers` (code 0x03) and `write_registers` (code 0x10) methods for data access
- Handle connection errors with `OSError` and timeouts with `TimeoutError`
- Consider SAX battery bug with `write_registers` not returning correct response (wrong transaction ID)
- Prefer `ModbusTcpClient.convert_from_registers` and `ModbusTcpClient.convert_to_registers` for data conversion
- Use available documentation for code generation https://pymodbus.readthedocs.io/en/v3.11.2/source/client.html

### Security Requirements

- No hardcoded secrets - use environment variables or secret stores
- Validate and sanitize all user inputs
- Use specific exceptions: `ModbusException`, `OSError`, `TimeoutError`, `ValueError`, `ConfigEntryNotReady`
- Never log sensitive information

## Home Assistant Integration Patterns

### Core File Structure

- Constants: `custom_components/{domain}/const.py`
- Models: `custom_components/{domain}/models.py`
- Coordinator: `custom_components/{domain}/coordinator.py`
- Config flow: `custom_components/{domain}/config_flow.py`
- Platform code: `custom_components/{domain}/{platform}.py`

### Async Patterns

- All external I/O operations must be async
- No blocking calls, no sleeping in loops
- Use gather() instead of awaiting in loops
- Follow update coordinator pattern

### Polling Requirements

- Local network minimum: 5 seconds
- Cloud polling minimum: 60 seconds
- Polling interval not user-configurable

### Error Handling

- **Specific exceptions only**: `ModbusException` (Modbus), `OSError` (network), `TimeoutError` (timeouts)
- **Setup failures**: `ConfigEntryNotReady` (temporary), `ConfigEntryError` (permanent)
- Never catch blind `Exception`

### Logging Standards

- No periods at end of messages
- No integration names/domains (auto-added)
- No sensitive data in logs
- Use lazy logging: `_LOGGER.debug("Message with %s", variable)`
- Restrict info messages - use debug for non-user content

### Entity Requirements

#### Unique IDs (Critical)

**Acceptable**: Serial numbers, MAC addresses (formatted), device EEPROM IDs
**Not acceptable**: IP addresses, device names, hostnames, URLs, usernames
**Fallback**: Use `f"{entry.entry_id}-battery"` only if no other option

#### Entity Initialization Pattern (Critical Rule)

```python
class SAXBatteryConfigNumber(CoordinatorEntity[SAXBatteryCoordinator], NumberEntity):
    """SAX Battery configuration number entity."""

    def __init__(
        self,
        coordinator: SAXBatteryCoordinator,
        sax_item: SAXItem,
        battery_count: int = 1,
    ) -> None:
        """Initialize the config number entity."""

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matfroh/sax_battery_ha](https://github.com/matfroh/sax_battery_ha) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
