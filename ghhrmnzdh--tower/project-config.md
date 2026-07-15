---
trigger: always_on
description: Tower: a control tower for your Claude agents — pin Claude Code to a country
---

# CLAUDE.md

Tower: a control tower for your Claude agents — pin Claude Code to a country
(the fence), isolate network faults (the weather), watch usage, and monitor
every running Claude agent. One daemon, two front-ends (menubar app + terminal
dashboard). Formerly Geo Guard, then Corral.

It is Claude-Code-only, and it is an **honest status layer** — a little helper
so Claude is never confused by a shifting location. If something looks wrong
(off-country, unstable net, a blocked request), that is the app *doing its job*
and reporting the truth, not the app being broken. There is no need to quit it
when you see a fault — quitting only removes the guard and routes Claude back to
a direct connection. Leave it running; it will clear itself when the underlying
condition (your location or your connection) recovers.

## Layout
- `src/towerd.py` — the daemon (proxy + geo + usage + plan + net health +
  agent monitor + file IPC). stdlib only.
- `src/*.swift` — native menu-bar app (main / AppDelegate / Model /
  DesignSystem / Glyph / StatusIcon / Popover / Dashboard / Notifier /
  Components).
- `src/Glyph.swift` — the identity marks: the tower **radar** (guard status,
  five animated states) and the still per-model marks. One geometry backs both
  the live SwiftUI `Canvas` views and the menu-bar `ImageRenderer` templates.
- `src/tower-tui.py` — terminal dashboard (curses, stdlib only).
- `Tower Identity Study.html` — the radar + model marks, live (design reference).
- `build.sh` — compiles the app bundle from `src/` (`-target arm64-apple-macos14.0`).
- `docs/` — ARCHITECTURE.md, DESIGN.md, APP.md, TUI.md.

## Build / run
- `./build.sh` then `open "Tower.app"`.
- TUI: `python3 "Tower.app/Contents/Resources/tower-tui.py"`.

## Invariants — don't break these
- **Never route Claude via the shell.** Routing edits `~/.claude/settings.json`
  `env` only (`HTTPS_PROXY`/`HTTP_PROXY`, plus `CLAUDE_CODE_RETRY_WATCHDOG` /
  `CLAUDE_CODE_MAX_RETRIES` so a blocked/outaged request stays PENDING in
  Claude's native retry spinner instead of erroring). Shell aliases broke the
  `claude` command before. `route_off` removes those keys, but leaves a
  retry value the user customised themselves (`RETRY_ENV`).
- **Fail-closed:** a Claude request is allowed ONLY when *confirmed* inside the
  target country AND the network has a usable path to Anthropic. Anything
  uncertain — location checking/cached/errored, or the net offline/captive/
  edge-unreachable — is blocked. There is no allow-through fallback; the cure
  for false blocks is durable, accurate detection (multi-source geo in
  `geo_loop`), not letting unconfirmed traffic through. `claude -p /usage` is
  itself a Claude request: it is gated on the *same* predicate
  (`State.claude_allowed()`) and never runs off-country or on an unstable net.
  The one intentional pass-through: while routing is ON the gate is fully
  fail-closed, but a double-confirmed route-OFF sets `State.routed = False`, which
  makes `should_block` a pass-through for sessions still pinned to the proxy —
  the same direct, unguarded connection new sessions get. "Off means off," not
  "stuck-gated." `state.routed` may only be flipped by the double-confirmed route
  command or the persisted `cfg["routed"]`; it never fails open on its own.
- **Block with 503, never 403 — the block is PENDING, not FAILED.** A blocked
  Claude request is *held* a few seconds (re-checking so a sub-second blip
  clears with no visible retry), then answered `503 + Retry-After`. Claude Code
  retries 5xx into its native "Retrying · attempt x/y" spinner — the durable
  "pending" UX that resumes on its own when the guard clears — but treats 403 as
  broken auth and kills the turn. The long-outage tolerance comes from the retry
  budget (`RETRY_ENV`), NOT from a long hold: the hold is pre-CONNECT and a long
  one risks tripping the client's tunnel connect timeout. Don't "fix" the block
  by returning 403, and don't stretch `BLOCK_HOLD_S` to cover outages.
- **Usage can't be shown when the guard isn't passing.** When `/usage` is
  gated, front-ends show a *spacious, honest message* (not stale numbers) that
  says whether it's the **connection** (net) or the **location/VPN** (geo) —
  driven by `plan.gate_reason` / `guard.net_ok`, never guessed.
- **No Claude request on an unstable connection.** "Unstable" = no trustworthy
  path to Anthropic (offline / captive portal / edge unreachable), NOT merely
  slow — a "degraded" (high-latency but reachable) link still allows traffic.
  `NetMonitor` publishes `state.net_ok`; it *does* feed `should_block()`.
- **On by default.** Opening the app starts the daemon, and the daemon routes
  Claude on startup unless you've *explicitly* turned routing off
  (`cfg["routed"] == False`). You never have to arm it by hand.
- **The proxy endpoint is durable.** A Claude session captures `HTTPS_PROXY` once
  at launch and can never change it, so the proxy address must not move under it.
  `bind_proxy` prefers the *same* port every run (persisted `cfg["proxy_port"]`,
  retrying briefly for a dying predecessor) so a pinned session survives a daemon
  restart; the app's pollTimer respawns a daemon that dies unexpectedly (kill -9);

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ghhrmnzdh/tower](https://github.com/ghhrmnzdh/tower) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
