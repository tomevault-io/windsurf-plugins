---
trigger: always_on
description: FramePort ports Meta Quest standalone APKs to the **Valve Steam Frame** (SteamOS, aarch64). Games run in Valve's **Lepton**
---

# FramePort — notes for Claude

FramePort ports Meta Quest standalone APKs to the **Valve Steam Frame** (SteamOS, aarch64). Games run in Valve's **Lepton**
(Waydroid-based Android container), one container per game, launched from a Steam library shortcut. Pipeline:
`scan → analyze → suggest recipe (catalog/heuristics) → user confirms → overport → Frame fixes → sign → static checks →
install over SSH (agent) → Steam shortcut → headless launch test + log triage`.

Read `docs/PLAYBOOK.md` (symptom → fix) before debugging a game, and `docs/FRAME_RUNTIME.md` for runtime facts.

## Layout
- `src/frameport/` — Python package. `pipeline.py` is the API the CLI (`cli.py`) and GUI (`ui/`, Flet 1.0) share.
  - `ui/` — `app.py` shell (sidebar with Frame connection + activity cards, routing, actions, 30 s connection poll),
    `theme.py` (dark design tokens; change colours/spacing only there), `components.py` (pill, card, callout,
    status_row, art_fill, confirm, `update()` = safe update: in Flet 1.0 reading `.page` of an unmounted control
    raises), `jobs.py` (background FIFO job queue, one at a time, cancel via Reporter; no Flet), `views/`
    (library: search/filters/tags/sort as pure tested helpers; game: hero + one-click install, patches under
    "Customize"; frame: device + readiness + installed, or connect wizard; settings; welcome; activity panel).
    User tags live in library entries (`tags`), filters in library setting `ui.library`.
    **Performance rules** (the app froze before): never put image bytes in controls — artwork is served by URL from the
    GUI assets dir (= user data dir; `ft.run(assets_dir=…)`), as thumbnails (`artwork/thumbs.py`, Pillow); the
    library view is persistent, streams cards in batches from a background thread and filters by toggling visibility;
    job/connection events call `app.refresh_view()` (targeted), not `render()`; no I/O in render paths.
    **Never recreate clickable controls on progress ticks** (sidebar, activity tiles): update their properties —
    replacing them 5×/s swallowed clicks (couldn't leave the Library during an upload).
    Help hints: wording for non-obvious terms lives in `ui/help.py` (`HELP`); show it with `C.help_icon(key)` or the
    `help=` argument of `section`/`status_row`/`kv`, tooltips via `C.tip()` (wraps). Game actions for the Library
    right-click menu (one `ft.ContextMenu` around the grid, filled on right-click) and the game page's "…" menu come
    from `app.game_actions()`. Picking art (`sources.apply_choice`) downloads into a staging dir and keeps the old
    art if nothing came back; "Update Steam art on Frame" re-sends the art set to the game's anchor (agent ≥ 12).
    Patch descriptions/reasons describe the general case, naming games only as "e.g. …".
  - Installs: queued/cancelled/failed ones are remembered (library setting `ui.installs`) → Library "Resume" bar;
    uploads are interruptible (Cancel checked per MiB) and resumable (big files via SFTP `.part` append, small files
    streamed in tar batches; the agent counts files already in `incoming/`). Multi-select in the Library queues
    installs after asking every needed question up front. Failures end in one pop-up (Resume / Uninstall / log).
  - **PC VR repacks are pre-patched to run directly** (proven: Rick and Morty, Vader Immortal run when the exe is
    launched directly; Revive breaks them). So Rift games default to `as_is` = install the copy unchanged and launch
    the exe directly (`pcvr.xr_timefix` for the Frame OpenXR-1.1→1.0 fix, `pcvr.no_crash_reporter` for Unreal).
    **Revive is off by default, opt-in** (`pcvr.revive`, and `pcvr.oculus_unreal`): only for an un-cracked Oculus game
    that fails at "Initializing OVR session". Those (Lone Echo, Robo Recall, Lies Beneath: crack .7z not extracted /
    Platform SDK) hit Revive's Oculus-runtime **signature check** under Proton-arm64 — Revive's LoadLibrary/WinVerifyTrust
    hooks don't install (ARM64EC), and the game's Oculus SDK shim rejects the unsigned Revive runtime (wintrust +
    crypt32 signer "Oculus VR") — so they don't run on the Frame without extracting the repack's crack (which FramePort
    doesn't do). `VD.bat` is Virtual Desktop's launcher: ignore it except as an exe-location hint. Library migration
    `rift_run_direct` resets existing recipes.
    Auto launch-test is skipped on PC installs (it would start the game on the user's desktop). Launch tests collect the Unreal
    game log + crash summaries from the Proton prefix; triage `unreal-crash`. Lies Beneath via Proton without Revive
    crashed (UE 4.23 "Unhandled exception").
  - Rift scanning: `sources/rift_dump.scan` = the scanned folder's subfolders are games (one per folder; a folder is a
    game if all candidate exes sit under one child), recursing into collections; `analysis/rift.py` walks once,
    filters helpers + non-GUI PEs, `rank_exes` (Unreal *-Shipping beats its launcher, Steam builds −10, ambiguous →
    `exe_confirmed=False` → GUI exe dialog), `clean_title`, fingerprint (unchanged folders aren't re-analyzed),
    modular-Unreal Oculus plugin DLLs + UTF-16 markers count as LibOVR, `revive_bundled` → as-is.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [austinv11/frameport](https://github.com/austinv11/frameport) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
