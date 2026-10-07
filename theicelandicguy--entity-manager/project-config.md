---
trigger: always_on
description: Home Assistant custom integration, domain `entity_manager`, **v3.6.0**.
---

# AGENTS.md — Entity Manager

Home Assistant custom integration, domain `entity_manager`, **v3.6.0**.
Repo `TheIcelandicguy/entity-manager`; source at `E:\entity-manager`.

An admin-only sidebar panel ("Entity Manager", `mdi:tune`) for viewing, enabling,
disabling, renaming, auditing and bulk-managing every entity across every
integration, plus device/area/label assignment and firmware updates. Built for
large installs. No Python requirements; `integration_type: service`,
`iot_class: calculated`, `config_flow: true`, `dependencies: ["frontend"]`,
minimum HA 2024.1.0.

`OVERVIEW.md` in this repo is current and was verified against source — use it
when you need more depth than this file. The unverified root docs that used to
sit beside it (STRUCTURE.md, PROJECT_SUMMARY.md, QUICKSTART.md, DEVREF.md,
INSTALL.md, cursorrules.md) were deleted on 2026-09-22; README.md covers
installation, and git history has the rest.

Before ending a session, run `python check_docs.py` and update this file.
The script checks CLAUDE.md for stale paths, constants, line counts, test counts
and services. Keep this guide aligned with it and validate both when editing.

## Layout

All paths below are relative to the repo root. Note that `tests/` lives at the
**root**, not inside the component directory.

| Path | Responsibility |
|---|---|
| `custom_components/entity_manager/__init__.py` | 132 lines. Registers the static path `/api/entity_manager/frontend` (served with long cache headers), the WS API, voice intents, the two services, and the sidebar panel (`require_admin=True`), and installs the voice sentences
(`async_install_sentences`). The panel JS `?v=` key is `<manifest version>-<first 10 hex of the file's SHA-256>`, so any redeploy that changes the panel reaches browsers and Companion apps after an HA restart, even without a version bump. |
| `.../const.py` | `DOMAIN`, `MAX_BULK_ENTITIES = 500`, `VALID_ENTITY_ID = ^[a-z][a-z0-9_]*\.[a-z0-9_]+$`. No VERSION constant — the version lives only in `manifest.json` and `package.json`. |
| `.../websocket_api.py` | 1,985 lines. All 24 WS handlers, `async_setup_ws_api()`, and the `enable_entity()` / `disable_entity()` helpers the services reuse. |
| `.../voice_assistant.py` | Enable/Disable intent handlers and `resolve_voice_target()`, which turns what was said into an entity ID. It returns a `VoiceResolution` (entity, how it matched, the other candidates, the near misses); `_resolve_entity_id` is the thin wrapper the intents use, so the panel and intents share entity matching; routing and pipeline execution are separate checks. |
| `.../voice_sentences.py` | 168 lines. Installing the sentence files into `<config>/custom_sentences/<lang>/` and reporting on them (`sentence_status`). Kept out of `__init__` because `websocket_api` reads the same files and cannot import `__init__` without a cycle. |
| `.../sentences/en/entity_manager.yaml` | Voice sentences, copied into `<config>/custom_sentences/en/` at startup. Inside the component, because only that directory is deployed. |
| `.../config_flow.py` | Single step, unique-ID guarded, no options flow. |
| `.../frontend/entity-manager-panel.js` | 19,197 lines. The whole UI as one `EntityManagerPanel extends HTMLElement`. |
| `.../frontend/entity-manager-panel.css` | 7,989 lines, all `--em-*` variables. |
| `tests/` | Python tests: `test_const.py`, `test_websocket_api.py`, `test_voice_assistant.py`, `conftest.py`. |
| `.../frontend/tests/` | Vitest specs + `vitest.setup.js`. |
| `deploy.ps1` | Thin wrapper over `E:\tools\deploy-to-ha.ps1` (see Deploy). No `sync-to-ha.ps1` helper is checked in; that old name is still used locally on this machine only. |
| `check_docs.py` | Verifies this file against the repo. Run it before ending a session. |
| `ruff.toml` | Pins Ruff lint rules for this repo so CI does not inherit widened future defaults. |

## Architecture

```
entity-manager-panel.js  --this.hass.callWS-->  websocket_api.py  -->  HA registries
```

- The panel talks to the backend **only** over HA's WebSocket bus. There is no
  HTTP view and no REST endpoint.
- All 24 commands are named `entity_manager/<name>` and carry **both**
  `@websocket_api.require_admin` and `@websocket_api.async_response` (24/24 in
  source). The panel itself is `require_admin=True`. Every command reads or
  writes registry data, so a handler missing either decorator is a security hole.
- `async_setup_ws_api()` is the single registration point. A handler that isn't
  registered there silently does not exist.
- Registry writes happen in `websocket_api.py` (`er.async_get` / `dr.async_get`)
  and in `voice_assistant.py`. Nowhere else.
- The frontend uses **native** HA WS APIs for anything HA already exposes:
  `config/{area,floor,device,entity,label}_registry/list|update`,
  `history/history_during_period`, `homeassistant/expose_entity[/list]`,
  `assist_pipeline/pipeline/list`, and services like `update.install` /
  `button.press`. Custom `entity_manager/*` commands are reserved for what HA
  does not expose cleanly — the grouped entity tree, YAML rewriting, recorder
  queries, HACS scanning, config-entry health. Do not add a command that
  duplicates a native API.

### The 24 commands


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheIcelandicguy/entity-manager](https://github.com/TheIcelandicguy/entity-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
