---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Home Assistant configuration repository for a smart home setup running on an Intel NUC. It uses YAML-based configuration with automations, scripts, sensors, and custom components.

## Commands

**Linting:**
```bash
yamllint -c .github/yamllint-config.yml .
```

CI runs yamllint automatically on push via GitHub Actions.

**Update README (auto-generates automation documentation):**
```bash
python utils/update-readme.py
```

**SSH Access to Home Assistant:**
```bash
ssh ha "ha core info"              # HA version and status
ssh ha "ha core stats"             # CPU, memory usage
ssh ha "ha supervisor info"        # Supervisor info
ssh ha "ha addons info"            # List add-ons
```

**Check Repairs/Issues (requires Spook integration):**
```bash
ssh ha "python3 /config/utils/check-repairs.py"
```

This shows the same repairs visible in Settings → System → Repairs (uses WebSocket API via `repairs/list_issues`).

⚠️ **IMPORTANT**: Only use commands that **inspect** state. Before running any command that **changes** state (restart, reload, service calls, etc.), explicitly ask for permission first.

## Git Commits

Write descriptive commit messages that explain **what** changed and **why**:

**Good:**
- `Turn off front house lights at midnight instead of 1am`
- `Add second motion sensor to utility room for better coverage`
- `Split baby_room_lights into individual light entities`

**Bad:**
- `Update light.yaml`
- `Fix lights`
- `Update binary sensors`

Group related changes into logical commits. Each commit should be a single coherent change.

## Architecture

### Configuration Structure

```
configuration.yaml          # Main entry point - includes all other configs
├── automations/            # Automation YAML files (merged via !include_dir_merge_list)
├── includes/               # Entity configs (sensors, switches, input_*, etc.)
├── scripts.yaml            # All scripts in one file
├── scenes.yaml             # Managed via UI
├── themes/                 # Lovelace themes
├── pyscript/               # Python scripts for HA (battery monitor, wake up light)
├── appdaemon/apps/         # AppDaemon applications
└── utils/                  # Helper scripts (README updater, DB analyzer, etc.)
```

### Key Patterns

**Automations** are split by domain into `automations/` directory:
- `climate.yaml` - thermostat scheduling and temperature control
- `light.yaml` - automated lighting
- `leaving.yaml` / `arriving.yaml` - presence-based actions
- `vacation_mode.yaml` - away mode behaviors
- `sleep_mode.yaml` - night mode settings

**Includes** contain entity definitions in `includes/`:
- `input_booleans.yaml` - toggle states (vacation_mode, guest_mode, etc.)
- `input_numbers.yaml` - numeric settings (temperature_high/low, etc.)
- `sensors.yaml` - template and platform sensors
- `binary_sensors.yaml` - presence detection and state sensors

**Scripts** in `scripts.yaml` are reusable actions called by automations:
- `set_high_temperature` / `set_low_temperature`
- `leaving` / `arriving`
- `cozy_lights_living_room` / `white_lights_living_room`

### Automation YAML Style Conventions

#### File Structure
- Each file starts with `---` followed by ASCII art header (domain name)
- Header ends with `#- from github.com/basnijholt/home-assistant-config`
- Sub-sections within files use ASCII art sub-headers (e.g., "Living room", "Bedroom")

#### Alias Format
Always `"Category: description"` with double quotes:
```yaml
- alias: "Light: turn on living room"
- alias: "Security: motion detected but we are not home"
- alias: "Climate: turn off the heating at 22:00"
```

#### Single vs Multiple Items (Critical!)
**Single item** → no dash, no list:
```yaml
triggers:
  trigger: time
  at: "07:00"
conditions:
  condition: state
  entity_id: input_boolean.guest_mode
  state: "off"
actions:
  action: light.turn_on
  entity_id: light.living_room_lights
```

**Multiple items** → list with dashes:
```yaml
triggers:
  - trigger: time
    at: "07:00"
  - trigger: state
    entity_id: input_boolean.alarm_clock
    to: "off"
conditions:
  - condition: state
    entity_id: input_boolean.guest_mode
    state: "off"
  - condition: time
    after: "16:00:00"
actions:
  - action: light.turn_on
    entity_id: light.living_room_lights
  - action: notify.all_iphones
    data:
      message: "Lights on!"
```

#### Quoting Rules
- **State values**: Always quoted `"on"`, `"off"`
- **Times**: Always quoted `"07:00"`, `"22:00:00"`
- **Custom states**: Unquoted when not boolean-like: `half`, `total`, `home`, `not_home`
- **Entity IDs**: Never quoted
- **Alias**: Always double-quoted

#### Template Conditions (Shorthand)
Use inline template strings instead of verbose `condition: template`:
```yaml
# Preferred (shorthand)
conditions:
  - "{{ states('input_select.sleep_mode') != 'total' }}"
  - "{{ is_state('media_player.spotify', 'playing') }}"

# Also valid for single condition
conditions: "{{ state_attr('climate.thermostat', 'temperature') >= 18 }}"
```

#### Entity ID Placement
Can be at same level as `action:` OR inside `data:`:
```yaml

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [basnijholt/home-assistant-config](https://github.com/basnijholt/home-assistant-config) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
