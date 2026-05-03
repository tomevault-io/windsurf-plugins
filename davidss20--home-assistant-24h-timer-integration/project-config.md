---
trigger: always_on
description: **NEVER create additional documentation/guide files (*.md) unless explicitly requested!**
---

# Timer 24H Integration - Cursor Rules

## ⚠️ CRITICAL RULES - READ FIRST

### Documentation Files Policy

**NEVER create additional documentation/guide files (*.md) unless explicitly requested!**

**Allowed documentation files ONLY:**
- ✅ `README.md` - Main project documentation (can be updated)
- ✅ `CHANGELOG.md` - Version history (can be updated)

**FORBIDDEN to create:**
- ❌ Any other `.md` files (guides, tutorials, info files, etc.)
- ❌ `CONTRIBUTING.md`
- ❌ `UPGRADE_INSTRUCTIONS.md`
- ❌ `CACHE_BUSTING_INFO.md`
- ❌ `SUMMARY_OF_CHANGES.md`
- ❌ Any other documentation files

**Rationale:**
- Keep documentation centralized in README.md
- Avoid documentation sprawl
- Maintain single source of truth
- Reduce maintenance burden

**When user asks for documentation:**
1. Update existing `README.md` or `CHANGELOG.md`
2. Add to code comments/docstrings
3. Only create new `.md` file if user **explicitly** requests it by name

---

## Project Context

This is a Home Assistant custom integration that provides a 24-hour timer with automatic entity control.
The project includes both a Python backend (Home Assistant integration) and a TypeScript/Lit frontend (Lovelace card).

## Languages & Frameworks

- **Backend**: Python 3.11+, Home Assistant Integration
- **Frontend**: TypeScript, Lit (Web Components), Rollup
- **Documentation**: Hebrew and English

## Project Structure

```
home-assistant-24h-timer-integration/
├── custom_components/timer_24h/     # Backend integration
│   ├── __init__.py                  # Main integration logic + card installation
│   ├── config_flow.py               # Configuration flow
│   ├── coordinator.py               # Data coordinator
│   ├── sensor.py                    # Sensor entity
│   ├── const.py                     # Constants
│   ├── manifest.json                # Integration manifest
│   └── dist/                        # Built frontend files
│       ├── timer-24h-card.js
│       └── timer-24h-card-editor.js
├── timer-24h-card.ts                # Frontend card source
├── timer-24h-card-editor.ts         # Frontend editor source
└── README.md                        # Documentation
```

## Code Style & Standards

### Python (Backend)

1. **Follow Home Assistant standards**:
   - Use type hints for all function parameters and return values
   - Use `async`/`await` for all I/O operations
   - Use Home Assistant's logger (`_LOGGER`)
   - Follow Home Assistant's naming conventions

2. **Imports**:
   - Group imports: standard library, third-party, Home Assistant, local
   - Use `from __future__ import annotations` at the top

3. **Error Handling**:
   - Always use try-except blocks for external operations
   - Log errors with appropriate severity
   - Provide user-friendly error messages

4. **Example**:
   ```python
   async def async_example(hass: HomeAssistant, entry: ConfigEntry) -> bool:
       """Example function with proper type hints."""
       try:
           result = await some_async_operation()
           _LOGGER.info("Operation successful: %s", result)
           return True
       except Exception as err:
           _LOGGER.error("Operation failed: %s", err)
           return False
   ```

### TypeScript (Frontend)

1. **Use Lit Web Components**:
   - Extend `LitElement` for custom elements
   - Use decorators: `@customElement`, `@property`, `@state`
   - Use `html` template literal for rendering

2. **Naming**:
   - Classes: PascalCase (e.g., `Timer24HCard`)
   - Properties: camelCase (e.g., `timeSlots`)
   - CSS classes: kebab-case (e.g., `timer-container`)

3. **Type Safety**:
   - Always define interfaces for complex objects
   - Use proper types, avoid `any`
   - Define HomeAssistant types properly

4. **Example**:
   ```typescript
   @customElement('timer-24h-card')
   export class Timer24HCard extends LitElement {
     @property({ attribute: false }) hass?: HomeAssistant;
     @state() private config?: CardConfig;
   }
   ```

## Critical Features & Behaviors

### 1. Cache Busting (IMPORTANT!)

**Always maintain cache busting logic in `__init__.py`**:
- Resource URL must include version parameter: `?v=X.X.X`
- Version must be read from `manifest.json`
- Update resource URL when version changes

```python
url = f"/local/timer-24h-card/timer-24h-card.js?v={version}"
```

### 2. Resource Registration

**The integration automatically**:
- Copies card files to `www/timer-24h-card/`
- Registers Lovelace resource with version parameter
- Updates resource on version change

**Never remove or modify** the `_async_register_lovelace_resource()` function without careful consideration.

### 3. Time Slots

- Always use 48 slots (24 hours × 2 half-hour segments)
- Format: `{hour: 0-23, minute: 0|30, isActive: boolean}`
- Persist in coordinator's data

### 4. Home Presence

- Check sensors according to logic (OR/AND)
- Only activate entities when "at home"
- Update every minute via coordinator

## Version Management

### When Updating Version:

1. **Update `manifest.json`**: Change `"version": "X.X.X"`
2. **Update `VERSION` file**: Change version number
3. **Update `CHANGELOG.md`**: Document changes
4. **Build frontend**: Run `npm run build`
5. **Copy to dist**: Ensure files in `custom_components/timer_24h/dist/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [davidss20/home-assistant-24h-timer-integration](https://github.com/davidss20/home-assistant-24h-timer-integration) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
