---
trigger: always_on
description: The LEDMatrix project uses a plugin-based architecture. All display
---

# LEDMatrix Plugin Development Rules

## Plugin System Overview

The LEDMatrix project uses a plugin-based architecture. All display
functionality (except core calendar) is implemented as plugins that are
dynamically loaded from the directory configured by
`plugin_system.plugins_directory` in `config.json` — the default is
`plugin-repos/` (per `config/config.template.json:130`).

> **Fallback note (scoped):** `PluginManager.discover_plugins()`
> (`src/plugin_system/plugin_manager.py:154`) only scans the
> configured directory — there is no fallback to `plugins/` in the
> main discovery path. A fallback to `plugins/` does exist in two
> narrower places:
> - `store_manager.py:1700-1718` — store operations (install/update/
>   uninstall) check `plugins/` if the plugin isn't found in the
>   configured directory, so plugin-store flows work even when your
>   dev symlinks live in `plugins/`.
> - `schema_manager.py:70-80` — `get_schema_path()` probes both
>   `plugins/` and `plugin-repos/` for `config_schema.json` so the
>   web UI form generation finds the schema regardless of where the
>   plugin lives.
>
> The dev workflow in `scripts/dev/dev_plugin_setup.sh` creates
> symlinks under `plugins/`, which is why the store and schema
> fallbacks exist. For day-to-day development, set
> `plugin_system.plugins_directory` to `plugins` so the main
> discovery path picks up your symlinks.

## Plugin Structure

### Required Files
- **manifest.json**: Plugin metadata, entry point, class name, dependencies
- **manager.py**: Main plugin class (must inherit from `BasePlugin`)
- **config_schema.json**: JSON schema for plugin configuration validation
- **requirements.txt**: Python dependencies (if any)
- **README.md**: Plugin documentation

### Plugin Class Requirements
- Must inherit from `src.plugin_system.base_plugin.BasePlugin`
- Must implement `update()` method for data fetching
- Must implement `display()` method for rendering
- Should implement `validate_config()` for configuration validation
- Optional: Override `has_live_content()` for live priority features

## Plugin Development Workflow

### 1. Creating a New Plugin

**Option A: Use dev_plugin_setup.sh (Recommended)**
```bash
# Link from GitHub
./scripts/dev/dev_plugin_setup.sh link-github <plugin-name>

# Link local repository
./scripts/dev/dev_plugin_setup.sh link <plugin-name> <path-to-repo>
```

**Option B: Manual Setup**
1. Create directory in `plugin-repos/<plugin-id>/` (or `plugins/<plugin-id>/`
   if you're using the dev fallback location)
2. Add `manifest.json` with required fields
3. Create `manager.py` with plugin class
4. Add `config_schema.json` for configuration
5. Enable plugin in `config/config.json` under `"<plugin-id>": {"enabled": true}`

### 2. Plugin Configuration

Plugins are configured in `config/config.json`:
```json
{
  "<plugin-id>": {
    "enabled": true,
    "display_duration": 15,
    "live_priority": false,
    "high_performance_transitions": false,
    "transition": {
      "type": "redraw",
      "speed": 2,
      "enabled": true
    },
    // ... plugin-specific config
  }
}
```

### 3. Testing Plugins

**On Development Machine:**
- Run the dev preview server: `python3 scripts/dev_server.py` (then
  open `http://localhost:5001`) — renders plugins in the browser
  without running the full display loop
- Or run the full display in emulator mode:
  `python3 run.py --emulator` (or equivalently
  `EMULATOR=true python3 run.py`, or `./scripts/dev/run_emulator.sh`).
  The `-e`/`--emulator` CLI flag is defined in `run.py:19-20`.
- Test plugin loading: Check logs for plugin discovery and loading
- Validate configuration: Ensure config matches `config_schema.json`

**On Raspberry Pi:**
- Deploy and test on actual hardware
- Monitor logs: `journalctl -u ledmatrix -f` (if running as service)
- Check plugin status in web interface

### 4. Plugin Development Best Practices

**Code Organization:**
- Keep plugin code in `plugin-repos/<plugin-id>/` (or its dev-time
  symlink in `plugins/<plugin-id>/`)
- Use shared assets from `assets/` directory when possible
- Follow existing plugin patterns — canonical sources live in the
  [`ledmatrix-plugins`](https://github.com/ChuckBuilds/ledmatrix-plugins)
  repo (`plugins/hockey-scoreboard/`, `plugins/football-scoreboard/`,
  `plugins/clock-simple/`, etc.)
- Place shared utilities in `src/common/` if reusable across plugins

**Configuration Management:**
- Use `config_schema.json` for validation
- Store secrets in `config/config_secrets.json` under the same plugin
  id namespace as the main config — they're deep-merged into the main
  config at load time (`src/config_manager.py:162-172`), so plugin
  code reads them directly from `config.get(...)` like any other key
- There is no separate `config_secrets` reference field
- Validate all required fields in `validate_config()`

**Error Handling:**
- Use plugin's logger: `self.logger.info/error/warning()`
- Handle API failures gracefully
- Cache data to avoid excessive API calls
- Provide fallback displays when data unavailable

**Performance:**
- Use `cache_manager` for API response caching
- Implement background data fetching if needed
- Use `high_performance_transitions` for smoother animations
- Optimize rendering for Pi's limited resources


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ChuckBuilds/LEDMatrix](https://github.com/ChuckBuilds/LEDMatrix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
