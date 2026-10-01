---
trigger: always_on
description: Rust CLI for the ClickUp API, optimized for AI agent consumption. Covers all ~130 endpoints across 28 resource groups and 4 utility commands.
---

# clickup-cli

Rust CLI for the ClickUp API, optimized for AI agent consumption. Covers all ~130 endpoints across 28 resource groups and 4 utility commands.

## Build & Test

```bash
cargo build                    # Build
cargo test                     # Run all tests
cargo test --test test_cli     # CLI smoke tests only
cargo test --test test_output  # Output formatting tests only
cargo run -- --help            # Show help
```

## Architecture

- Ships two binaries: `clickup-cli` (canonical) and `clkup` (short alias, identical behaviour)
- Entry: `src/main.rs` → `src/lib.rs`
- Commands: `src/commands/{resource}.rs` — one file per API resource group
- Models: `src/models/{resource}.rs` — serde structs for API responses
- Core: `src/client.rs` (HTTP + retry), `src/config.rs` (TOML), `src/output.rs` (formatting), `src/error.rs` (errors)

## CLI Pattern

```
clickup-cli <resource> <action> [ID] [flags]
```

## Command Groups

### Core (v0.1)
- `setup` — configure API token and default workspace
- `auth` — whoami, check
- `workspace` — list, seats, plan
- `space` — list, get, create, update, delete
- `folder` — list, get, create, update, delete
- `list` — list, get, create, update, delete, add-task, remove-task
- `task` — list, search, get, create, update, delete, time-in-status, add-tag, remove-tag, add-dep, remove-dep, link, unlink, move, set-estimate, replace-estimates

### Collaboration (v0.2)
- `checklist` — create, update, delete, add-item, update-item, delete-item
- `comment` — list, create, update, delete, replies, reply
- `tag` — list, create, update, delete
- `field` — list, set, unset
- `task-type` — list
- `attachment` — list, upload

### Tracking (v0.3)
- `time` — list, get, current, create, update, delete, start, stop, tags, add-tags, remove-tags, rename-tag, history
- `goal` — list, get, create, update, delete, add-kr, update-kr, delete-kr
- `view` — list, get, create, update, delete, tasks
- `member` — list
- `user` — invite, get, update, remove

### Communication (v0.4)
- `chat` — channel-list, channel-create, channel-get, channel-update, channel-delete, channel-followers, channel-members, dm, message-list, message-send, message-update, message-delete, reaction-list, reaction-add, reaction-remove, reply-list, reply-send, tagged-users
- `doc` — list, create, get, pages, add-page, page, edit-page, embed-image
- `webhook` — list, create, update, delete
- `template` — list, apply-task, apply-list, apply-folder

### Admin (v0.5)
- `guest` — invite, get, update, remove, share-task, unshare-task, share-list, unshare-list, share-folder, unshare-folder
- `group` — list, create, update, delete
- `role` — list
- `shared` — list
- `audit-log` — query (Enterprise)
- `acl` — update (Enterprise)

### Utilities
- `status` — show current config, token (masked), workspace
- `completions` — generate shell completions (bash, zsh, fish, powershell)
- `agent-config` — show or inject CLI reference (auto-detects CLAUDE.md, agent.md, .cursorrules, etc.)
- `mcp serve` — start MCP server (JSON-RPC over stdio, 144 tools, 100% API coverage)

## Global Flags

- `--token TOKEN` — override config file token
- `--workspace ID` — override default workspace
- `--output MODE` — table (default), json, json-compact, csv
- `--fields LIST` — comma-separated field names
- `--no-header` — omit table header
- `--all` — auto-walk every page; applies to every paginated list command (page, cursor, start-id, body styles)
- `--limit N` — cap total items returned (enforced after walking, so `--all --limit 500` returns up to 500 across N pages)
- `--page N` — manual page (v2 page-style endpoints: `task list`, `task search`, `view tasks`, `template list`)
- `--cursor X` — manual cursor (v3 cursor-style endpoints: `doc list`, all `chat *` list commands)
- `--start MS` + `--start-id ID` — manual boundary pair, local to `comment list` / `comment replies` (not global; a global `--start` collided with `time create`/`time update --start`, see #91)
- `-q` / `--quiet` — IDs only
- `--timeout SECS` — HTTP timeout (default 30)

## Config

| Level | File |
|-------|------|
| Project | `.clickup.toml` (current directory) |
| Global | `~/.config/clickup-cli/config.toml` |

```toml
[auth]
token = "pk_..."

[defaults]
workspace_id = "12345"

[git]
enabled = true      # auto-detect task ID from current git branch (default: true)
verbose = true      # print "resolved task X from branch Y" breadcrumb (default: true)
```

Resolution: `--flag` > `CLICKUP_TOKEN`/`CLICKUP_WORKSPACE` env > `.clickup.toml` > global config

## Auto-detect task ID from git branch

When a task-scoped command runs without an explicit ID, the CLI resolves the ID from the current git branch. Matches `CU-abc123` (→ `abc123`) and custom `PREFIX-NUMBER` IDs (→ `PROJ-42` with `custom_task_ids=true&team_id=<ws>` auto-injected). Conventional-commits prefixes (`feat/`, `fix/`, etc.) are stripped first; workflow keywords (`FEATURE-`, `BUGFIX-`, `WIP-`, `TMP-`, …) are excluded.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nicholasbester/clickup-cli](https://github.com/nicholasbester/clickup-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
