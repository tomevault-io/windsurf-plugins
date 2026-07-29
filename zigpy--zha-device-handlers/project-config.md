---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository.
---

This file provides guidance to AI coding agents when working with code in this repository.

## Overview

ZHA-Quirks provides device-specific handlers for ZHA (Zigbee Home Automation) in Home Assistant. Quirks handle devices that don't follow standard ZCL (Zigbee Cluster Library) specifications by defining custom attributes, clusters, and entity mappings.

## Commands

```bash
# Setup development environment (includes uv sync and pre-commit install)
script/setup

# Sync dependencies after switching branches or pulling updates
uv sync

# Run all tests
pytest tests/

# Run a single test file
pytest tests/test_tuya.py

# Run a specific test function
pytest tests/test_tuya.py::test_function_name -v

# Run pre-commit checks (ruff, mypy, codespell)
pre-commit run --all-files

# Lint and format
ruff check zhaquirks/
ruff format zhaquirks/

# Type checking
mypy zhaquirks/
```

## Zigbee Concepts

**Clusters**: Functionality groupings containing attributes and commands.
- **in_clusters** (Server clusters): Control the device, send attribute reports. E.g., `OnOff` on a light bulb.
- **out_clusters** (Client clusters): Send commands to other devices. E.g., `OnOff` on a remote control.

**Endpoints**: Groupings of clusters. Multi-gang switches have separate endpoints per switch.

## Quirk Architecture

### V2 Quirks (Preferred for New Quirks)

Use `QuirkBuilder` from `zhaquirks.builder` for declarative quirk definition:

```python
from zhaquirks.builder import QuirkBuilder

(
    QuirkBuilder("Manufacturer", "Model")
    .applies_to("AltManufacturer", "Model")  # Additional models
    .replaces(CustomClusterClass)             # Replace standard cluster
    .device_automation_triggers({...})        # Button/action mappings
    .switch(attribute_name=..., fallback_name=...)  # HA entity
    .add_to_registry()
)
```

#### QuirkBuilder Methods Reference

**Device Matching:**
- `.applies_to(manufacturer, model)` - Add manufacturer/model pair to match
- `.filter(filter_function)` - Custom filter function `(device) -> bool`
- `.firmware_version_filter(min_version, max_version, allow_missing)` - Filter by firmware version

Firmware version filtering is useful when different firmware versions need different quirks:
```python
# Quirk for OLD firmware (before bug was fixed)
(
    QuirkBuilder("innr", "SP 240")
    .firmware_version_filter(max_version=0x191B3685, allow_missing=False)
    .replaces(OldFirmwareCluster)
    .add_to_registry()
)

# Quirk for NEW firmware (after bug was fixed)
(
    QuirkBuilder("innr", "SP 240")
    .firmware_version_filter(min_version=0x191B3685, allow_missing=True)
    .replaces(NewFirmwareCluster)
    .add_to_registry()
)
```
- `min_version`: Minimum firmware version (inclusive)
- `max_version`: Maximum firmware version (exclusive) - the version specified is NOT included
- `allow_missing`: If `True`, quirk applies when device has no firmware version

Note: In the example above, `0x191B3685` appears in both quirks because `max_version` is exclusive (old quirk applies to versions *before* this) while `min_version` is inclusive (new quirk applies to this version *and newer*).

**Cluster Modification:**
- `.adds(cluster, endpoint_id=1, cluster_type=ClusterType.Server, constant_attributes={})` - Add a cluster. `constant_attributes` dict forces specific attribute values (same as `_CONSTANT_ATTRIBUTES` on a custom cluster)
- `.removes(cluster_id, endpoint_id=1, cluster_type=ClusterType.Server)` - Remove a cluster
- `.replaces(replacement_cluster_class, endpoint_id=1, cluster_type=ClusterType.Server)` - Replace cluster with custom implementation
- `.replace_cluster_occurrences(cluster_class, replace_server=True, replace_client=True)` - Replace across all endpoints

`cluster_type` can be `ClusterType.Server` (in_clusters) or `ClusterType.Client` (out_clusters).

**Endpoint Modification:**
- `.adds_endpoint(endpoint_id, profile_id, device_type)`
- `.removes_endpoint(endpoint_id)`
- `.replaces_endpoint(endpoint_id, profile_id, device_type)` - Change endpoint's profile/device type

Example: Change device type so HA creates correct entity (ZHA profile used by default):
```python
.replaces_endpoint(1, device_type=zha.DeviceType.DIMMABLE_LIGHT)
```

**Entity Creation (Home Assistant):**
All entity methods require `fallback_name`. Common parameters:
- `attribute_name`: ZCL attribute to expose
- `cluster_id`: Cluster containing the attribute
- `endpoint_id`: Endpoint (default 1)
- `translation_key`: For HA translations (required if no device_class)
- `fallback_name`: English name in sentence case (always required)
- `entity_type`: `EntityType.STANDARD`, `CONFIG`, or `DIAGNOSTIC`
- `initially_disabled`: Start disabled in HA
- `device_class`: HA device class for the entity
- `reporting_config`: Configure ZCL attribute reporting
- `unique_id_suffix`: Suffix appended to the entity's unique_id. Defaults to `attribute_name` (or `command_name` for command-based entities). Required when creating multiple entities from the same attribute/command on the same endpoint, since otherwise the default suffixes collide. See **Entity unique_id format** below before changing this on existing quirks.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zigpy/zha-device-handlers](https://github.com/zigpy/zha-device-handlers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
