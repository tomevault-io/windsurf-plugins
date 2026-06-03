---
trigger: always_on
description: Read this **before** exploring. Saves ~3 Explore-agent roundtrips per session.
---

# CLAUDE.md — Orienting notes for kittyhack

Read this **before** exploring. Saves ~3 Explore-agent roundtrips per session.

## What this project is

Shiny-for-Python web app that replaces the defunct Kittyflap cloud/app. Controls a smart cat flap with RFID, camera-based cat+prey detection, per-cat rules, MQTT, optional remote inference. Debian/Raspberry Pi OS.

**Repos (user has maintainer rights on both):**
- Upstream: `floppyFK/kittyhack`
- Fork:     `FabulousGee/kittyhack` (local `origin` → here, `upstream` → upstream repo)
- Feature PRs go fork → upstream, user merges. Release tags live on upstream.

User-facing docs: `README.md` (EN + DE). This file is for the assistant.

## Runtime topology — target vs remote mode

Two deployment shapes, gated by `is_remote_mode()` (`src/mode.py`, reads `.remote-mode` marker or env):

| Mode | Where UI+AI run | What runs on the Kittyflap |
|------|------------------|----------------------------|
| **target** (default) | on the Kittyflap itself | `kittyhack.service` (Shiny UI) + `kittyhack_control.service` (supervisor, watchdog) |
| **remote**           | on a separate Linux PC/VM | `kittyhack_control.service` only (handles sensors, relays camera stream to the remote) |

`kittyhack_control.py` **refuses to start in remote mode** (hard guard at `main()`). So features like the WLAN watchdog live there and only exist on real Kittyflap hardware.

Setup files: `setup/kittyhack.service`, `setup/kittyhack_control.service`, `setup/kittyhack-setup.sh`.

## Where things live

| Concern | File | Notes |
|---------|------|-------|
| ASGI entry + middleware chain | `app.py` | Order: `ApiMiddleware → TabRoutingMiddleware → shiny_app` |
| Config defaults / load / save | `src/baseconfig.py` | `CONFIG` global dict, `DEFAULT_CONFIG`, `load_config()`, `save_config()` |
| Shiny UI + reactive handlers | `src/server.py` | ~9800 lines. `ui_system()`, `ui_configuration()`, `ui_info()`, `ui_live_view`, etc. |
| Backend loop (runs in kittyhack.service) | `src/backend.py` | `backend_main()`, `manual_door_override` dict, MQTT publish/receive hooks |
| Target-side supervisor | `src/kittyhack_control.py` | `_wlan_watchdog_loop`, relay camera, boot-wait logic |
| Subprocess wrappers (nmcli, systemctl, git, pip, …) | `src/system.py` | `update_kittyhack()`, `switch_wlan_connection()`, `apply_wlan_runtime_settings()`, `systemctl()` |
| Version check / release notes / update-repo resolution | `src/helper.py` | `read_latest_kittyhack_version()`, `fetch_github_release_notes()`, `resolved_update_repo()` |
| Door hardware | `src/magnets_rfid.py` | `Magnets` singleton (`Magnets.instance`), `queue_command("unlock_inside" | "lock_inside" | "unlock_outside" | "lock_outside")`, state getters |
| DB access | `src/database.py` | `db_get_cats`, `db_get_motion_blocks`, `get_cat_settings_map`, backup helpers |
| MQTT | `src/mqtt.py` | `MqttPublisher`, topics, `handle_manual_override()` bridge in backend |
| REST API (PR #153, not yet merged to main) | `src/api.py` | Token storage, auth, Starlette routes, middleware |
| Translations | `locales/{de,en}/LC_MESSAGES/messages.{po,mo}` | gettext, compile with `msgfmt` |
| Per-release changelogs | `doc/changelogs/changelog_vX.Y.Z_{en,de}.md` | One file per version per language |
| Static assets (JS toggles, CSS) | `www/` | `server-ui.js` handles conditional-visibility toggles |

## Config pattern (adding a new setting)

Three places, exactly in this order, or load/save drifts silently:

1. `DEFAULT_CONFIG["Settings"][…]` in `baseconfig.py` (~line 185+): lowercase ini-key → default value.
2. `new_config = { … "UPPER_KEY": safe_str/int/bool/float/enum(…) }` in `load_config()` (~line 442+).
3. `settings["lowercase_key"] = CONFIG["UPPER_KEY"]` in `save_config()` (~line 611+).

For UI: add input to `ui_configuration()` in `server.py` (~line 6441+), then assign from `input.<id>()` in `on_save_kittyhack_config()` (~line 7645+).

Conditional visibility pattern: give the container an `id=` and toggle via a new init function in `www/server-ui.js` (see `initIpCameraUrlToggle`, `initUpdateRepoToggle`).

## Door control — do NOT bypass

Canonical: set a flag in `backend.manual_door_override = { unlock_inside, unlock_outside, lock_inside, lock_outside }`. The backend loop in `backend.py` picks it up, runs safety checks, and calls `Magnets.instance.queue_command(…)` with correct state tracking (`inside_manually_unlocked`, event annotations, MQTT mirror). MQTT and the REST API both use this path.

Directly calling `Magnets.instance.queue_command("…")` **skips** that state tracking. Only do it for teardown/shutdown.

Read state with `Magnets.instance.get_inside_state()` / `get_outside_state()` (returns `True` when unlocked).

## Shiny patterns used throughout

- `@output @render.ui def ui_xxx()` → builds a DOM fragment returned to `output_ui("ui_xxx")` slots.
- `@reactive.Effect` (or `@reactive.effect`) + `@reactive.event(input.btn_id)` → button-click handler.
- `reactive.Value(0)` as a trigger → bump with `reload_trigger_X.set(reload_trigger_X.get() + 1)` to force a `@render.ui` that reads `.get()` to re-run. Global triggers sit at `server.py:838+`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [floppyFK/kittyhack](https://github.com/floppyFK/kittyhack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
