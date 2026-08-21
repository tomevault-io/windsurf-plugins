---
trigger: always_on
description: **Claudesk** — a macOS-only, single-user, open-source "lite IDE" that puts the daily Claude Code + Sublime Text workflow in **one window with multiple virtual workspaces inside it**. The pain point: starting work on any given project takes minutes of repetitive setup (open terminal → cd → `claude`, open Sublime Text and load project, open Sublime Merge and load again, occasionally a second terminal and cd again). Over 20+ rotating projects with 3–4 in flight on any given day, this cost compounds
---

# Claudesk

## Project Overview

**Claudesk** — a macOS-only, single-user, open-source "lite IDE" that puts the daily Claude Code + Sublime Text workflow in **one window with multiple virtual workspaces inside it**. The pain point: starting work on any given project takes minutes of repetitive setup (open terminal → cd → `claude`, open Sublime Text and load project, open Sublime Merge and load again, occasionally a second terminal and cd again). Over 20+ rotating projects with 3–4 in flight on any given day, this cost compounds. Compounding it: when several projects ARE in flight, finding the one waiting on input means clicking through windows or switching Spaces — a second-order tax on top of the launch tax.

Claudesk provides:
- **VSCode-style project picker** — click a project → full environment fires up in <10s. Each pick opens a new **workspace** inside the existing Claudesk window (a new tab/stage), not a new OS window.
- **One workspace = one project = one CC session.** Single window holds N workspaces concurrently.
- **Mission Control-inspired layout.** Center stage = the focused workspace, full-size; top filmstrip = live thumbnails (or status tiles, pending the Phase 1 thumbnail-rendering probe) of every other open workspace, ordered, with project name + idle/running/awaiting-input dot. Clicking a filmstrip tile promotes that workspace to center stage and demotes the previous one. Filmstrip is collapsible to a row of mini status tiles (project name + status dot only) for reclaiming vertical space.
- **Left half of each workspace:** Claude Code in a true PTY-backed terminal, yolo mode by default, already `cd`'d into the project. Rendered with xterm.js DOM renderer (no WebGL).
- **Right half of each workspace:** a placeholder in Phase 1; a built-in lite editor + git diff viewer arrives in Phase 3.
- **Stateful CC controller (Phase 2):** Claudesk owns each workspace's CC process lifecycle, watches workflow state files, and exposes workflow operations (skill buttons, Recycle Session) as clicks rather than typed slash commands.
- **Menu-bar status item (Phase 2):** an aggregate idle/running/awaiting-input dot in the macOS menu bar — click to open a popover listing every workspace + status; clicking a row brings Claudesk forward and switches the center stage. Always visible system-wide, even when the Claudesk window is hidden, minimized, or on a different Space.
- **Picture-in-picture mini player (Phase 2, conditional):** a small always-on-top floating panel (via `tauri-nspanel`) the user can summon when the Claudesk window is out of focus. Mirrors the same status surface as the filmstrip. Display-only in v1 — clicking a tile does NOT bring the workspace forward. Conditional on Phase 2 dogfooding: if the menu-bar item alone suffices, PiP may defer to Phase 4.
- **Smart auto-resume on workspace open (M12 — ✅ SHIPPED 2026-08-05):** opening a project fires the right resumption command by itself and **announces it before you click**. ⚠️ **TWO** signals, not three: unclean-exit flag → spawn with the `--continue` CLI flag; else `.session.md` present → inject `/session-restore`; else **nothing** (`/session-start` is *never* auto-fired — it gets an explicit button). ⚠️ **The unclean flag BEATS `.session.md`.** ⚠️ **`/session-resume` and `/session-pause` DO NOT EXIST** — renamed `/session-restore` / `/session-handoff` at M9 WP5. The refuted three-branch design + precedence proof: `arch/session-resumption.md` → "The two signals".
- **Drive-mode selector on the PICKER ROW (M12; ⚠️ NOT the workspace header):** a compact readout, click to edit, showing the project's drive mode (1 `stepping` / 2 `orchestrated` / 3 `autopilot` / 4 `fsd` — ⚠️ **not** `step-by-step`/`full-autopilot`, which no workflow skill recognizes; authority is upstream `transitions.md`). ⚠️ **As built it IS a native `<select>`** — reversing the "never a live `<select>` on every row" rule, because the four values are a **closed** set and a bad mode string fails serde on read and takes the whole project list down. The model override's open-string rule must NOT be generalized here. See `arch/session-resumption.md` → "The picker-row cell".


- **Sublime launchers (both KEPT permanently — revised 2026-06-20, WP8):** Sublime Text and Sublime Merge are each one click away via icon buttons in the right-panel tab row. ⚠️ The Sublime *Text* pop is **NOT removed** — the in-app editor is the *primary* editing surface, but Sublime Text stays as a permanent escape hatch. See "Key Decisions" below.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [StaymanHou/Claudesk](https://github.com/StaymanHou/Claudesk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
