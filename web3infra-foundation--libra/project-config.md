---
trigger: always_on
description: Manage external-agent capture for Claude Code, Codex, and OpenCode.
---

# `libra agent`

Manage external-agent capture for Claude Code, Codex, and OpenCode.

## Synopsis

```bash
libra agent status
libra agent list [--schema-version <1|2>] [--json]
libra agent import (--session <id> | --path <path> | --since <rfc3339> | --all) [--agent <name>] [--limit <n>] [--cursor <n>] --yes
libra agent graph <session> [--repo <path>]
libra agent enable [--agent <name>]...
libra agent add [<name>...]
libra agent disable [--agent <name>]...
libra agent remove [<name>...]
libra agent session <subcommand>
libra agent checkpoint <subcommand>
libra agent skill <subcommand>
libra agent clean [--all]
libra agent doctor [--repair]
libra agent push [--remote <name>] [--force-rewrite]
libra agent rpc <subcommand>
```

## Description

`libra agent` manages Libra's external-agent capture surface. It installs and
removes provider hooks, reports captured session/checkpoint state, exposes
read-only diagnostics, and can push `refs/libra/traces` to a remote.

The supported roster is `claude-code`, `codex` and `opencode` (first batch),
and all three are hook-installable: `claude-code` writes `.claude/settings.json`,
`codex` writes user-level `$CODEX_HOME/hooks.json` plus Libra-managed trust
entries in `$CODEX_HOME/config.toml` (untrusted Codex hooks are skipped
silently, so trust entries are part of the install), and `opencode` writes the
Libra-managed plugin `.opencode/plugin/libra-hooks.js` (note: `opencode --pure`
disables all external plugins, including capture).
`gemini` was demoted out of the supported roster and is uninstall-only:
`libra agent remove gemini` removes previously installed Libra-managed hooks
(idempotent), captured sessions stay readable, and `add`/`enable` for it — or
for any other non-roster agent — return an actionable unsupported error.

## Subcommands

| Subcommand | Description |
|------------|-------------|
| `status` | Report captured external-agent session status |
| `list` | List the supported agents with their capability matrix (roster, hooks, install state) |
| `import` | Discover and import historical Claude/Codex transcript files or one trusted, sandboxed OpenCode export after explicit consent |
| `graph <session>` | Browse the read-only session → turn → revision → subagent capture graph; use global `--json`/`--machine` outside a terminal |
| `enable` | Enable one or more external agents and install hooks |
| `add` | Alias of `enable`: `add <name>` ≡ `enable --agent <name>` |
| `disable` | Disable one or more external agents and uninstall hooks |
| `remove` | Alias of `disable`: `remove <name>` ≡ `disable --agent <name>` |
| `session list` | List captured sessions |
| `session show <id>` | Show a captured session |
| `session stop <id>` | Mark a captured session as stopped |
| `session resume <id>` | Mark a stopped captured session active again |
| `session promote <id>` | Promote a captured session into Libra intent metadata |
| `session derive-tool-calls <id>` | Derive tool-call records from a captured session |
| `checkpoint list` | List captured checkpoints |
| `checkpoint show <id>` | Show checkpoint metadata |
| `checkpoint rewind <id>` | Inspect or apply a working-tree rewind for one checkpoint |
| `checkpoint export <id>` | Export a checkpoint's transcript. Redacted by default (no authorization); raw (un-redacted) export requires `--allow-raw --raw` and is recorded in the append-only `agent_audit_log` (`LBR-AGENT-013` when refused without it) |
| `skill search` | Search captured skill events by `--skill`, `--provider`, `--session`, and RFC3339 `--since`/`--until` (keyset-paginated with `--limit`/`--cursor`, `--json`). A read-time projection over checkpoint metadata — no dedicated table |
| `skill list` | Alias of `skill search` (same filters) |
| `skill registry` | Show the curated per-agent discoverable-skill registry (`--provider <slug>` to scope; the public SkillDiscoverer surface) |
| `clean` | Clean up temporary checkpoints from stopped sessions (prune fails closed while a checkpoint write is in flight, the traces ref reaches uncataloged commits, or durable object-index repair remains pending; also drops `object_index` rows made unreachable) |
| `doctor` | Diagnose hook installation and capture state; detect (and with `--repair` fix) checkpoint-store inconsistencies |
| `push` | Push `refs/libra/traces` to a remote (`--force-rewrite` for the non-fast-forward push after a `clean` prune, using force-with-lease) |
| `rpc list` | List discovered `libra-agent-*` binaries on `PATH` (with trusted/quarantined state); requires the external-agents opt-in |
| `rpc trust <slug>` | Trust a discovered binary — records path + sha256 + device/inode/mtime provenance (refused when its directory is world-writable, or when the binary is not under a trusted directory — `LBR-AGENT-005`). The provider-exporter slug `opencode` instead pins the provider's own CLI binary — resolved only from registered trusted directories, never `$PATH` — for the sandboxed export bridge; this form needs no external-agents opt-in |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [web3infra-foundation/libra](https://github.com/web3infra-foundation/libra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
