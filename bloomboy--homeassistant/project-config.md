---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Home Assistant configuration repository running version 2025.10.1. The setup includes:

- Main Home Assistant configuration with YAML-based config
- Custom ZHA (Zigbee Home Automation) device quirks
- ESPHome device configurations for M5Stack Atom Echo devices
- Custom components (HACS-managed)
- Automation blueprints for various lighting and sensor scenarios

## Architecture

### Core Configuration Structure
- `configuration.yaml` - Main HA configuration using include directives to organize all components
  - **Automations**: `automation: !include automations.yaml` - Single file containing ALL automations as a YAML list
  - MQTT lights/sensors loaded from `mqtt/` directory
  - ZHA setup with custom quirks enabled
  - Notification groups from `notify/groups.yaml`
  - Lovelace dashboards configured in YAML mode from `dashboards/` directory
  - Input helpers organized by type in `input_helpers/` subdirectories
- `automations.yaml` - **CRITICAL**: Single file containing all automations as a flat YAML list
  - All automations are stored directly in this file, NOT in subdirectories
  - Each automation is a list item starting with `- id: 'unique_id'`
  - Common automation types: Blueprint-based (Shelly i4 buttons, motion sensors), custom event-driven, time-based triggers
  - Heavy use of `use_blueprint` for consistency (Danieldz/shelly-i4-4-buttons-actions.yaml, Blackshome/sensor-light.yaml)
- `scripts/` - Scripts organized by category subdirectories (lighting, family, dashboard, device config, testing)
  - Loaded via `!include_dir_merge_named scripts/` in configuration.yaml
- `scenes/` - Scene definitions organized by room subdirectories (bathroom, living_room, dashboard)
  - Loaded via `!include_dir_merge_list scenes/` in configuration.yaml

### Custom Components (HACS-managed)
- `custom_components/family_safety/` - Microsoft Family Safety integration
- `custom_components/hacs/` - Home Assistant Community Store integration
- `custom_components/bermuda/` - Bluetooth proximity tracking
- `custom_components/circadian_lighting/` - Adaptive lighting based on time of day
- `custom_components/dreame_mower/` - Dreame robot mower integration
- `custom_components/ha_strava/` - Strava fitness tracking integration
- `custom_components/zaptec/` - Zaptec EV charger integration
- Custom components follow standard HA integration patterns with manifests, coordinators, and entity classes

### ZHA Quirks
- `custom_zha_quirks/hue_wall_switch_rdm004.py` - Custom device quirk for Philips/Signify RDM004 wall switches
- Implements custom clusters for handling button press events and device-specific behaviors

### ESPHome Configurations
- `esphome/` - Device configs for M5Stack Atom Echo voice assistants
- Uses packages from GitHub firmware repository
- Each device has unique encryption keys and WiFi credentials from secrets

### Blueprints
- `blueprints/automation/` - Community automation blueprints for motion lighting, wall switches, and notifications
- `blueprints/script/` - Script blueprints for device configuration and notifications
- `blueprints/template/` - Template blueprints for sensor inversions

## Development Commands

This is a configuration-based Home Assistant setup with no build/test/lint commands. Development involves:

- Edit YAML configuration files directly
- Use Home Assistant's built-in configuration validation
- Test automations and scripts through HA UI
- ESPHome configurations can be validated with `esphome config <file.yaml>`

## Git Workflow and Deployment

**CRITICAL REMINDER**: Claude Code should NEVER commit changes or update Home Assistant directly. The user handles all Git operations and Home Assistant updates.

⚠️ **IMPORTANT**: Claude CANNOT test changes against the live Home Assistant system until AFTER the user has committed and deployed the changes. All REST API calls and MCP operations only work AFTER user deployment.

**Claude's Role:**
- Make configuration file changes locally in the repository
- Validate configuration syntax when possible (YAML structure only)
- Provide recommendations and implementation
- **NEVER assume entity_IDs exist until user confirms after deployment**

**User's Role:**
- Git commit and push changes to repository
- Update Home Assistant from Git repository
- Test and verify changes in live Home Assistant environment
- Report back any validation errors to Claude for fixes

**Deployment Process:**
1. Claude makes local file changes
2. User commits and pushes to Git
3. User updates Home Assistant from Git
4. User tests configuration validation
5. User reports any errors back to Claude for fixes

This separation ensures proper change control and prevents Claude from making assumptions about live system state before deployment.

## MCP Integration

This repository has Model Context Protocol (MCP) integration set up for direct Home Assistant access:

- **Home Assistant MCP Server**: Installed and running at `http://192.168.1.191:8123/mcp_server/sse`
- **Access Token**: Long-lived token stored in `.env` file (HA_TOKEN)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BloomBoy/homeassistant](https://github.com/BloomBoy/homeassistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
