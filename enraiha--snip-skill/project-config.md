---
trigger: always_on
description: |
---


# snip: Save Tokens Without Breaking Your Agent

snip is a CLI proxy that sits between your AI coding assistant and the shell.
It filters verbose command output — turning 689 tokens of `go test` into
"10 passed, 0 failed" (16 tokens). **60-90% token savings.**

But there's a catch: filters work silently. If a filter is too aggressive,
your agent gets incomplete information and doesn't know it missed something.
One wrong filter can turn a debugging session into a guessing game.

This guide shows you exactly which filters are safe, which are dangerous,
and — most importantly — **how to escape at any moment**.

---

## Before You Start: The Escape Protocol

Think of this like a plane safety briefing. You don't need it until you do.

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  OUTPUT LOOKS WRONG?                                           │
│                                                                 │
│  Step 1 — Bypass one command:                                   │
│    snip proxy <same command>                                    │
│    Example: snip proxy curl http://localhost:8585/health        │
│    Forces raw passthrough for that single command.              │
│                                                                 │
│  Step 2 — See what snip hid:                                   │
│    ls ~/.local/share/snip/tee/                                  │
│    Every filtered command's raw output is saved to disk.        │
│    Find the latest file and cat it.                             │
│                                                                 │
│  Step 3 — Disable the bad filter permanently:                  │
│    Edit ~/.config/snip/config.toml                              │
│    [filters.enable]                                             │
│    curl = false                                                 │
│    No restart needed — next command uses the new config.        │
│                                                                 │
│  Step 4 — Nuke from orbit (last resort):                        │
│    Remove "opencode-snip@latest" from opencode.json plugins.    │
│    Close terminal, reopen. snip is gone.                        │
│                                                                 │
│  SAFETY NET: Every filter has on_error: passthrough.            │
│  If a filter crashes or panics — raw output flows through.      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Test the escape now, before you need it:**

```bash
snip proxy echo "this works"     # bypasses snip entirely
snip echo "this goes through snip"  # (if snip has a filter for echo)
```

The plugin skips `cd`, `source`, `.`, `export`, `alias`, `unset`, `set`,
`shopt`, `eval`, and `exec` automatically. Those always run raw.

---

## How snip Filters Actually Work

A filter is a YAML file. The binary is the engine; filters are data.
They evolve independently. You can write one without touching Go.

Here's a real filter, fully annotated:

```yaml
name: "git-log"
version: 1
description: "Condense git log to hash + message + author + date"

match:
  command: "git"
  subcommand: "log"
  exclude_flags: ["--format", "--pretty", "--graph", "--oneline"]
  # ^ Only matches `git log` without these flags

inject:
  args: ["--pretty=format:%h %s (%ar) <%an>", "--no-merges"]
  defaults:
    "-n": "10"
  # ^ Injects arguments. If you run plain `git log`, snip runs:
  #   git log --pretty=format:"%h %s (%ar) <%an>" --no-merges -n 10
  # ^ THIS IS THE KEY INSIGHT — some filters modify your command.

pipeline:
  - action: "keep_lines"
    pattern: "\\S"
  - action: "truncate_lines"
    max: 80
    ellipsis: "..."
  - action: "format_template"
    template: "{{.count}} commits:\n{{.lines}}"
  # ^ Three pipeline stages: keep non-empty, truncate long lines, format

on_error: "passthrough"
  # ^ EVERY filter has this. If ANYTHING goes wrong — raw output passes through.
```

**Three things filters can do:**

| Action | Example | What it does |
|--------|---------|-------------|
| **Remove noise** | `remove_lines`, `strip_ansi` | Strips progress bars, ANSI colors, download logs |
| **Inject args** | `inject.args` | Rewrites your command to produce cleaner output |
| **Condense output** | `head`, `truncate_lines`, `aggregate` | Keeps first N lines, summarizes counts |

**The 19 pipeline actions:** `keep_lines`, `remove_lines`, `truncate_lines`,
`strip_ansi`, `head`, `tail`, `group_by`, `dedup`, `json_extract`,
`json_schema`, `ndjson_stream`, `regex_extract`, `state_machine`,
`aggregate`, `format_template`, `compact_path`, `replace`, `match_output`,
`on_empty`.

Every filter has at least one of these. Understanding which ones go too far
is the difference between safe savings and silent data loss.

### Commands without a matching filter

If you run a command that has no filter, snip passes it through unchanged
with ~10ms overhead. Over 70% of commands hit no filter in a typical session.

---

## Installation (3 ways)

```bash
# Option 1: Quick install script (macOS/Linux) — needs curl + tar
curl -fsSL https://raw.githubusercontent.com/edouard-claude/snip/master/install.sh | sh


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EnRaiha/snip-skill](https://github.com/EnRaiha/snip-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
