---
trigger: always_on
description: This document provides comprehensive context for AI assistants working on the DucoBox Integration for Home Assistant.
---

# Home Assistant DucoBox Integration - AI Assistant Context

This document provides comprehensive context for AI assistants working on the DucoBox Integration for Home Assistant.

## Integration Overview

**Purpose:** Control and monitor DucoBox ventilation systems with Communication Print (0000-4251) hardware via local REST API.

**Why This Fork Exists:**
- Original upstream repository (degeens/ha-ducobox) supports Connectivity Board 2.0
- This fork adds full Communication Print support
- Upstream maintainer couldn't test Communication Print changes (doesn't have hardware)
- PR was rejected, leading to independent fork
- **Version 1.0.0** is the first independent release supporting Communication Print only

**Architecture:** Clean abstract base class pattern allows hardware-specific API implementations without affecting entity code.

## Hardware Supported

**Communication Print (0000-4251):**
- Older DucoBox connectivity hardware
- REST API over HTTP (no authentication)
- All features fully implemented and tested
- Reference device: DucoBox Energy with Communication Print

**NOT Supported:**
- Connectivity Board 2.0 (newer hardware)
- Connectivity Board 1.0 (legacy)

Users with Connectivity Board should use upstream: https://github.com/degeens/ha-ducobox

## Communication Print REST API Endpoints

### Device Detection & Info

**`GET /nodeinfoget?node=1`**
- Primary detection endpoint (must respond for integration to work)
- Returns device type, serial number, software version, location
- Required fields in response: `devtype`, `state`, `serialnb`

### Box State Data

**`GET /nodeinfoget?node=1`**
- Box ventilation state and sensor data
- Returns: `state`, `mode`, `trgt` (target flow), `rh` (humidity), `cntdwn` (countdown), `endtime`
- State codes mapped to friendly names: AUTO→Auto, MAN1→Manual 1, MAN2→Manual 2, MAN3→Manual 3, CNT1/2/3→Manual 1/2/3 Forced, EMPT→Away

**`GET /boxinfoget`**
- Energy information from main box
- Returns: `EnergyInfo` (temperatures, bypass, filter), `EnergyFan` (fan speeds/PWM)
- Temperatures returned in deciselsius (divide by 10)

### Ventilation Control

**`GET /nodesetoperstate?node=1&value=<STATE>`**
- Set ventilation state/preset mode
- State values: AUTO, MAN1, MAN2, MAN3, CNT1, CNT2, CNT3, EMPT
- Returns HTTP 200 on success (no JSON body)

**`GET /nodesetoverrule?node=<NODE_ID>&value=<0-100|255>`**
- Manual flow override (percentage slider)
- 0-100: Set specific flow percentage (triggers EXTN/Override mode)
- 255: Clear override, return to auto/preset mode
- Works for both main box (node 1) and room sensors

### Node Discovery

**`GET /nodeinfoget?node=<NODE_ID>`**
- Query individual node by ID
- Valid nodes have both `location` and `devtype` fields
- Scan ranges: 2-10 (room sensors), 50-100 (box sensors like UCRH)
- Integration uses 2-second timeout for fast scanning

**Typical Node Types:**
- UCCO2: CO2 sensor with temperature
- UCRH: Humidity sensor
- UCTEMP: Temperature-only sensor

### Box Configuration

**`GET /boxconfigget`**
- Box-level configuration parameters
- Returns `Energy` section with parameters like:
  - BypassMode, BypassAdaptive, ComfortTemperature
  - CalibPinMax, CalibPoutMax, CalibQout (airflow calibration)
  - ProgramModeZone1, ProgramModeZone2
  - FilterReset

**`GET /boxconfigset?mod=Energy&para=<PARAMETER>&value=<VALUE>`**
- Set box-level configuration
- All box params go through `Energy` module
- Returns HTTP 200 on success

**Special: ComfortTemperature**
- API stores value with 8-unit offset
- Formula: `api_value = (celsius * 10) + 8`
- Example: 20°C → API value 208

### Node Configuration

**`GET /nodeconfigget?node=<NODE_ID>`**
- Node-specific configuration for room sensors
- Returns parameters like:
  - CO2Setpoint, RHSetpoint (demand thresholds)
  - Manual1, Manual2, Manual3 (manual mode flow rates)
  - ManualTimeout (auto-return timer)
  - TempDependent, RHDelta (boolean switches)
  - SensorVisuLevel (display sensitivity)

**`GET /nodeconfigset?node=<NODE_ID>&para=<PARAMETER>&value=<VALUE>`**
- Set node-specific configuration
- Direct parameter names (no module prefix)

**Parameter Structure:**
All config parameters return object with:
- `Val`: Current value
- `Min`: Minimum allowed
- `Max`: Maximum allowed
- `Inc`: Increment step

**Main DucoBox (node 1) Special Handling:**
- Fetches BOTH `/nodeconfigget?node=1` AND `/boxconfigget`
- Merges node-level and box-level config into single DucoBoxNodeConfig object
- Box-level params (bypass, comfort temp, filter, calibration) only available for node 1

## Differential Polling Strategy

**Problem:** Simultaneous polling of box + energy + all nodes caused timeout cascades on slow devices.

**Solution:** Staggered polling with caching:

| Data Type | Frequency | Fetch Parameter | Cache When Skipped |
|-----------|-----------|-----------------|---------------------|
| Box State | Every 15s | Always fetched | N/A (always fresh) |
| Node Data | Every 9s | `fetch_nodes` conditional | Yes (uses `_cached_nodes`) |
| Energy Info | Every 60s | `fetch_energy` conditional | Yes (uses `_cached_energy`) |

**Coordinator Logic (coordinator.py):**
```python
now = time.time()

# Node data: fetch every 9 seconds

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danielpetrovic/ha-ducobox](https://github.com/danielpetrovic/ha-ducobox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
