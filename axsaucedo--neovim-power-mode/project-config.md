---
trigger: always_on
description: Guidance for AI coding agents (and humans) working on **neovim-power-mode**. Read this before making changes.
---

# AGENTS.md

Guidance for AI coding agents (and humans) working on **neovim-power-mode**. Read this before making changes.

## What this is

A pure-Lua Neovim plugin that renders particle effects, a combo counter, a cacafire-style fire wall and screen shake while you type. Neovim >= 0.9. **Zero external dependencies, no compilation, no build step.**

Because it runs on every keystroke and on a timer, **performance is a feature**. Every change is judged on CPU/battery cost as much as correctness.

## Repo layout

```
lua/power-mode/
  init.lua        Orchestrator: setup/enable/disable/toggle/status, autocmds, on_key backspace hook
  config.lua      Defaults, vim.g mapping, validate(), resolve(), get()
  engine.lua      vim.loop timer render loop; module injection; idle fast-path
  renderer.lua    Pool of 1x1 floating windows for particles (NOT extmarks)
  particles.lua   Preset dispatcher; lazy-loads presets/*; cancel-on-new fade
  fire_wall.lua   2D heat buffer in its own bottom float; heat LUTs; RLE extmark runs
  combo.lua       Combo streak state machine + floating window
  shake.lua       Screen shake: none / scroll / applescript (iTerm2)
  highlights.lua  PowerModeParticle1..8 and PowerModeCombo0..4 groups
  utils.lua       random, clamp, lerp, get_editor_dimensions
  presets/        explosion, fountain, rightburst (default), shockwave, emoji, stars, disintegrate, fire
plugin/power-mode.lua   Load guard, VimEnter auto-setup, all user commands
tests/            Headless unit tests, tmux e2e + smoke, perf suite under tests/perf/
doc/power-mode.txt      Hand-maintained vimdoc help file
```

## Architecture invariants

These are load-bearing. Breaking them causes circular requires, stale windows, or CPU regressions.

- **Dependency injection, not circular requires.** `init.setup()` calls `engine.set_modules(particles, fire, renderer, combo, fire_wall)`. `engine.lua` requires only `config`. Never add a `require("power-mode.renderer")` to `engine.lua`.
- **Two independent render mechanisms.** Particles use a *floating window pool* (`renderer.lua`); the fire wall uses *extmarks* in its own buffer and namespace (`power_mode_fire`, created once at module load). Do not mix them.
- **Extmarks after lines.** In `fire_wall.lua`, `nvim_buf_set_extmark` must run *after* `nvim_buf_set_lines`, or the target rows don't exist yet.
- **Z-order:** combo `zindex=100` > particles `50` > fire wall `40`.
- **Every subsystem exposes a cheap `is_idle()`.** The engine skips all update/merge/render work when all of particles/fire/fire_wall/combo are idle, and then **stops its timer entirely** rather than continuing to wake. Any new subsystem must participate, or idle CPU regresses.
- **The engine cadence follows demand.** It runs at the configured `engine.fps` whenever particles, backspace fire or the fire wall are animating — `fps` is a promise that visible animation is never deliberately slowed. When only the quantized combo timeout bar remains, the timer re-arms at the bar's next visible change via `combo.next_update_delay()`. When everything is idle the timer stops, and `engine.wake()` restores full cadence on the next keystroke. **Because the engine can be fully stopped, anything that makes a subsystem non-idle must go through the plugin's input wiring or call `engine.wake()` itself** — spawning directly into a preset module will not restart the loop. Stale-callback safety comes from `timer_generation` guards; keep them if you touch `start`/`stop`.
- **Any new time-driven combo visual must participate in `next_update_delay()`** or invalidate and wake the engine, or it will not animate while only the combo is on screen.
- **No `table.clear`.** It is LuaJIT-only and missing on some embedded forks. Nil out from the tail (see `clear_array` in `engine.lua`).
- **`vim.loop`, not `vim.uv`** — kept for Neovim 0.9 compatibility.
- **No API calls off the main loop.** The libuv timer callback computes `dt` and immediately `vim.schedule()`s the body.
- **`disable()` must fully restore the editor**: delete the augroup, unregister `vim.on_key`, stop+close every timer, close every window and delete every buffer the plugin created.
- **Works with zero config.** `plugin/power-mode.lua` auto-runs `setup()` on `VimEnter` if the user never called it.
- **Backward compatibility via warning aliases**, never breakage (`:PowerModeCancel`, `fire_wall.mode`, `fire_wall.set_mode`).
- **256-color fallback is required.** Every highlight carries both `fg`/`bg` and `ctermfg`/`ctermbg`.

## Code conventions

- 2-space indent, double-quoted strings, no semicolons.
- Module shape: `local M = {}` ... `return M`; private state as file-locals; test-only accessors prefixed `_` (`M._is_hidden`, `M._scroll_shake`).
- `snake_case` everywhere. Unused params prefixed `_` (`_dt`, `_level`).
- Each file opens with `--- <Module name> for neovim-power-mode` plus a one-line description. `---` for doc comments (`--- @param opts table|nil`), `--` inline.
- **Cite perf work in comments.** Optimisation-motivated code carries a block comment naming the tag and the report it came from (e.g. `-- F8: cache strdisplaywidth ...`). Keep this habit — it's how the perf history stays legible.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [axsaucedo/neovim-power-mode](https://github.com/axsaucedo/neovim-power-mode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
