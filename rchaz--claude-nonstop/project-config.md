---
trigger: always_on
description: - Repo: https://github.com/rchaz/claude-nonstop
---

# Repository Guidelines

- Repo: https://github.com/rchaz/claude-nonstop
- Architecture: [DESIGN.md](DESIGN.md)

## Project Overview

claude-nonstop is a Node.js CLI tool that provides multi-account switching and Slack remote access for Claude Code. It handles OAuth credentials, rate limit detection, session migration, and Slack channel lifecycle.

## Project Structure & Module Organization

```
bin/claude-nonstop.js         CLI entry point, command routing (ESM)
lib/                          Core logic (ESM, .js)
  config.js                   Account registry, path constants, validation
  runner.js                   Process wrapper, rate limit detection, session migration
  usage.js                    Anthropic usage API client
  scorer.js                   Best-account selection algorithm
  keychain.js                 OS credential store reading
  session.js                  Session file migration + cross-profile search
  service.js                  launchd service management (macOS)
  tmux.js                     tmux session management
  platform.js                 OS detection
  reauth.js                   Re-authentication flow
remote/                       Slack remote access subsystem (CJS, .cjs)
  hook-notify.cjs             Hook entry point (called by Claude Code hooks and runner.js)
  webhook.cjs                 Socket Mode handler (Slack -> tmux relay)
  start-webhook.cjs           Webhook process entry point
  channel-manager.cjs         Slack channel lifecycle (create, post, archive)
  paths.cjs                   Shared path constants for CJS modules
  load-env.cjs                .env loader for CJS modules
scripts/
  postinstall.js              Restart webhook service on npm install
```

### Module System (ESM / CJS split)

- `lib/` uses **ESM** (`import`/`export`, `.js` extension)
- `remote/` uses **CJS** (`require`/`module.exports`, `.cjs` extension)
- `bin/claude-nonstop.js` is ESM

CJS is required in `remote/` because Claude Code spawns hook scripts as standalone Node.js processes — they must be self-contained.

**Do not** convert `remote/*.cjs` to ESM. **Do not** add ESM imports to `remote/` files.

### Path Constants

Two sources of truth for paths (one per module system):
- **ESM modules** import from `lib/config.js`: `CONFIG_DIR`, `DEFAULT_CLAUDE_DIR`, `PROFILES_DIR`
- **CJS modules** import from `remote/paths.cjs`: `CONFIG_DIR`, `ENV_PATH`, `DATA_DIR`, `CHANNEL_MAP_PATH`

Do not duplicate path definitions. Do not hardcode `~/.claude-nonstop/` anywhere.

## Build, Test, and Verification

### Verify syntax after changes

```bash
npm run check
```

Or individually:

```bash
node --check lib/*.js
node --check remote/*.cjs
node --check bin/claude-nonstop.js
node --check scripts/postinstall.js
```

### Run the CLI

```bash
claude-nonstop help
claude-nonstop list
claude-nonstop status
claude-nonstop resume           # resume most recent session (any account)
claude-nonstop resume <id>      # resume specific session by ID
```

### Test account name validation

```bash
# These should fail with validation errors:
claude-nonstop add '../bad'
claude-nonstop add 'has spaces'
claude-nonstop add ''
```

### Verify hooks

```bash
claude-nonstop hooks status
```

## Coding Style & Conventions

- Runtime baseline: Node **22+** (24 LTS recommended)
- Use `const` over `let` where possible
- Use descriptive variable names
- Keep functions focused and small
- Follow existing patterns in the codebase
- Add brief comments only for tricky or non-obvious logic
- No TypeScript — this is a plain Node.js project

## Security Rules

These are **hard rules**. Do not relax them.

- **Never** use `exec()` or `execSync()` with string interpolation for subprocess calls. Always use `execFile`/`execFileSync` with array arguments.
- **Never** construct shell commands by concatenating user input (account names, session IDs, Slack messages, file paths).
- **Always** validate account names with `validateAccountName()` from `lib/config.js` before using them in file paths.
- **Always** use `AbortController` with timeout on HTTP fetch calls.
- **Always** use atomic writes (write-to-temp + rename) for shared data files like `channel-map.json` and `config.json`.
- **Never** log or expose OAuth tokens (`sk-ant-oat01-*`, `sk-ant-ort01-*`), Slack tokens (`xoxb-*`, `xapp-*`), or other credentials in output, error messages, or stack traces.
- Truncate tmux message relay to `MAX_TMUX_MESSAGE_LENGTH` (4096 chars).
- Slack `.env` files must live in `~/.claude-nonstop/.env`, never in the project directory.

## Hook Notification Types

`hook-notify.cjs` handles these notification types (passed as first CLI argument):

| Type | Source | Purpose |
|------|--------|---------|
| `session-start` | Claude Code SessionStart hook | Create per-session Slack channel |
| `completed` | Claude Code Stop hook | Post structured completion message |
| `tool-use` | Claude Code PostToolUse hook | Buffer tool activity, flush to Slack every 10s |
| `waiting-for-input` | Claude Code PreToolUse hook (ExitPlanMode, AskUserQuestion) | Notify when Claude is waiting for user input |
| `account-switch` | runner.js (on rate limit) | Notify about account switch |

## Slack Control Commands

In per-session Slack channels, these commands are handled before tmux relay:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rchaz/claude-nonstop](https://github.com/rchaz/claude-nonstop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
