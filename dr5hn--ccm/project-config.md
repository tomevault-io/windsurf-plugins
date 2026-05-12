---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CCM (Claude Code Manager) is a Bash CLI toolkit for managing multiple Claude Code accounts, sessions, environments, and health. Single-file architecture (`ccm.sh`, ~6600 lines) with a static landing page (`index.html`), a statusline visual guide (`statusline.html`), and a standalone statusline installer (`statusline.sh`).

## Commands

```bash
# Release (bumps version in ccm.sh + CHANGELOG.md, commits, pushes, creates GitHub release)
./release.sh patch|minor|major|X.Y.Z [--dry-run]

# Test locally after changes
bash ccm.sh version
bash ccm.sh doctor
bash ccm.sh help
bash ccm.sh permissions audit
bash ccm.sh clean tmp --days 365   # should find nothing
bash ccm.sh usage history --days 1
bash ccm.sh profiles list
bash ccm.sh watch status
bash ccm.sh recover
bash ccm.sh session archives

# Landing page — open index.html directly in browser, no build step
# Statusline guide — open statusline.html directly in browser
```

There is no test suite, linter, or build system. Validate changes by running commands manually.

## Architecture

### ccm.sh — Single-file modular Bash script

The script follows a strict top-to-bottom section layout:

1. **Constants & Utilities** (lines 1–550) — `CCM_VERSION`, color init, platform detection (`detect_platform()` → macos/wsl/linux), JSON helpers, validation functions, `write_json()` (atomic: temp file → validate → mv)
2. **Credential Management** (lines 261–370) — macOS uses Keychain, Linux/WSL uses file-based storage with atomic writes (temp + mv). `read_credentials()`/`write_credentials()` are platform-dispatched
3. **Sequence & Cache** (lines 370–550) — `sequence.json` is the account registry (schema v3.1, auto-migrates from v1/v2/v3). `resolve_account_identifier()` matches by number, email, or alias. Bindings stored in `sequence.json` under `"bindings"` key
4. **Session Management** (lines 550–1160) — `session list|info|search|relocate|clean|archive|restore|archives`. Path encoding: `/` → `-` for directory names under `~/.claude/projects/`
5. **Account Management** (lines 1160–2800) — Switching (checks project bindings first, supports `--isolated [--quiet]` for CLAUDE_CONFIG_DIR profiles), reordering (two-pass credential rename with pre-validated JSON), bind/unbind, shell hook (`ccm hook [--isolated]`), export/import
6. **Help System** (lines 2870–3350) — Topic-based help with `show_help()`, covers all modules including profiles, watch, recover, setup
7. **Environment Snapshots** (lines 3350–3755) — Capture/restore settings.json, MCP config, CLAUDE.md (strips tokens on save)
8. **Usage Module** (lines 3755–4345) — `usage summary|top|history|dashboard|compare` (history parses JSONL for token analytics using jq, dashboard attributes usage to accounts via switch history)
9. **Health & Maintenance** (lines 4345–5330) — `doctor` (13 checks), `clean` (9 targets + all), `permissions audit` (duplicate/dead rule detection)
10. **Profiles Module** (lines 5333–5535) — `switch_isolated [--quiet]` creates CLAUDE_CONFIG_DIR profiles (quiet mode prints only the path for hook use), `cmd_profiles` routes list/sync/delete
11. **Watch Module** (lines 5536–5724) — `cmd_watch` routes start/stop/status, background polling of `rate-limits.json`
12. **Usage Dashboard Module** (lines 5725–5942) — `usage_dashboard` with per-account token attribution, `format_token_count` helper
13. **Session Archive Module** (lines 5943–6144) — `session_archive` compresses old JSONL to tar.gz, `session_restore`, `session_archives_list`
14. **Setup Module** (lines 6145–6296) — `cmd_setup` interactive first-run wizard (6 steps)
15. **Recover Module** (lines 6297–6409) — `cmd_recover` checks credential consistency (4 checks)
16. **Statusline Module** (lines 6410–6691) — `statusline install|remove` generates a bash script that reads Claude Code session JSON via stdin, writes rate-limits.json for the watcher
17. **Init Module** (lines 6693–6889) — `init` auto-generates `.claudeignore` by detecting project type from manifest files
18. **Permissions Module** (lines 6890–7050) — `permissions audit [--fix]` scans settings.json for duplicate/contradictory/dead rules
19. **Main Entry** (lines 7050–7115) — `--no-color` parsing, dependency checks, case-based command dispatch with deprecation notices for removed commands

### Data layout

```
~/.claude-switch-backup/
├── sequence.json              # Account registry (metadata, history, aliases, bindings)
├── credentials/               # Per-account OAuth backups (atomic writes)
├── configs/                   # Per-account config backups
├── snapshots/                 # Environment snapshots
├── profiles/                  # Isolated CLAUDE_CONFIG_DIR profiles (NEW in v4.0)
│   ├── work/                  # Complete Claude Code config directory
│   └── personal/
├── archives/                  # Compressed session archives (NEW in v4.0)
│   ├── index.json             # Archive metadata
│   └── *.tar.gz               # Compressed sessions
├── usage-history.json         # Per-account usage aggregates (NEW in v4.0)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dr5hn/ccm](https://github.com/dr5hn/ccm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
