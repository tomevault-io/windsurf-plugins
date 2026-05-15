---
trigger: always_on
description: Principles that guide every decision in dux. If a change conflicts with a tenet, the tenet wins.
---

# CLAUDE.md

## Design Tenets

Principles that guide every decision in dux. If a change conflicts with a tenet, the tenet wins.

### Configuration

- **All settings are configurable.** Every single one. If a user can't change it, it shouldn't be hardcoded.
- **The config file is the documentation.** It must clearly explain what each setting does through inline comments. A user should never need to leave the config file to understand an option.

### UI and Navigation

- **New UI must use the theme engine.** Any new screen, pane, dialog, status text, or visual state must derive colors and styles from `Theme`/`theme.rs` rather than hardcoding visual values. If the existing theme surface lacks the needed meaning, add a semantic theme field first and wire every supported theme through it.
- **Tab and Shift-Tab navigate between panes.** This is the primary spatial navigation model.
- **Panes have local key combinations.** A key combo bound in one pane does not necessarily work in another.
- **Panes have interactive and non-interactive modes.** In interactive mode, all key combos (including global ones) are suppressed and input is forwarded to the PTY. In non-interactive mode, both local and global key combos are active.
- **Key combinations are documented in the help page.** The in-app `?` overlay is the authoritative reference. External docs describe how to configure bindings, not enumerate them.
- **Line-scroll keys are gated by context.** In interactive (PTY) mode, arrow keys and Space only scroll when already scrolled back (`scrollback_offset > 0`); otherwise they pass through to the child process. In non-interactive views (diff overlay, non-interactive agent pane, help overlay), there is no competing use for these keys, so they scroll unconditionally. Page-scroll keys (`PgUp`/`PgDn`) always initiate scrolling regardless of context.
- **Space activates the focused button in confirmation dialogs.** Every modal that presents buttons (confirm/cancel, delete/cancel, etc.) must treat Space as equivalent to Enter — it executes whichever button is currently highlighted. This is hardcoded, not a configurable keybinding, because it is a universal accessibility convention. New dialogs with buttons must include this behavior.
- **Clickable controls need misclick-safe spacing.** Separate adjacent click targets enough that an imprecise click cannot accidentally activate a different item. When a checkbox sits above modal buttons, include a blank row or equivalent spacing between the checkbox and the buttons.
- **Animations and periodic refreshes use wall-clock time, not tick counts.** Tying visual updates to tick rate couples frame cadence to logic timing. Use `Instant::elapsed()` (or equivalent) so animations stay smooth and refreshes stay consistent regardless of how often ticks fire.

### Agents and Providers

- **A provider is supported if and only if it supports PTY and oneshot mode.** PTY for interactive sessions; oneshot (headless: send a prompt, get one response) for automated tasks like commit message generation.
- **Any CLI tool can be a provider.** Configure `command` in `config.toml` and dux spawns it. No adapters, no protocol layer. Adding a new provider is a config-only change, not a code change.
- **Claude, Codex, OpenCode, and Gemini CLI are the defaults.**
- **No protocol layer.** No JSON-RPC, no custom message format, no adapter binaries. The CLI runs exactly as it would in a normal terminal.

### Git and Data Safety

- **Worktrees are user data.** Never removed or mutated casually. Deletion requires explicit user confirmation.
- **Git operations are conservative.** Source checkout refresh uses `--ff-only`. Destructive operations require confirmation.
- **Prefer explicit failure over silent waiting.** If something fails, say so immediately with context.

### Tone

- **Welcome tips are playful and sassy.** They should feel fun, not like a manual. Lead with surprise or delight, highlight non-obvious features that differentiate dux, and keep keybinding references secondary to the feature discovery. Avoid dry "press X to do Y" phrasing.

### Quality

- **Prove your work with tests.** Every change should include unit tests. When feasible and low-lift, add integration tests as well.
- When debugging a problem, before fixing it, **aim to prove your findings with a unit test**.

## Project Summary

`dux` is a Rust terminal UI for orchestrating AI coding sessions per git worktree.

The current app provides:

- A left pane for projects and agent sessions
- A center pane showing the agent's terminal output (embedded via PTY) or diff viewing
- A right pane for changed files and diffs
- Commented user config in the platform-specific dux config directory (`~/.dux/` on macOS, `~/.config/dux/` on Linux)
- Session persistence in `sessions.sqlite3` alongside the config
- Logging in `dux.log` alongside the config
- PTY-based agent startup: spawns CLI tools (`claude`, `codex`, `opencode`, `gemini`) directly in a pseudo-terminal

## Important Constraints

- The app spawns CLI tools directly via PTY (portable-pty) and renders their output using an embedded terminal emulator built on alacritty_terminal. There is no protocol layer (no ACP, no JSON-RPC).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [patrickdappollonio/dux](https://github.com/patrickdappollonio/dux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
