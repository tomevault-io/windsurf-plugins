---
trigger: always_on
description: A configurable status bar for Claude Code. Pure bash core with optional Node.js helper for live activity. Cross-platform (macOS, Linux, Windows/MSYS2).
---

# Claude Code Status Bar — Project Guide

## What this is

A configurable status bar for Claude Code. Pure bash core with optional Node.js helper for live activity. Cross-platform (macOS, Linux, Windows/MSYS2).

## Architecture

```
VERSION                    # Single source of truth for version (bump ONLY this file for releases)
statusline-command.sh      # The runtime script — installed to ~/.claude/
statusline-helper.js       # Optional Node.js transcript parser — installed to ~/.claude/
statusline-subagent.js     # Optional Node.js subagent panel renderer — installed to ~/.claude/
install.sh                 # Installer/updater — downloads script + helpers + VERSION from GitHub
install.ps1                # Windows PowerShell installer/updater — native JSON settings merge
.claude-plugin/plugin.json # Plugin manifest for marketplace distribution
.claude-plugin/marketplace.json # Marketplace catalog (required by /plugin marketplace add)
commands/setup.md          # Slash command: /claude-code-status-bar:setup
commands/configure.md      # Slash command: /claude-code-status-bar:configure
docs/assets/               # README images + their generators (hero, animated hero-demo.svg/.gif, banner pair, social card)
docs/assets/themes/        # per-theme --demo preview SVGs + generate-theme-demos.sh (captures real --demo output) + ansi-to-svg.js
README.md                  # User-facing docs
```

### Installed files (at ~/.claude/)

| File | Purpose | Overwritten on update? |
|------|---------|----------------------|
| `statusline-command.sh` | The script Claude Code runs | Yes |
| `statusline-helper.js` | Node.js transcript parser (optional) | Yes |
| `statusline-subagent.js` | Node.js subagent panel renderer (optional) | Yes |
| `.statusline-version` | Local copy of VERSION | Yes |
| `statusline.conf` | User config overrides | **Never** (created from the template on first install only) |
| `statusline.conf.example` | Commented reference template (all options, all commented) | Yes |
| `.statusline-update-cache` | Update check cache (timestamp + version) | Cleared on update |
| `.statusline-usage-cache` | Usage API response cache (JSON) | Auto-refreshes every 10 min |
| `.statusline-activity-cache` | Transcript activity cache (JSON) | Auto-refreshes every call |
| `.statusline-claude-status-cache` | Claude API status cache (`<epoch> <indicator>`) | Auto-refreshes every `claude_status_cache_seconds` (opt-in) |
| `.statusline-transcript-cache/` | Parsed transcript cache (by SHA256) | Auto-invalidates on change |

## Key design decisions

- **No jq** — Uses bash regex (`BASH_REMATCH`) for JSON parsing. Windows/MSYS2 users don't have jq. Because the regex captures the raw bytes between the quotes, `extract_from` then runs `json_unescape` on the captured value to decode JSON string escapes (`\\`→`\`, `\/`, `\"`, `\n \t \r \b \f`) — without it a Windows `cwd` (JSON-escaped as `C:\\laragon\\www`) renders with doubled backslashes (the v2.23.2 fix). `json_unescape` is single-pass, fork-free, and fast-paths to a no-op when the value has no backslash (so macOS/Linux paths and every non-path field cost nothing); the left-to-right scan handles adjacent escapes and UNC `\\\\server` prefixes that a naive global replace would mangle.
- **Bash 3.2 minimum** — Must work on stock macOS. No associative arrays, no `readarray`, no `${var,,}`.
- **Background update check** — Fetches VERSION from GitHub every 6h in a background subshell. Never blocks the status bar. The `↑ <version>` notice is an OSC 8 link to that release's notes (reuses the PR segment's hyperlink form; gated on `pr_link`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [briansmith80/claude-code-status-bar](https://github.com/briansmith80/claude-code-status-bar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
