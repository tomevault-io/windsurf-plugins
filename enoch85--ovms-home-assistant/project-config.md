---
trigger: always_on
description: Handles various topic structures including custom formats.
---

# OVMS Home Assistant Integration - AI Coding Agent Instructions

Home Assistant custom integration providing MQTT-based vehicle monitoring via OVMS (Open Vehicle Monitoring System). **Production code affecting real vehicle data and commands.**

## Core Principles

1. **Read entire files before editing** - Never edit based on grep/partial reads
2. **Configuration-driven, never hardcode** - All values from `const.py`, never hardcode numbers, strings, timeouts, or dynamic values
3. **No regressions** - Preserve existing behavior unless explicitly improving with clear justification
4. **No breaking changes without reason** - Respect existing structure, only refactor when materially beneficial
5. **Test after every change** - Run `python3 scripts/tests/test_code_quality.py` after every code iteration to verify quality
6. **No verbose summaries** - User sees changes in editor, only respond if asked
7. **Keep it simple** - Clean code over complexity, cleanup as you go
8. **Ask before acting** - When uncertain, clarify first
9. **Format with black** - Run `black` on all Python code before committing (line length 88)

## Project Overview

This is a **Home Assistant custom integration** that connects electric vehicles via OVMS (Open Vehicle Monitoring System) modules through MQTT. The integration automatically discovers vehicle metrics, creates appropriate entities (sensors, binary sensors, device trackers, switches), and provides bidirectional command capability.

**Key Architectural Principle**: Discovery-based entity creation with pattern matching - the integration dynamically creates entities from MQTT topics rather than requiring manual configuration.

## OVMS Firmware Changelog

**IMPORTANT**: When developing new features or improving existing functionality, always check the OVMS firmware changelog for new capabilities that could simplify implementation or enable new features:

**Firmware Changelog URL**: https://raw.githubusercontent.com/openvehicles/Open-Vehicle-Monitoring-System-3/refs/heads/master/vehicle/OVMS.V3/changes.txt

**When to check the changelog:**
1. Before implementing new discovery mechanisms
2. When adding command/response functionality  
3. When troubleshooting MQTT communication issues
4. Before major refactoring of MQTT-related code
5. When users report features available in newer firmware

This ensures the integration takes advantage of the latest OVMS capabilities rather than using legacy workarounds.

## Core Architecture

### Component Boundaries

```
MQTT Broker (external)
    ↓
MQTTConnectionManager (mqtt/connection.py)
    ↓
OVMSMQTTClient (mqtt/__init__.py) - orchestrates all components
    ├── TopicParser (mqtt/topic_parser.py) - identifies entity types from topics
    ├── StateParser (mqtt/state_parser.py) - converts MQTT payloads to HA states
    ├── EntityFactory (mqtt/entity_factory.py) - creates entities
    ├── UpdateDispatcher (mqtt/update_dispatcher.py) - pushes state updates
    ├── EntityRegistry (mqtt/entity_registry.py) - tracks created entities
    └── CommandHandler (mqtt/command_handler.py) - sends commands to vehicles
```

### Critical Data Flows

1. **Topic Discovery Flow**: During setup, the integration subscribes to `{prefix}/{mqtt_username}/{vehicle_id}/#` wildcard, collects all topics for ~5 seconds, then parses them to determine entity types.

2. **Entity Creation Pattern**: Topic → TopicParser (matches patterns in `metrics/patterns.py`) → EntityFactory → Platform-specific entity class → Home Assistant entity registry.

3. **State Updates**: MQTT message → StateParser (handles type conversion, array processing) → UpdateDispatcher → Entity state update via signal dispatch.

4. **Command Flow**: Service call → CommandHandler.async_send_command() → MQTT publish to `{prefix}/{username}/{vehicle_id}/client/rr/command/{command_id}` → Response on `{prefix}/{username}/{vehicle_id}/client/rr/response/{command_id}`.

### Configuration Management

- **Config Version**: Track in `const.py::CONFIG_VERSION` (currently 6)
- **Migration Pattern**: `__init__.py::async_migrate_entry()` handles version upgrades
- **Config Merge**: Always use `get_merged_config(entry)` - options override data
- **Stable Client IDs**: Generated as `ha_ovms_{sha256_hash[:12]}` from host+username+vehicle_id to prevent MQTT connection issues

## Critical Rules

### Always Read Full Files

Before editing, use `read_file` for entire file. Understand context, identify all change locations. Never edit based on grep results or partial file reads.

### Use Configuration Constants

**ALL numeric values, thresholds, and tuning parameters MUST be constants from `const.py`.**

```python
# ✅ Do this
from .const import DEFAULT_SCAN_INTERVAL, MAX_STATE_LENGTH, DEFAULT_COMMAND_TIMEOUT
timeout = DEFAULT_SCAN_INTERVAL
max_length = MAX_STATE_LENGTH
command_timeout = DEFAULT_COMMAND_TIMEOUT

# ❌ Never this
timeout = 60  # NEVER hardcode
max_len = 255  # NEVER hardcode
limit = 5  # NEVER hardcode
```

**Constant Reuse Across Files:**
- Production code imports from `const.py`
- Test code imports SAME constants
- NO duplicate definitions - single source of truth
- If constant exists, use it everywhere applicable

**When to Add New Constants:**
1. Any numeric threshold or tuning parameter

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [enoch85/ovms-home-assistant](https://github.com/enoch85/ovms-home-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
