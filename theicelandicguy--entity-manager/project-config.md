---
trigger: always_on
description: CLAUDE.md - AI Assistant Guide for Entity Manager
---

#
 CLAUDE.md - AI Assistant Guide for Entity Manager

This document provides comprehensive guidance for AI assistants working with the Entity Manager codebase.

## Project Overview

**Entity Manager** is a custom Home Assistant integration (v2.19.0) that provides a centralized interface for managing entities across all integrations and devices. It solves the common pain point of navigating through multiple settings pages to manage entities.

### Key Value Proposition
- Bulk enable/disable entities in seconds instead of minutes
- Organized tree view by Integration → Device → Entity
- Multiple smart grouping modes (Integration, Room/Area, Type, Device Name)
- Search and filter capabilities across all entities
- Advanced tools: entity renaming, YAML reference updater, firmware manager, automation impact analysis
- Theme system with 4 built-in themes and a custom theme editor
- Voice assistant support for hands-free control

## Repository Structure

```
entity-manager/
├── custom_components/entity_manager/   # Main integration code
│   ├── __init__.py                     # Entry point, service registration, panel setup
│   ├── config_flow.py                  # UI-based configuration flow
│   ├── const.py                        # Constants (DOMAIN, MAX_BULK_ENTITIES, VALID_ENTITY_ID)
│   ├── manifest.json                   # Integration metadata (v2.19.0)
│   ├── services.yaml                   # Service schema for HA UI
│   ├── strings.json                    # UI strings for config flow
│   ├── voice_assistant.py              # Voice intent handlers
│   ├── websocket_api.py                # WebSocket command handlers (16 commands)
│   ├── frontend/
│   │   ├── entity-manager-panel.js     # Custom web component UI (~9,600 lines)
│   │   └── entity-manager-panel.css    # External stylesheet (~2,500 lines)
│   └── translations/
│       └── en.json                     # English translations
├── sentences/en/
│   └── entity_manager.yaml             # Voice assistant sentence patterns
├── tests/
│   └── test_const.py                   # Basic constant tests
├── hacs.json                           # HACS configuration
├── info.md                             # HACS info page
├── README.md                           # User documentation
├── INSTALL.md                          # Installation guide
├── QUICKSTART.md                       # Quick reference
├── STRUCTURE.md                        # Code structure documentation
├── PROJECT_SUMMARY.md                  # Project overview
├── package.json                        # Node dev dependencies (eslint)
└── LICENSE                             # MIT License
```

## Architecture

### Data Flow

```
User Interface (JavaScript Web Component)
         ↓ WebSocket
WebSocket API (Python handlers)
         ↓
Home Assistant Core APIs
         ↓
Entity Registry / Device Registry / Area Registry / Label Registry
```

### Component Responsibilities

| Component | Responsibility |
|-----------|----------------|
| `__init__.py` | Integration setup, service registration, sidebar panel + frontend resource registration |
| `config_flow.py` | Handle UI-based integration setup (single-step, no config options) |
| `const.py` | Domain, bulk operation limits, entity ID validation regex |
| `websocket_api.py` | 18 WebSocket handlers covering entity ops, registry access, YAML management |
| `voice_assistant.py` | Intent handlers for enable/disable voice commands |
| `entity-manager-panel.js` | Complete frontend UI as a custom element |
| `entity-manager-panel.css` | External stylesheet loaded at startup |

## Key Patterns and Conventions

### Python Code Style

1. **Async/Await**: All HA interactions use async patterns
2. **Entity Registry**: Access via `er.async_get(hass)`
3. **Device Registry**: Access via `dr.async_get(hass)`
4. **Logging**: Use module-level `_LOGGER = logging.getLogger(__name__)`
5. **Error Handling**: Wrap registry operations in try/except; return structured error responses
6. **Type Hints**: Use Python type hints (e.g., `dict[str, Any]`)
7. **Bulk Limit**: Never exceed `MAX_BULK_ENTITIES = 500` in bulk operations
8. **Entity ID Validation**: Use `VALID_ENTITY_ID` regex from `const.py` before acting

### JavaScript Code Style

1. **Vanilla JS**: No frameworks — pure ES6+ JavaScript
2. **Web Components**: `EntityManagerPanel extends HTMLElement`
3. **State Management**: Instance properties persisted to `localStorage` (see State section)
4. **HA Integration**: Use `this.hass.callWS()` for WebSocket calls
5. **Styling**: External CSS file + HA CSS variables; avoid inline styles for non-dynamic values
6. **DOM Updates**: Batch changes, use `requestAnimationFrame` where possible
7. **Keyboard Shortcuts**: Ctrl+Z (undo), Ctrl+Y (redo), Ctrl+E (export), Ctrl+G (grouping), Ctrl+B (sidebar)

### Home Assistant Conventions

1. **Domain**: `entity_manager` (used consistently across all files)
2. **Services**: `entity_manager.enable_entity`, `entity_manager.disable_entity`
3. **WebSocket Types**: `entity_manager/<command>` naming pattern
4. **Admin Required**: All WebSocket commands use `@websocket_api.require_admin`
5. **IoT Class**: `calculated` (not `local_push`)
6. **Frontend Resources**: Registered at `/api/entity_manager/frontend` via `__init__.py`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheIcelandicguy/entity-manager](https://github.com/TheIcelandicguy/entity-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
