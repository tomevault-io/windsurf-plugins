---
trigger: always_on
description: This file provides guidance to agents when working with code in this repository.
---

﻿# AGENTS.md

This file provides guidance to agents when working with code in this repository.

## Project Type
Python file synchronization utility for `.kilocode` directories across multiple project folders.

## Run Commands
GUI: `py main_gui.py` or `py app.py`
CLI: `python cli_sync.py <folder1> <folder2> ...`

--- do not remove this ---
## Environment & Shell
- **Context**: Windows 11, VS Code, PowerShell Core (`pwsh`).
- **Base folder**: `d:/Dropbox/Projects/AgentAutoFlow/app/`.
- **Rule**: Always use forward slashes `/` in paths to avoid escaping errors.
- **Terminal**: **STRICTLY PowerShell**. Do not use `cmd.exe`, `bash`, or `wsl`.
- **Syntax Rules**:
  - **Chaining**: Use `;` (sequential) or `&&` (conditional).
  - **Variables**: Use `$env:VAR = 'val'` (not `export` or `set`).
  - **Replacements**: Use `Select-String` (not `grep`), `Get-Content` (not `cat`), `New-Item` (not `touch`), `Remove-Item` (not `rm`).
   - **NO terminal line feeds**: 
      - **No:**:
         ```
         python -c "
         print('stuff')
         print('more')
         "
         ```
      - **Yes:**:
         ```
         python -c "print('stuff'); print('more')"
         ```
- **Prohibited**: `tail`, `sed`, `awk`, `sudo`, and `cmd.exe` flags (like `/d`).

## Tool-Call Path Hygiene (read_file, write_to_file, etc.)

**The `path` field must be a clean string. Nothing else.**

### Required format
- Project-relative, forward slashes only.
- Begins with a single `.` or a directory name — NEVER `..kilocode` (that is a typo for `.kilocode`).
- Skills live at: `.kilocode/skills/<skill-name>/SKILL.md`
- Skill AGENTS lives at: `.kilocode/skills/<skill-name>/AGENTS.md`

### Correct
- `.kilocode/skills/testing/SKILL.md`
- `utils_sync/sync_core.py`
- `tests/test_sync_core.py`

### Wrong — do not emit any of these
- `..kilocode/skills/testing/SKILL.md`        ← missing dot+slash
- `.kilocode/skills/testing/SKILL.md'}]}`     ← JSON closers leaked into string
- `'.kilocode/skills/testing/SKILL.md'`       ← quotes inside the string value
- `D:\Dropbox\...\skills\testing\SKILL.md`    ← absolute + backslashes
- `D:/Dropbox/.../skills/testing/SKILL.md`    ← absolute, even with forward slashes

### Construction rule
When building a `read_file` payload, write the JSON literally — do not template the
path from a Python repr, a `<location>` block, or any rendered list. The `<location>`
field in `<available_skills>` is INFORMATIONAL ONLY; normalize it to the project-
relative form above before placing it into any tool argument.

If you are uncertain whether a path is well-formed, type it character by character
rather than copying.

## Run Commands
- Never use Linux commands in terminal.
- Start app: `py app.py` (not `flask run`). Do not test to see if it worked.
- Activate venv: `.\activate.ps1`
--- do not remove this ---

## Common paths through app

- **Scaffold folder**: `.kilocode`.
- **Scaffold_file_modes**: `.kilocodemodes`.

### GUI: app startup
1) `main_gui.py` (module entrypoint): creates the Tk root window.
2) `MainApp.__init__()`: loads config, creates the event queue, and constructs the sync engine.
3) `MainApp._create_widgets()`: builds the UI.
4) `MainApp._process_events()`: starts periodic polling of the event queue.
5) Tkinter event loop: `root.mainloop()`.

### GUI: "Load favorites"
1) User clicks **Load Favorites** → `MainApp._load_favorite_folders()`.
2) Loads the favorites list from in-memory `MainApp.favorite_folders` (initialized from config during `MainApp.__init__()`).
3) For each favorite folder:
   - `file_path_utils.normalize_path(fav)`
   - `file_path_utils.ensure_roo_dir(normalized)` (creates `{base folder}/{scaffold_folder}/` for new projects)
   - `file_path_utils.has_roo_dir(normalized)` (validates folder has `{scaffold_folder}/`)
   - if valid and not already selected, appends to `MainApp.selected_folders`.
4) If any folders were added, refreshes the folder list via `MainApp._update_folder_list_ui()`.
5) If any favorites were invalid, shows an informational dialog via `messagebox.showinfo("Favorites Skipped", ...)`.

### GUI: "Scan" (plan/preview only)
1) User clicks **Scan** → `MainApp._start_sync()`.
2) `SyncEngine.scan_folders(folders)`
   - emits `EventType.SCAN_START`, then many `EventType.SCAN_FILE` events while walking each `{base folder}/{scaffold_folder}/*`.
   - also scans any `root_allowlist` files at the folder root.
3) `SyncEngine.plan_actions(file_index, scanned_folders=folders)`
   - computes the newest source file per relative path, then creates a list of copy actions for older/missing destinations.
4) GUI stores the planned actions and renders previews via `MainApp._update_overwrite_previews()`.

### GUI: "Execute" (apply planned actions)
1) User clicks **Execute** → `MainApp._confirm_sync()`.
2) `SyncWorker.__init__(sync_engine, folders, actions=planned_actions)` then `SyncWorker.start()`.
3) Background thread runs `SyncWorker.run()`:
   - calls `SyncEngine.execute_actions(actions)`.
   - for each copy action, emits `EventType.COPY` (or `EventType.SKIP` in dry-run) and finally `EventType.COMPLETE`.
4) GUI receives updates in `MainApp._process_events()` and updates folder row statuses.

### CLI: headless sync

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ScotterMonk/AgentAutoFlow](https://github.com/ScotterMonk/AgentAutoFlow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
