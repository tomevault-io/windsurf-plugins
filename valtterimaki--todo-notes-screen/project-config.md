---
trigger: always_on
description: Fetches Google Tasks → renders HTML at native screen resolution → sets as macOS/Windows lock screen wallpaper.
---

# todo-notes-screen

Fetches Google Tasks → renders HTML at native screen resolution → sets as macOS/Windows lock screen wallpaper.

## Pipeline
`main.py` → `core/tasks.py` → `core/renderer.py` → `template/lockscreen.html` → `platforms/macos.py` or `platforms/windows.py`

- `tasks.py` — OAuth + Google Tasks API; `fetch_tasks()` normalises the selected list into `{title, notes, due_label, due_state, subtasks[]}`; `fetch_other_lists()` returns `[{title, tasks:[{title}]}]` for all other lists (incomplete top-level tasks only)
- `renderer.py` — Jinja2 → temp HTML; Playwright (headless Chromium) screenshots at `SCREEN_WIDTH × SCREEN_HEIGHT`; accepts `tasks` and `other_lists`
- `lockscreen.html` — sole visual file; authored at 1920 px wide, CSS `zoom`-scaled to native res
- `config.py` — paths, env vars; `SCREEN_WIDTH=3456`, `SCREEN_HEIGHT=2234` (MacBook Pro 16" native)

## CLI flags (main.py)
| Flag | Purpose |
|---|---|
| *(none)* | Full pipeline: fetch → render → set wallpaper |
| `--no-wallpaper` | Render only, skip wallpaper setter (Swift app calls this, then sets via NSWorkspace) |
| `--fingerprint` | Print SHA-256 of tasks JSON to stdout; used by Swift to skip renders when tasks unchanged |
| `--list-task-lists` | Print `[{id, title}]` JSON; used by Swift to populate list picker |

### Due state → appearance
| `due_state` | Trigger | Card | Badge |
|---|---|---|---|
| `none` | no due date | beige | none |
| `future` | >7 days | beige | ghost pill, outline border |
| `soon` | 0–7 days | beige | yellow `#fff673` pill |
| `overdue` | past due | orange `#ff784f` | black pill, alert icon |

### CSS token system (two-layer — do not break this)
**Base colors** (`--color-*`) — raw palette, never used directly in component rules:
`--color-beige: #d9d1cb`, `--color-black`, `--color-border-alpha: rgba(0,0,0,0.3)`, `--color-yellow: #fff673`, `--color-orange: #ff784f`

**State tokens** — semantic aliases defined in `:root` (default/none/future state), overridden per modifier class:
`--background`, `--text`, `--border`, `--badge-background`, `--badge-border`, `--badge-text`, `--card-border-width`

State classes (`.task-card--soon`, `.task-card--overdue`) override only the tokens that change. Components use only `var(--semantic-token)` — no hardcoded hex in component rules.

### Font
**Roboto Flex** via Google Fonts `@import`. Axes: `opsz, wdth, wght, GRAD, XOPQ, XTRA, YOPQ, YTAS, YTDE, YTFI, YTLC, YTUC`

`font-variation-settings` per element:
- **Heading / task title:** `'GRAD' 80, 'XOPQ' 96, 'XTRA' 468, 'YOPQ' 79, 'YTAS' 750, 'YTDE' -203, 'YTFI' 738, 'YTLC' 514, 'YTUC' 712, 'wdth' 100`
- **Badge / notes text:** same but `'wdth' 114, 'opsz' 45` (wider)
- **Order number circle:** same as badge but `'wdth' 70`

### Other lists panel (right side of screen)
All task lists except the selected one are shown to the right as a flowing text panel:
- `fetch_other_lists()` in `tasks.py` fetches them; passed as `other_lists` to the template
- Layout: `flex-direction: column; flex-wrap: wrap-reverse` — content fills rightmost column first, then wraps leftward
- Container: `position: absolute; left: 986px; right: 120px; top: 362px`; height set inline as `{{ effective_height - 362 - 120 }}px` (120 px bottom safe area)
- Each list: title (bold) + individual `<p>` per task name; truncated to one line with ellipsis
- **Anchor structure**: title + first 3 tasks wrapped in `.other-list__anchor` (`display: block`) — this is a single flex item so the group moves to the next column together if it would leave fewer than 4 items isolated. Tasks 4+ flow freely.
- 32px `<span class="other-list__spacer">` between lists; `column-gap: 32px` between columns; `max-width: 280px` on items

### Figma workflow
When the user shares a Figma URL or asks for a visual change: call `get_design_context` (nodeId + fileKey) and `get_variable_defs` before writing any CSS. The file has a "base colors" collection and a "states" collection — inspect both to map token values before touching the template.

## Swift menu bar app (macOS only)
Source: `app/Sources/TodoNotesScreen/`. Built app: `TodoNotesScreen.app` (install to `/Applications/`).

**Refresh loop (every 2 minutes):**
1. `main.py --fingerprint` → compare SHA-256; if unchanged, skip
2. `main.py --no-wallpaper` → render PNG
3. `WallpaperManager.set()` — calls `desktoppr` if installed (sets both desktop + lock screen via private WallpaperKit APIs); falls back to `NSWorkspace.setDesktopImageURL` (desktop only) if not
4. `AppState` also re-applies the wallpaper on `screensDidWakeNotification` to prevent macOS from reverting on display wake

**Menu bar features:** status line, Refresh Now, Pause/Resume, task list picker, Launch at Login (`SMAppService`), Quit

**Hard-coded paths in `AppState.swift`** (update if project moves):
```
~/Documents/todo-notes-screen/venv/bin/python3
~/Documents/todo-notes-screen/main.py
```

**Build:**
```bash
./build.sh          # build
./watch-swift.sh    # auto-rebuild on source changes
```

## Scheduling
| Mechanism | Interval | Status |
|---|---|---|
| Swift menu bar app | 2 min (fingerprint-gated) | Active |
| `scheduling/macos_launchd.plist` | 30 min | **Not installed** (optional fallback) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/valtterimaki) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-13 -->
