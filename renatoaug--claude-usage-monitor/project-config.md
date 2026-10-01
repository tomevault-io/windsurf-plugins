---
trigger: always_on
description: A cute pixel-art desktop pet (Electron) that tracks Claude Code usage: real session/weekly % (via OAuth login) plus token counts from local logs. macOS-first, with Windows (x64) support and Linux on the way.
---

# Clauddy — project guide

A cute pixel-art desktop pet (Electron) that tracks Claude Code usage: real session/weekly % (via OAuth login) plus token counts from local logs. macOS-first, with Windows (x64) support and Linux on the way.

## Stack

- **Electron** (frameless, transparent, always-on-top widget)
- **Bun** for install/scripts, **Node 24** (pinned in `.nvmrc`)
- **Biome** for format + lint
- Vanilla JS in `renderer/` (SVG sprite + CSS/WAAPI animations) — no framework

## Layout

- `main.js` — Electron main process (window, usage polling, notifications, file watchers)
- `preload.js` — `contextBridge` API exposed to the renderer
- `usage.js` — reads `~/.claude/projects/**/*.jsonl` (tokens, session window, activity)
- `auth.js` — OAuth (PKCE) login + authoritative usage % fetch
- `accounts.js` — the account list (several subscriptions, one active at a time)
- `codex.js` — reads Codex's `~/.codex/sessions/**/rollout-*.jsonl` (limits %, tokens, 30-day history). Opt-in from Settings (`config.codex.enabled`); with more than one service connected, tabs under the pet pick which one the whole panel shows
- `cursor.js` — Cursor: usage % from Cursor's (unofficial) dashboard API with the token the Cursor app keeps in its `state.vscdb` (read via `node:sqlite`, `bun:sqlite` in tests), activity from the agent transcripts under `~/.cursor/projects`. Monthly billing cycle, no tokens. Opt-in from Settings (`config.cursor.enabled`)
- `reminders.js` — persistent, one-shot session reset reminders, keyed by provider and Claude account
- `renderer/` — `index.html`, `pet.js`, `style.css` (the pet + UI)
- `renderer/companion.js` — provider activity transitions, with a shared cooldown
- `renderer/voice.js` — the pet's voice: remark lines for the speech bubble and the chiptune blips (Web Audio square waves, no assets). Talk is on by default, sound is opt-in (`config.sound`) and silent in menu-bar mode, collapsed, or muted
- `make-icon.js` — generates the macOS `.icns` from the pixel sprite
- `make-ico.js` — packs the Windows `.ico` (`build-icon.sh` drives both + the Linux `.png`)
- `make-tray.js` — generates the menu-bar (tray) template icon from the same sprite
- `scripts/adhoc-sign.js` — `afterPack` hook: ad-hoc signs the macOS bundle with the real appId, without which macOS silently drops the app's notifications
- `pet` — dev script to simulate pet states (writes `~/.claude-usage-monitor/debug.json`)

## Commands

```bash
bun start          # dev run
bun run pack       # build dist/mac-arm64/Clauddy.app
bun run dist       # macOS zip (used by the release pipeline)
bun run dist:win   # Windows zip — run on Windows/CI (needs a native runner)
bun run dist:linux # Linux tar.gz + AppImage (run on Linux/CI)
bun run icon       # regenerate build/icon.{icns,ico,png} from the sprite
bun run check      # Biome format + lint (autofix)
bun test           # the suite — but prefer `bun run test` (see below)
bun run test       # all three groups, each in its own process
bun run test:coverage
./pet <state>      # simulate a state, e.g. ./pet fire
```

`./pet <state>` states: `fire`, `sleeping`, `working`, `tired`, `idle`, `poke`, `celebrate`, `say`, `auto`, plus the activity scenes `reading`, `editing`, `running`, `planning`, `researching`, `delegating`, `waiting`.

> Windows/Linux artifacts must be built on their own OS (or CI runner) — electron-builder can't reliably cross-build them from macOS. `release.yml` handles that in three stages: `version` (semantic-release dry-run) → `build` (matrix on macOS/Windows/Linux, each stamping the computed version via `scripts/set-version.js`) → `publish` (downloads every artifact and runs semantic-release for real).

## Tests

`bun run test` — three processes, not plain `bun test`:

- `test/unit/` — `usage.js`, `auth.js`, `codex.js`, `cursor.js`, `renderer/burn.js`, `renderer/voice.js`, `renderer/companion.js` (no mocks)
- `test/main/` — `main.js`, with `reminders.js` through it (mocks `electron`, `./usage`, `./auth`, `./codex`, `./cursor`)
- `test/dom/` — `renderer/pet.js`, `preload.js` (happy-dom)

Split because bun mocks are per-runtime and it loads every test file before running any, so a mock in one file reaches the others. Each group must be run by its own path — `bun test` (or `bun test test/`) globs all three into one process and fails.

`bun run test:coverage` gates: 80% total, 60% per file, and no shipped `.js` without coverage. Runs on every PR.

`pet.js`, `burn.js`, `voice.js` and `companion.js` end with a `module.exports` guard so one file works as a `<script>` in the widget and as an import in the tests.

> `main.js`'s update path spawns `curl … | bash` — always stub `child_process.spawn`, or it installs over the running app.

## Data & secrets

- All user data lives in `~/.claude-usage-monitor/` (NOT in the repo): `auth.json` (OAuth token, mode 600), `config.json` (settings), `debug.json` (simulator), `alerts.json` (which notifications are already armed, so restarts don't repeat them), `reset-reminders.json` (one-shot deadlines and recently delivered reminders), `accounts.json` (the account list + the active one).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [renatoaug/claude-usage-monitor](https://github.com/renatoaug/claude-usage-monitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
