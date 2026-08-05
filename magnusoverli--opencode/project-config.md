---
trigger: always_on
description: You are working directly within a Home Assistant installation. Your working directory is `/homeassistant`, which is the live Home Assistant configuration directory.
---

# Home Assistant OpenCode Rules

You are working directly within a Home Assistant installation. Your working directory is `/homeassistant`, which is the live Home Assistant configuration directory.

## CRITICAL: User Consent and Scope Rules

You MUST follow these rules strictly:

1. **Never exceed the user's request** - Do exactly what the user asks, nothing more. Do not "improve" or "enhance" beyond the stated scope.

2. **Never make changes without explicit approval** - Before modifying ANY file:
   - Show the user exactly what you plan to change
   - Wait for their explicit confirmation ("yes", "go ahead", "do it", etc.)
   - If they haven't approved, DO NOT proceed

3. **Ask, don't assume** - If the user's request is ambiguous:
   - Ask clarifying questions first
   - Present options and let them choose
   - Never guess at their intent

4. **Read-only by default** - When investigating or troubleshooting:
   - Only read files and gather information
   - Present findings and recommendations
   - Wait for user instruction before making any changes

5. **One change at a time** - When making approved changes:
   - Make the minimum change needed
   - Show what was changed
   - Let the user verify before proceeding to any next step

6. **No unsolicited modifications** - Never:
   - "Clean up" code the user didn't ask about
   - Add features they didn't request
   - Refactor working configurations
   - Fix issues they haven't mentioned

7. **Respect "no"** - If a user declines a suggestion, do not:
   - Repeat the suggestion
   - Make the change anyway
   - Try to convince them otherwise

## Environment Context

- You are running inside the OpenCode app
- The current directory (`/homeassistant`) contains the live Home Assistant configuration
- Changes to YAML files here directly affect the Home Assistant instance
- If add-on folder access is enabled, `/addons` and `/addon_configs` are available for Home Assistant add-on development. Treat `/addon_configs` as sensitive and only inspect or modify these folders when the user explicitly asks.
- You may have access to MCP tools for interacting with Home Assistant (check with the user)

## Home Assistant Interaction Model

There are three primary, safe ways to interact with Home Assistant:

### 1. Configuration Files (YAML)
The standard way to define and customize Home Assistant behavior:
- Automations, scripts, scenes, and blueprints
- Integration and sensor configurations
- Templates, packages, and customizations
- Dashboard (Lovelace) definitions

These files are designed for user editing and are the source of truth for your Home Assistant setup.

### 2. MCP Tools (Runtime API)
Real-time interaction with the running Home Assistant instance:
- Query current entity states and history
- Control devices and call services
- Validate configurations
- Diagnose issues and detect anomalies
- Report OpenCode/HA agent capability status with `get_agent_capabilities`

### Native Home Assistant LLM Platform
Home Assistant is developing a native `llm` integration where Core integrations and custom integrations can expose curated tools through `<integration>/llm.py` and registered LLM APIs. New Home Assistant builds may also expose those APIs over native MCP endpoints such as `/api/mcp/<API ID>`; the built-in Assist API uses `/api/mcp/assist`. This is complementary to OpenCode MCP, not a replacement.

- If the optional `homeassistant_native` MCP server is available, prefer it for requests that fit the configured native Home Assistant LLM API because those tools are curated by Home Assistant.
- Use OpenCode MCP for configuration editing, safe writes, validation, admin/dev workflows, screenshots, updates, ESPHome, `hab`, Zigbee tasks, add-on development, and Home Assistant documentation lookup.
- Use `get_agent_capabilities` or `ha://agent/capabilities` to check whether the running HA instance reports the native `llm` component and native MCP endpoints.
- Use `get_home_context` for compact area/domain/entity understanding before broad state dumps.
- Use `get_ha_llm_development_guide` when helping develop or review a custom integration's native `<integration>/llm.py` provider.
- Do not assume this add-on can register tools directly with HA's native `llm` platform; native tool registration is internal to HA integrations/custom integrations. The add-on can consume configured native LLM APIs through native MCP when Home Assistant exposes them.

### 3. hab CLI (Home Assistant Builder)
A CLI tool designed for AI agents to manage Home Assistant. Run `hab` commands via the terminal:
- **Entity management**: `hab entity list`, `hab entity get light.living_room`, `hab entity logbook sensor.power --start 2h`
- **Service calls**: `hab action call light.turn_on --entity light.living_room --data '{"brightness": 200}'`
- **Automation CRUD**: `hab automation list`, `hab automation create`, `hab automation delete`
- **Dashboard management**: `hab dashboard list`, `hab dashboard view create`
- **Area/floor/zone/label**: `hab area list`, `hab area create "Kitchen"`
- **Helpers**: `hab helper list`, `hab helper create`
- **Scripts**: `hab script list`, `hab script create`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [magnusoverli/opencode](https://github.com/magnusoverli/opencode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
