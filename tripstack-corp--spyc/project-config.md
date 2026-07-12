---
trigger: always_on
description: A vi-keyboard-driven terminal file manager in Rust, on ratatui/crossterm. In the lineage of keyboard-driven terminal file commanders like `spy`. Single-developer project.
---

# spyc

A vi-keyboard-driven terminal file manager in Rust, on ratatui/crossterm. In the lineage of keyboard-driven terminal file commanders like `spy`. Single-developer project.

> **Canonical agent guide** — the MVU contract, the per-module map, and the
> day-to-day conventions, in one place so every tool reads the same source
> (Claude Code loads it via `CLAUDE.md` → `@AGENTS.md`; codex / agy / others read
> it directly). Deeper design detail lives in [`ARCHITECTURE.md`](ARCHITECTURE.md);
> UI language in [`DESIGN.md`](DESIGN.md); the full feature reference in
> [`FEATURES.md`](FEATURES.md); the config reference (`.spycrc.toml`, notifications,
> keymap DSL, Lua) in [`CONFIGURATION.md`](CONFIGURATION.md); how the agent
> activity-dots / notifications / merge-scope registry fit together in
> [`docs/AGENT_ORCHESTRATION.md`](docs/AGENT_ORCHESTRATION.md). **Keep this file
> slim** — it's always in context.

## What it does

One line per feature; see [`FEATURES.md`](FEATURES.md) for the full reference.

- **Vi navigation** — motions, marks, numeric prefix (`3j`, `5G`), cursor jumps.
- **Chord hints** — hold a chord prefix (`g`, `^a`, `H`, `W`, `y`, `m`, `[`/`]`, …) and after `chord_hint_delay_ms` (default 300) a which-key popup lists the continuations. The discovery surface for the dense keymap.
- **Leader / global menu** — `Space` (list focus) or `^a Space` (from the pane) opens the global/workspace menu: `Space w l|n|d` worktree list/new/delete, `Space p`/`Space P` jump/set PROJECT_HOME, `Space s` session info, `Space ?` help. `Space` is literal text to the child, so the pane path is `^a Space` (the `^a` interception wakes spyc, then `Space` enters the menu — `is_spyc_meta_when_pane_focused` + `PendingSeq::Leader`). `W l|n|d` stays a list-focus alias. (`gh` is gone → `Space p`; `gw` worktree-root jump stays as frame nav.)
- **Embedded pty pane** (horizontal split) with tabs, primarily hosting `claude` for dog-fooding. Divider shows the active tab's live cwd (`↪` when drifted). `^a z` zooms the active region (tmux-style fullscreen toggle).
- **MCP server** on a PID-scoped Unix socket — Claude (`.mcp.json`) and codex (`.codex/config.toml`) both re-exec `spyc --mcp` as a stdio proxy to the one socket. Configs written lazily on agent-pane launch, cleaned up on exit. Queries context + mutates the TUI. Multiple instances coexist (takeover prompted). See ARCHITECTURE.md → "MCP server".
- **`gf`/`gF`** — jump from pane output to the referenced file (or `file:line`); honors scroll mode.
- **`^a u` Quick Select** (wezterm-style) — label URLs / paths / SHAs / IPv4 / custom-regex matches in the visible pane; lowercase yanks, uppercase opens. Custom patterns in `.spycrc.toml` `[[scan.patterns]]`.
- **In-app pager** — search, ANSI, hex-dump, line numbers, `:N` jump, save. Closed views go on a back/forward buffer history (`:bprev`/`:bnext`, `[b`/`]b`, `gp`).
- **Shell prompt** — vi-editable, persistent history (`!` capture, `;` foreground, `$` interactive shell); `!?` history editor.
- **`:` command line** — vim-style (`:limit`, `:!cmd`, `:!!`, `:;cmd`, `:fg`, `:task`, `:grep`, `:bprev`/`:bnext`, `:why-status`, `:why-git` (per-column git-marker refresh dump), `:hooks`, `:notify test`, `:graveyard`, `:activity`, `:agent list`/`:agent registry` (P2 scope-registry pager dumps), `:longlist`, `:filetype`, `:chmod`, `:setenv`, `:q`, …). Less-frequent features ship as `:` commands so they don't each consume a default key; re-bind one via `map KEY command <name>`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Tripstack-Corp/spyc](https://github.com/Tripstack-Corp/spyc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
