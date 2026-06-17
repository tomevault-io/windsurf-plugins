---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This repo is a collection of documentation files and scripts that fix Claude Code papercuts on WSL2 + Windows Terminal. There is no build system, test suite, or package manager — the "deliverables" are markdown docs (explaining problems and fixes), shell scripts, and Claude Code config files (agents, skills).

## Repository Structure

- **`*.md` at root** — Each file documents one fix: the problem, root cause, exact config or script to install, and troubleshooting steps. These are the primary artifacts.
- **`agents/`** — Custom Claude Code subagent definitions (YAML frontmatter + instructions). Installed to `~/.claude/agents/`.
- **`skills/`** — Custom Claude Code slash-command skills. Installed to `~/.claude/skills/`.
- **`codex-skills/`** — Codex-native skills adapted from the Claude skill set. Installed to `~/.codex/skills/`.

## The Fixes

| File | What it configures |
|------|-------------------|
| `image-paste.md` | `~/.local/bin/wsl-screenshot-cli` (Go daemon polling Windows clipboard) + `~/.claude/keybindings.json` (Alt+V) + `SessionStart` hook only (no SessionEnd) |
| `shift-enter.md` | VSCode `/terminal-setup` + Windows Terminal `settings.json` action (`\u001b\r`) |
| `claude-notify.md` | `~/bin/claude-notify` (bash → PowerShell balloon tip) + Claude `Stop` and `PermissionRequest` hooks — **WSL2 only** — skips if Windows Terminal is foreground |
| `codex-notify.md` | Reuses `~/bin/claude-notify` via Codex top-level `notify` key in `~/.codex/config.toml`; `jq` pulls `last-assistant-message` into the balloon — **WSL2 only** |
| `claude-notify-powershell.md` | `%USERPROFILE%\.claude\claude-hook-toast.ps1` + `PermissionRequest` hook only — **native Windows PowerShell only** |
| `statusline.md` | `~/.claude/statusline-command.sh` + `statusLine` in `~/.claude/settings.json` |
| `settings.md` | `~/.claude/settings.json` `attribution` field + `~/.claude.json` `hasTrustDialogAccepted` |
| `browser.md` | `BROWSER` env var in `~/.zshrc` pointing to Windows `.exe` |
| `mcp-setup.md` | DeepWiki (HTTP, user-scoped), Playwright (npx), Figma Desktop (localhost:3845) |
| `playwright-cli.md` | `@playwright/cli` global install + `install --skills`; CLI alternative to Playwright MCP, token-efficient, preferred for coding agents |
| `lsp-setup.md` | LSP binaries: typescript-language-server, pyright, gopls (Go 1.26+), rust-analyzer; PATH in `~/.zshrc`; install official LSP plugins; `enabledPlugins` in `settings.json`; optional `ENABLE_LSP_TOOL` workaround |
| `voice.md` | `pulseaudio-utils` + `libasound2-plugins`; `~/.asoundrc` routing ALSA default PCM to `pulse` plugin at WSLg socket; `PULSE_SERVER` in `~/.zshrc` |
| `capslock-esc.md` | SharpKeys registry remap: CapsLock → Escape, system-wide, Windows-side only — no WSL config needed |

## Key Technical Details

**wsl-screenshot-cli architecture**: `image-paste.md` uses [wsl-screenshot-cli](https://github.com/Nailuu/wsl-screenshot-cli). A Go daemon in WSL keeps a persistent `powershell.exe -STA` subprocess alive to access the Windows clipboard through .NET (`System.Windows.Forms.Clipboard`), side-stepping WSLg/Wayland clipboard limitations.

**wsl-screenshot-cli polling**: The daemon polls the Windows clipboard every 250 ms by default. When it detects a new screenshot, it receives PNG bytes from PowerShell, deduplicates by SHA256, and stores the file under `/tmp/.wsl-screenshot-cli/<hash>.png`.

**wsl-screenshot-cli clipboard formats**: After saving the screenshot, the daemon updates the Windows clipboard with three formats at once: `CF_UNICODETEXT` for WSL terminal paste (the WSL path string), `CF_BITMAP` for image apps like Paint, and `CF_HDROP` for paste-as-file in Explorer and file dialogs. The same screenshot therefore pastes as a path in Claude Code / Codex, but still behaves like an image or file in Windows apps.

**wsl-screenshot-cli SessionEnd pitfall**: Keep the repo docs aligned with `image-paste.md`: do not add a `SessionEnd` hook in Claude Code. Claude Code fires `SessionStart`/`SessionEnd` for every Task-tool subagent, so a subagent `SessionEnd` would stop the daemon mid-session for the main agent.

**claude-notify async (WSL2)**: For Claude Code, wrap the `Stop` hook command as `bash -c '... &'` because the PowerShell script stays alive while the balloon is visible. The Codex variant lives in `codex-notify.md` (reuses the same script via the top-level `notify` key) — keep the two docs cross-linked. The Codex trap: `[tui].notifications` is in-terminal only and runs no external program; the balloon needs the separate top-level `notify` key, which passes the `agent-turn-complete` JSON as the final arg (`$1`; `"--"` is `$0`) so `jq` can pull `last-assistant-message`. Requires `jq`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [congmnguyen/claude-code-wsl2-setup](https://github.com/congmnguyen/claude-code-wsl2-setup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
