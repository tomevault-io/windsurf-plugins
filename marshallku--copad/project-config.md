---
trigger: always_on
description: Cross-platform custom terminal emulator with shared Rust core and platform-native UIs.
---

# copad

Cross-platform custom terminal emulator with shared Rust core and platform-native UIs.

## Documentation

**Always read `docs/INDEX.md` first** when starting a session. Read only the specific doc files relevant to your current task.

**Always update docs** when making changes:

- New features or modules → update `docs/architecture.md` and relevant doc
- Bug fixes or gotchas → add to `docs/troubleshooting.md`
- Design decisions → add to `docs/decisions.md`
- Completed/new tasks → update `docs/roadmap.md`

## Project Structure

- `copad-core/` — Shared Rust library (config, background, plugin, protocol, theme, error)
- `copad-usage/` — Shared subscription rate-limit readout (Claude OAuth + Codex rollout) used by `coctl usage --limits` and comux's status bar. Kept out of `copad-core` so its `reqwest`+`chrono` deps don't reach the GUIs (decision #74 amendment).
- `copad-linux/` — GTK4 + VTE4 native terminal app (binary: `copad`)
- `copad-cli/` — CLI control tool (binary: `coctl`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marshallku/copad](https://github.com/marshallku/copad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
