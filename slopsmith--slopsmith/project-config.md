---
trigger: always_on
description: Slopsmith is a self-hosted web app for browsing, playing, and practicing Rocksmith 2014 Custom DLC. It runs as a Docker container with a FastAPI backend (`server.py`), vanilla JavaScript frontend (`static/`), shared Python libraries (`lib/`), and an extensive plugin system (`plugins/`). There are no frontend frameworks — everything is plain JS, HTML, and Tailwind CSS.
---

# Slopsmith — AI Agent Guide

Slopsmith is a self-hosted web app for browsing, playing, and practicing Rocksmith 2014 Custom DLC. It runs as a Docker container with a FastAPI backend (`server.py`), vanilla JavaScript frontend (`static/`), shared Python libraries (`lib/`), and an extensive plugin system (`plugins/`). There are no frontend frameworks — everything is plain JS, HTML, and Tailwind CSS.

## Architecture Quick Reference

```
server.py              FastAPI app — library API, WebSocket highway, plugin loading
static/
  app.js               Main frontend — screens, library views, player, plugin loader
  highway.js           Canvas note highway renderer (createHighway factory)
  index.html           Single-page app shell
  style.css            Custom CSS loaded alongside Tailwind
lib/
  song.py              Core data models (Note, Chord, Arrangement, Song)
  psarc.py             PSARC archive reading and extraction
  sloppak.py           Sloppak format support
  audio.py             WEM/OGG/MP3 audio handling
  retune.py            Pitch-shifting logic
  tunings.py           Tuning name/offset utilities
  gp2rs.py             Guitar Pro to Rocksmith XML conversion
  gp2midi.py           Guitar Pro to MIDI
plugins/
  __init__.py           Plugin discovery, loading, requirements install
  <plugin_name>/        Each plugin is its own directory (often a git submodule)
tests/
  test_*.py             pytest test suite
```

## Plugin System

Plugins are the primary extension point. Each plugin lives in `plugins/<name>/` with a `plugin.json` manifest. Plugins are typically their own git repositories — see [CONTRIBUTING.md](CONTRIBUTING.md) for the licensing policy (curated plugins should be AGPL-3.0 or AGPL-compatible: MIT, BSD, Apache-2.0).

```json
{
  "id": "my_plugin",
  "name": "My Plugin",
  "version": "1.0.0",
  "private": false,
  "type": "visualization",
  "nav": { "label": "My Plugin", "screen": "plugin-my_plugin" },
  "screen": "screen.html",
  "script": "screen.js",
  "styles": "assets/plugin.css",
  "routes": "routes.py",
  "settings": {
    "html": "settings.html",
    "server_files": ["my_plugin.db", "my_plugin_models/"]
  },
  "diagnostics": {
    "server_files": ["my_plugin.diag.json"],
    "callable": "diagnostics:collect"
  }
}
```

All fields except `id` and `name` are optional. Plugins can have any combination of frontend (screen/script), backend (routes), and settings.

`version` and `private` are advisory metadata — the plugin loader does not currently consume them, but plugins commonly include them for publishing/tooling purposes.

`styles` is the **opt-in** for self-hosted CSS (Principle II — prebuilt Tailwind, no Play CDN). Core's `static/tailwind.min.css` only contains classes scanned from core source at build time, so a plugin installed at runtime (community / NAS) that uses classes core didn't scan — especially arbitrary values like `text-[11px]` — renders unstyled. Declaring `styles` makes the frontend inject one versioned `<link rel="stylesheet">` into `<head>` (covering the plugin's screen *and* its settings panel) pointing at the plugin's own compiled stylesheet. The value is a **plugin-root-relative path that must live under `assets/`** (e.g. `"assets/plugin.css"`) so it serves through the sandboxed `/api/plugins/<id>/assets/...` route. Build it with `corePlugins: { preflight: false }` (utilities only — core ships the single base reset; don't duplicate it) and **never** the Tailwind Play CDN. Plugins that use only core-guaranteed utilities, or ship no Tailwind, omit `styles` and are byte-for-byte unaffected. Full authoring guide + scaffold: [docs/plugin-styles.md](docs/plugin-styles.md).

`settings.server_files` is the **opt-in** for the unified Settings export/import flow (slopsmith#113). It's a list of relpaths under `context["config_dir"]` that the plugin wants included in user-triggered backups. A trailing `/` denotes a directory (recurse). Plugins that omit this field have no server-side files exported; their state lives entirely in browser `localStorage`, which is bundled wholesale on every export. Rules:
- Relpaths only. Absolute paths, drive letters, `..` segments, and backslashes are rejected at load time with a `[Plugin]` warning.
- The same allowlist is consulted at both export and import: a bundle that references a file the *importing host*'s manifest no longer declares is skipped with a warning (handles plugin updates between export and import). A bundle that references a file your host's manifest never declared is also skipped — no surprise writes.
- Files are encoded as `{"encoding": "json", "data": <parsed>}` for `.json` files that parse cleanly (diff-friendly), `{"encoding": "base64", "data": "..."}` otherwise (sqlite, model blobs, IRs).
- Plugins own their internal data migration. Importing a bundle whose data schema predates your current code restores bytes verbatim — your plugin must cope at next load.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [slopsmith/slopsmith](https://github.com/slopsmith/slopsmith) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-14 -->
