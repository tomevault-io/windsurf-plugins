---
trigger: always_on
description: FeedBack is a self-hosted web app for browsing, playing, and practicing interactive music notation, built around its own open `.sloppak` chart format. Charts come from importing Guitar Pro (GP5/GP8) or MusicXML, or from authoring in the built-in editor. It runs as a Docker container with a FastAPI backend (`server.py`), vanilla JavaScript frontend (`static/`), shared Python libraries (`lib/`), and an extensive plugin system (`plugins/`). There are no frontend frameworks — everything is plain JS,
---

# FeedBack — AI Agent Guide

FeedBack is a self-hosted web app for browsing, playing, and practicing interactive music notation, built around its own open `.sloppak` chart format. Charts come from importing Guitar Pro (GP5/GP8) or MusicXML, or from authoring in the built-in editor. It runs as a Docker container with a FastAPI backend (`server.py`), vanilla JavaScript frontend (`static/`), shared Python libraries (`lib/`), and an extensive plugin system (`plugins/`). There are no frontend frameworks — everything is plain JS, HTML, and Tailwind CSS.

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
  sloppak.py           Sloppak format support
  loosefolder.py       Loose-folder XML chart support
  audio.py             OGG/MP3 audio handling
  retune.py            Pitch-shifting logic
  tunings.py           Tuning name/offset utilities
  gp2rs.py             Guitar Pro to arrangement XML conversion
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

`description`, `category`, and `icon` are **optional, additive v3 Pedalboard metadata** (surfaced in `/api/plugins`, consumed by the v3 Plugins page `static/v3/plugins-page.js`). `description` is a short one-sentence summary shown under the pedal name. `category` (`audio | creation | practice | game | tools`, free-form; unknown/absent → curated default → `"other"`) picks which pedalboard the plugin sits on. `icon` is an assets-relative thumbnail path (e.g. `"assets/thumb.png"`, ~square ~256×256, same containment rule as `styles`, served via `/api/plugins/<id>/assets/...`); if omitted the loader auto-detects `assets/thumb.png`, and plugins with no thumbnail get a default pedal graphic. All three are backward-compatible — omit them and the plugin still loads. See [docs/plugin-v3-ui.md](docs/plugin-v3-ui.md).

`styles` is the **opt-in** for self-hosted CSS (Principle II — prebuilt Tailwind, no Play CDN). Core's `static/tailwind.min.css` only contains classes scanned from core source at build time, so a plugin installed at runtime (community / NAS) that uses classes core didn't scan — especially arbitrary values like `text-[11px]` — renders unstyled. Declaring `styles` makes the frontend inject one versioned `<link rel="stylesheet">` into `<head>` (covering the plugin's screen *and* its settings panel) pointing at the plugin's own compiled stylesheet. The value is a **plugin-root-relative path that must live under `assets/`** (e.g. `"assets/plugin.css"`) so it serves through the sandboxed `/api/plugins/<id>/assets/...` route. Build it with `corePlugins: { preflight: false }` (utilities only — core ships the single base reset; don't duplicate it) and **never** the Tailwind Play CDN. Plugins that use only core-guaranteed utilities, or ship no Tailwind, omit `styles` and are byte-for-byte unaffected. Full authoring guide + scaffold: [docs/plugin-styles.md](docs/plugin-styles.md).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [got-feedBack/feedBack](https://github.com/got-feedBack/feedBack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-15 -->
