---
trigger: always_on
description: Single-purpose Lua plugin for Hyprland's embedded Lua API (≥ v0.55.0). The
---

# AGENTS.md

Single-purpose Lua plugin for Hyprland's embedded Lua API (≥ v0.55.0). The
`README.md` is the authoritative user-facing spec — read it once before
changing semantics.

## What this repo is NOT

- **No tests, no build, no CI, no package manager, no rockspec.** Don't run
  `make`, `npm`, `cargo`, `busted`, `luarocks`, etc. — none of it exists or
  applies. Don't add any of it without an explicit request.
- **Not runnable standalone.** The code only executes inside Hyprland's
  embedded Lua state, where the `hl.*` global is injected by the
  compositor. `lua lua/hyprdeck.lua` will crash on the first `hl.on(...)`.

## Verification, such as it is

- **Syntax / type check (closest thing to a test):**
  `lua-language-server --check .` — uses `.luarc.json`, which declares the
  Hyprland-injected `hl` global and points at the API stubs at
  `/usr/share/hypr/stubs/hl.meta.lua`. Grep that stub file when you need
  the real shape of an `hl.*` call (events, dispatchers, getters).
- **Behavioural verification is manual:** reload Hyprland with
  `hyd.setup({ log_level = "trace" })` and watch the Hyprland log. The
  diagnostics module exists specifically to make event ordering visible at
  trace level.

## Layout & entry-point indirection

- `init.lua` (repo root) is a **shim** — it prepends `lua/?.lua` and
  `lua/?/init.lua` to `package.path` and then `dofile`s
  `lua/hyprdeck.lua`. It must not contain logic. Real module code lives
  under `lua/hyprdeck/`.
- The shim uses `dofile` (not `require`) on purpose; `require("hyprdeck")`
  would deadlock against Lua's in-progress sentinel. Don't "simplify" it.
- `lua/hyprdeck/utils.lua` is the **root of the dependency graph** —
  documented as such in its header. It must not `require` any other
  `hyprdeck.*` module; everything else may depend on it.

## Load-time side effects (important)

`require("hyprdeck")` is not pure. At first require it:

1. applies config defaults,
2. sets the log level,
3. subscribes production event handlers (`lua/hyprdeck/events.lua`) — **exactly once per Lua state**,
4. lazily subscribes diagnostic handlers (`lua/hyprdeck/diagnostics.lua`)
   only when log level ≥ `trace`, also **exactly once**.

`setup()` is optional, idempotent, and never re-subscribes production
handlers. If you add new event subscriptions, wire them through
`events.subscribe()` (or `diagnostics.subscribe()` if trace-only) — never
call `hl.on` from a module body.

## Coding conventions worth preserving

- 4-space indent, no tabs. Every file opens with a
  `-- hyprdeck.<module> — short description` header followed by a prose
  paragraph explaining the module's role / invariants.
- Public functions carry LuaCATS annotations (`---@param`, `---@return`,
  `---@class`). `HL.Window`, `HL.Workspace`, etc. come from the stub.
- All log output goes through `hyprdeck.log` (`error` / `warning` / `info`
  / `debug` / `trace`). `log.error` also pops a sticky red Hyprland
  notification, so reserve it for user-actionable misconfiguration.
- Dispatcher constructors in `lua/hyprdeck/dispatchers.lua` must **always
  return a function** (use the module-local `NOOP` on validation failure).
  Returning `nil` crashes `hl.bind`.
- Event handlers touching an `HL.Window` should start with
  `window.is_alive(w)` — reading any field of an expired window emits an
  unavoidable "Tried to access an expired object" warning per access; the
  guard keeps it to one.

## Behaviour gotchas an agent will otherwise re-discover

- `hyd.dsp.focus` is a **full drop-in** replacement for `hl.dsp.focus`.
  `hyd.dsp.window.move` is **not** — it handles only `direction` /
  `workspace` / `monitor` and rejects everything else. Don't expand its
  surface without intent; the README documents the contract.
- In `hyd.dsp.focus({ direction = ... })`, `direction` must be the **only**
  key. The constructor logs an error and returns NOOP otherwise.
- `regroup_workspace` deliberately **skips the merge phase on non-visible
  workspaces** — Hyprland's `into_or_create_group` filters out windows on
  hidden workspaces and would silently no-op. Hidden workspaces self-heal
  via the `workspace.active` handler.
- Workspaces containing a fullscreen window are **frozen**:
  `regroup`/`ensure` early-return via `fullscreen.workspace_has_fullscreen`.
  This is "Strategy D" — see comments in `fullscreen.lua` and the
  `window.fullscreen` / `window.close` / `window.destroy` handlers in
  `events.lua` for the cascade-suppression state machine.
- `group.eject` assumes the user has set
  `group.focus_removed_window = false` in Hyprland (documented as required
  in README). It re-focuses the ejected window explicitly.
- Special workspaces (`ws.special == true`) are intentionally never
  touched by `regroup_all_workspaces`.
- hyprsplit integration is duck-typed against `package.loaded` and cached;
  reset via `hyprsplit.reset_cache()` (already wired into
  `config.reloaded`). There's no required load order between hyprdeck and
  hyprsplit.

## Distribution

Users install by cloning into `~/.config/hypr/hyprdeck` and calling
`require("hyprdeck")` from `hyprland.lua`. No other distribution channel
exists. Keep the install path documented in README in sync with any

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chpock/hyprdeck](https://github.com/chpock/hyprdeck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
