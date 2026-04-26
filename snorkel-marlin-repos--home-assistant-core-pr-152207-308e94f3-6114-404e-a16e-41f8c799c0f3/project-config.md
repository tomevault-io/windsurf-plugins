---
trigger: always_on
description: This repository contains the core of Home Assistant, a Python 3 based home automation application.
---

# GitHub Copilot & Claude Code Instructions

This repository contains the core of Home Assistant, a Python 3 based home automation application.

## Integration Quality Scale

Home Assistant uses an Integration Quality Scale to ensure code quality and consistency. The quality level determines which rules apply:

### Quality Scale Levels
- **Bronze**: Basic requirements (ALL Bronze rules are mandatory)
- **Silver**: Enhanced functionality 
- **Gold**: Advanced features
- **Platinum**: Highest quality standards

### How Rules Apply
1. **Check `manifest.json`**: Look for `"quality_scale"` key to determine integration level
2. **Bronze Rules**: Always required for any integration with quality scale
3. **Higher Tier Rules**: Only apply if integration targets that tier or higher
4. **Rule Status**: Check `quality_scale.yaml` in integration folder for:
   - `done`: Rule implemented
   - `exempt`: Rule doesn't apply (with reason in comment)
   - `todo`: Rule needs implementation

### Example `quality_scale.yaml` Structure
```yaml
rules:
  # Bronze (mandatory)
  config-flow: done
  entity-unique-id: done
  action-setup:
    status: exempt
    comment: Integration does not register custom actions.
  
  # Silver (if targeting Silver+)
  entity-unavailable: done
  parallel-updates: done
  
  # Gold (if targeting Gold+)
  devices: done
  diagnostics: done
  
  # Platinum (if targeting Platinum)
  strict-typing: done
```

**When Reviewing/Creating Code**: Always check the integration's quality scale level and exemption status before applying rules.

## Code Review Guidelines

**When reviewing code, do NOT comment on:**
- **Missing imports** - We use static analysis tooling to catch that
- **Code formatting** - We have ruff as a formatting tool that will catch those if needed (unless specifically instructed otherwise in these instructions)

## Python Requirements

- **Compatibility**: Python 3.13+
- **Language Features**: Use the newest features when possible:
  - Pattern matching
  - Type hints
  - f-strings (preferred over `%` or `.format()`)
  - Dataclasses
  - Walrus operator

### Strict Typing (Platinum)
- **Comprehensive Type Hints**: Add type hints to all functions, methods, and variables
- **Custom Config Entry Types**: When using runtime_data:
  ```python
  type MyIntegrationConfigEntry = ConfigEntry[MyClient]
  ```
- **Library Requirements**: Include `py.typed` file for PEP-561 compliance

## Code Quality Standards

- **Formatting**: Ruff
- **Linting**: PyLint and Ruff
- **Type Checking**: MyPy
- **Testing**: pytest with plain functions and fixtures
- **Language**: American English for all code, comments, and documentation (use sentence case, including titles)

### Writing Style Guidelines
- **Tone**: Friendly and informative
- **Perspective**: Use second-person ("you" and "your") for user-facing messages
- **Inclusivity**: Use objective, non-discriminatory language
- **Clarity**: Write for non-native English speakers
- **Formatting in Messages**:
  - Use backticks for: file paths, filenames, variable names, field entries
  - Use sentence case for titles and messages (capitalize only the first word and proper nouns)
  - Avoid abbreviations when possible

## Code Organization

### Core Locations
- Shared constants: `homeassistant/const.py` (use these instead of hardcoding)
- Integration structure:
  - `homeassistant/components/{domain}/const.py` - Constants
  - `homeassistant/components/{domain}/models.py` - Data models
  - `homeassistant/components/{domain}/coordinator.py` - Update coordinator
  - `homeassistant/components/{domain}/config_flow.py` - Configuration flow
  - `homeassistant/components/{domain}/{platform}.py` - Platform implementations

### Common Modules
- **coordinator.py**: Centralize data fetching logic
  ```python
  class MyCoordinator(DataUpdateCoordinator[MyData]):
      def __init__(self, hass: HomeAssistant, client: MyClient, config_entry: ConfigEntry) -> None:
          super().__init__(
              hass, 
              logger=LOGGER, 
              name=DOMAIN, 
              update_interval=timedelta(minutes=1),
              config_entry=config_entry,  # ✅ Pass config_entry - it's accepted and recommended
          )
  ```
- **entity.py**: Base entity definitions to reduce duplication
  ```python
  class MyEntity(CoordinatorEntity[MyCoordinator]):
      _attr_has_entity_name = True
  ```

### Runtime Data Storage
- **Use ConfigEntry.runtime_data**: Store non-persistent runtime data
  ```python
  type MyIntegrationConfigEntry = ConfigEntry[MyClient]
  
  async def async_setup_entry(hass: HomeAssistant, entry: MyIntegrationConfigEntry) -> bool:
      client = MyClient(entry.data[CONF_HOST])
      entry.runtime_data = client
  ```

### Manifest Requirements
- **Required Fields**: `domain`, `name`, `codeowners`, `integration_type`, `documentation`, `requirements`
- **Integration Types**: `device`, `hub`, `service`, `system`, `helper`
- **IoT Class**: Always specify connectivity method (e.g., `cloud_polling`, `local_polling`, `local_push`)
- **Discovery Methods**: Add when applicable: `zeroconf`, `dhcp`, `bluetooth`, `ssdp`, `usb`
- **Dependencies**: Include platform dependencies (e.g., `application_credentials`, `bluetooth_adapters`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/snorkel-marlin-repos) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
