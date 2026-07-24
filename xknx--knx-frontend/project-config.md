---
trigger: always_on
description: You are an assistant helping with development of the Home Assistant KNX Frontend Panel. This is a TypeScript/Lit web application that provides KNX integration management within Home Assistant.
---

# KNX Frontend AI Agent Instructions

You are an assistant helping with development of the Home Assistant KNX Frontend Panel. This is a TypeScript/Lit web application that provides KNX integration management within Home Assistant.

Always follow the general Home Assistant frontend guidance in [homeassistant-frontend/AGENTS.md](homeassistant-frontend/AGENTS.md) for shared patterns (dialogs, forms, view transitions, accessibility, etc.). This file only adds KNX-specific context and repo-local conventions; avoid duplicating upstream rules.

## KNX Stack Architecture

### Projects & Repositories

| Layer                         | Role in the stack                                                      | GitHub repository                                                            |
| ----------------------------- | ---------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| **Home Assistant Core**       | Main Python application and framework                                  | https://github.com/home-assistant/core                                       |
| **KNX integration (in-tree)** | Glue code that wires KNX into HA (`homeassistant/components/knx`)      | https://github.com/home-assistant/core/tree/dev/homeassistant/components/knx |
| **KNX integration proxy**     | Issue & discussion tracker for the integration                         | https://github.com/XKNX/knx-integration                                      |
| **XKNX**                      | Asynchronous Python KNX/IP library that does the heavy lifting         | https://github.com/XKNX/xknx                                                 |
| **KNX-frontend**              | TypeScript + Lit-Element HomeAssistant panel; packed as a Python wheel | https://github.com/XKNX/knx-frontend                                         |

### How Components Interact

1. **Backend flow**: Integration instantiates XKNX for KNX/IP connection; entities proxy between HA state machine and XKNX group-address abstractions
2. **Frontend flow**: Panel registered via `panel_custom.async_register_panel()`, communicates via WebSocket API (`/api/websocket`)
   - **Registration**: Panel loads as iframe with admin-only access, served from `/knx_static/entrypoint.{hash}.js`
   - **Initialization**: Main component `knx-frontend` initializes KNX object with config entry and WebSocket services
   - **Router**: `knx-router` handles navigation between views (info, group monitor, project, entities)
   - **WebSocket Communication**: All backend communication via `hass.callWS()` with KNX-specific message types (`knx/info`, `knx/group_monitor_info`, `knx/create_entity`, etc.)
   - **Theming**: Inherits HA themes, applies KNX-specific CSS custom properties (`--knx-green`, `--knx-blue`)
   - **Components**: Built with Lit 3.x web components, uses HA design system (`<ha-*>` components)
   - **Real-time Updates**: Telegram subscription via WebSocket for live KNX bus monitoring
3. **Release pipeline**: Wheels composition (version pins + PyPI) - no git merges required
4. **Issue tracking**: Use XKNX/knx-integration for integration issues, XKNX/knx-frontend for UI issues

## Core Principles

### Architecture

- **Web Components (Lit 3.x)**: Use LitElement with `@customElement`, `@property`, `@state`
- **Strict TypeScript**: No `any` types, define interfaces for KNX data structures
- **Theming**: Use HA CSS variables and `this.hass.localize()` for text

### Naming Conventions

- Classes: PascalCase (`KnxGroupMonitorPanel`)
- Variables/Functions: camelCase (`processTelegram()`)
- Private members: underscore prefix (`_buffer`)
- Elements: `knx-` prefix for all custom elements (`knx-group-monitor`, `knx-telegram-info-dialog`)
- Files: lowercase with hyphens/underscores (`group-monitor.ts`)

### Code Quality

- **Linting**: ESLint + Prettier enforced (`yarn lint`, `yarn format`)
- **Error Handling**: Show `<ha-alert>` for errors, never fail silently
- **Resource Cleanup**: Unsubscribe WebSocket listeners on disconnect
- **Accessibility**: ARIA labels, keyboard navigation, WCAG AA contrast
- **No Console Logs**: Use proper logging utilities, never `console.log` in production code

## KNX Domain Knowledge

### Core Concepts

- **KNX**: Decentralized building automation protocol (EN 50090)
- **Group Addresses**: Logical addresses for device communication. Frontend supports all 3 formats:
  - **3-level**: `1/2/3` (Main/Middle/Sub - most common)
  - **2-level**: `1/2` (Main/Sub)
  - **Free**: `12345` (single number 0-65535)
- **Individual Addresses**: Physical device addresses (Area.Line.Device)
- **Telegrams**: KNX messages sent between devices on the bus
- **DPT (Datapoint Types)**: Data formats (DPT 1.001 = boolean, DPT 9.001 = temperature)
- **ETS**: Engineering Tool Software for KNX configuration

### Common Use Cases

- **Lighting**: On/off (DPT 1.001), dimming levels (DPT 5.001, 0-100%) → HA `light` entities
- **Covers**: Up/down commands (DPT 1.008), position feedback (DPT 5.001) → HA `cover` entities

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [XKNX/knx-frontend](https://github.com/XKNX/knx-frontend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
