---
trigger: always_on
description: receiver/           — Receiver-page plugins (loaded via receiver/init.js)
---

# OpenWebRX+ Plugins — Development Guide

## Project structure

```text
receiver/           — Receiver-page plugins (loaded via receiver/init.js)
map/                — Map-page plugins (loaded via map/init.js)
docs/               — Documentation (GitHub Pages / Jekyll)
```

Each plugin is a folder under `receiver/` or `map/` containing at minimum `pluginname.js` and optionally `pluginname.css`. The CSS is auto-loaded unless `Plugins.pluginname.no_css = true`.

## Plugin manifest

- `receiver/plugins.json` lists every receiver plugin and every built-in OpenWebRX+ plugin. It is used by `plugin_loader` and to generate the README plugin tables.
- When adding, renaming, deprecating or changing the description/dependencies of a receiver plugin, update `receiver/plugins.json` and run `python3 tools/plugins.py`.
- Never edit the README tables between `<!-- plugins:<category>:start -->` and `<!-- plugins:<category>:end -->` by hand.
- New built-in OpenWebRX+ plugins get a `"category": "builtin"` entry and a commented-out line in `receiver/init.js.sample`.
- Third-party plugins get `"category": "thirdparty"` with `homepage`; add `url` only for single-file plugins that work with `Plugins.load()` and need no server-side setup.
- Map plugins are not in the manifest; their README table is edited by hand.
- Field reference: `DEVELOPMENT.md`, section "Adding a New Plugin to This Repository".
- Plugins that cannot work without administrator configuration use `"setup": "required"` in the manifest. `plugin_loader` only offers them when `plugin_options.<id>` is configured and invokes their `setup()` after loading.
- Developer documentation (plugin API, utils API, conventions) lives in `DEVELOPMENT.md`; keep it in sync when utils or the OpenWebRX+ plugin API change.

## Plugin options

- Plugins that take options from `init.js` expose `Plugins.<name>.setup(options)`, called after `Plugins.load()`.
- Options that are only read at runtime may be plain properties set after `Plugins.load()`.
- Never create `Plugins.<name>` before `Plugins.load()` - the loader then skips the plugin as already loaded.
- When migrating an existing plugin to `setup()`, keep the old way of passing options working as a fallback.
- `setup()` must also work after `init()` and apply the options to existing UI.
- Known plugins still to migrate: `uikit` (`Plugins.uikit.settings` before load), `tune_precise` (`Plugins.tune_precise_steps`).

## Plugin conventions

- **Namespace**: `Plugins.pluginname = Plugins.pluginname || {};`
- **Version**: `Plugins.pluginname._version = 0.1;`
- **init()**: Must return `true` on success, `false` on failure.
- **Dependencies**: Check with `Plugins.isLoaded('dep', minVersion)`.
- **Load order matters**: Use `await Plugins.load(...)` in `init.js`.
- **External scripts/styles**: Use `Plugins._load_script(url)` and `Plugins._load_style(url)`.
- **No CSS**: Set `Plugins.pluginname.no_css = true` before `init()` if no CSS file.

## Key infrastructure plugins

- **utils** (`receiver/utils/utils.js`) — `wrap_func()`, `on_ready()`, event system, `deepMerge()`, `fillTemplate()`. Almost all plugins depend on this.
- **uikit** (`receiver/uikit/uikit.js`) — Dockable panel, settings modal, plugin modals, toasts, loading overlays. Version 0.3+.
- **notify** (`receiver/notify/notify.js`) — Deprecated in favor of `uikit.toast()`. Has backward-compat shim.

## Utility plugin governance

- When adding **new functionality** to a utility plugin (for example `receiver/utils/utils.js`), always bump that plugin's `_version` and add a matching changelog entry in the file header comments.
- Always document new utility functions in two places:
  - thorough inline function comments in the JS source
  - plugin README updates with usage and examples
- Any plugin that depends on newly added utility functionality must require the exact new minimum version with `Plugins.isLoaded('utils', x)`.
- Keep utility functions backward-compatible.
- If backward compatibility cannot be preserved, stop and ask for direction before implementing a breaking change.

## Instruction alignment

- Always follow these project instructions together with `copilot-instructions.md` and any higher-priority assistant/system instructions.
- When new stable conventions are learned during implementation, update this file and related utility/plugin docs to keep guidance in sync.

## uikit migration

Existing plugins are being migrated to use uikit for their UI. Rules:

- The migrated plugin gets a new folder and name prefixed with `ui_` (e.g. `magic_key` → `ui_magic_key`).
- The original plugin is left untouched for backward compatibility.
- Migrated plugins capture `_baseUrl` at load time via `document.currentScript.src` and use it for auto-loading dependencies.
- Migrated plugins require `uikit >= 0.3` and `utils >= 0.6`.
- **Only bump dependency version checks** (`Plugins.isLoaded('uikit', x)` and `Plugins.isLoaded('utils', x)`) when the plugin actually uses a feature introduced in that version. Do not blindly bump to latest just because you touched the plugin. Current versions: `uikit = 0.5`, `utils = 0.9`.
- Use `var ui = Plugins.uikit;` as the local shorthand alias inside migrated plugin `init()` functions.

## uikit specifics


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [0xAF/openwebrxplus-plugins](https://github.com/0xAF/openwebrxplus-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
