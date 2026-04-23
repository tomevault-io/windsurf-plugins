---
trigger: always_on
description: > This file is read by AI coding assistants (Claude Code, Zencoder, Copilot, etc.)
---

# Selora AI — Home Assistant Integration

> This file is read by AI coding assistants (Claude Code, Zencoder, Copilot, etc.)
> to maintain consistency across developers and models. Keep it up to date.

## What This Is

A custom Home Assistant integration (`custom_components/selora_ai/`) that acts as a "smart butler":
- Analyzes device states and usage patterns via LLM (Anthropic Claude or local Ollama)
- Auto-generates HA automations (disabled, prefixed `[Selora AI]` for user review)
- Accepts natural language commands via the Selora panel and Home Assistant Assist
- Discovers and onboards network devices during initial setup

## Architecture

```
HA entity registry / state machine / recorder (SQLite)
    |
    v
DataCollector  ──snapshot──>  LLMClient (Anthropic API or local Ollama)
    |                              |
    |                         suggestions
    v                              v
logging + sensors         automations.yaml (disabled) + reload
```

## Project Structure

```
custom_components/selora_ai/
├── __init__.py          # Integration setup/teardown, entry routing
├── config_flow.py       # UI config flow (LLM setup → device discovery → area assignment → results)
├── collector.py         # Hourly data collection + LLM automation writer
├── llm_client.py        # Unified LLM client (Anthropic + Ollama)
├── device_manager.py    # Device discovery, pairing, area assignment, dashboard generation
├── button.py            # Hub action buttons (Discover, Scan, Cleanup, Reset)
├── sensor.py            # Hub sensors (Status, Devices, Discovery, Last Activity)
├── const.py             # Constants, config keys, known integrations database
├── manifest.json        # HA integration manifest
├── strings.json         # UI strings for config flow
├── translations/en.json # English translations (must match strings.json)
├── brand/               # Logo and icon assets
└── frontend/
    └── src/
        ├── panel.js                  # LitElement host (properties, lifecycle, render dispatch)
        └── panel/
            ├── render-automations.js # Automation list, cards, flowchart, unavailable modal
            ├── render-chat.js        # Chat messages, YAML editor, new-automation dialog
            ├── render-settings.js    # Settings tab
            ├── render-suggestions.js # Suggestion cards
            ├── render-version-history.js # Version history drawer + diff viewer
            ├── stale-automations.js  # Stale detection helpers + stale modal/detail
            ├── automation-crud.js    # CRUD websocket calls
            ├── automation-management.js # Bulk edit, enable/disable, filter
            ├── session-actions.js    # Session list actions
            ├── suggestion-actions.js # Accept/dismiss/snooze suggestion actions
            ├── chat-actions.js       # Send message, streaming
            └── styles/               # CSS-in-JS style modules
```

## Key Conventions

### Code Style
- Python 3.12+, async/await throughout
- `from __future__ import annotations` in every file
- Type hints using modern syntax (`str | None`, not `Optional[str]`)
- Logging via `_LOGGER = logging.getLogger(__name__)`
- No hardcoded secrets — API keys come from user config entry, never from constants

### Home Assistant Patterns
- Config entries have an `entry_type` field: `"llm_config"` or `"device_onboarding"`
- Entity platforms: `sensor`, `button` (registered in `PLATFORMS` list in `__init__.py`)
- All entities use `_attr_has_entity_name = True` and reference the hub device `(DOMAIN, "selora_ai_hub")`
- Dispatcher signals for real-time updates: `SIGNAL_DEVICES_UPDATED`, `SIGNAL_ACTIVITY_LOG`
- Dashboard generation uses HA's Lovelace API (`LovelaceStorage.async_save`), not direct file writes

### Config Flow
- First entry: LLM provider selection → credentials → device discovery → area assignment → results
- Subsequent "Add Entry": skips LLM config, goes straight to device discovery
- Anthropic step shows a form for the user's API key (never auto-configure)
- `strings.json` and `translations/en.json` must always stay in sync
- Step IDs must match keys in strings.json: `user`, `anthropic`, `ollama`, `select_devices`, `results`

### Frontend File Organization
- `panel.js` is the LitElement host — it owns properties, lifecycle, and render dispatch only. Do not add feature logic or templates here.
- Each tab/feature has its own `render-*.js` file under `panel/`. New features (modals, sections, views) go in dedicated files, not appended to existing render files.
- Action helpers (websocket calls, state mutations) go in `*-actions.js` or `*-crud.js` files, not inline in templates.
- Configurable values (like stale days threshold) should come from `host._config` (populated via websocket), not hardcoded as JS constants. This keeps the backend `const.py` as the single source of truth.
- Keep individual `panel/` files under ~400 lines. If a file grows past that, split the new feature into its own module.
- Run `node build.js` from `frontend/` after any source change — the bundled `panel.js` is committed.

### Git & Branching
- Main branch: `main`
- Feature branches: `selora-ai-<feature>`
- Commit messages: conventional commits (`feat:`, `fix:`, `refactor:`, `docs:`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SeloraHomes) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
