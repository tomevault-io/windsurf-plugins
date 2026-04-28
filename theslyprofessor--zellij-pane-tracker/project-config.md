---
trigger: always_on
description: A Zellij plugin + MCP server that gives AI assistants visibility into all terminal panes.
---

# Zellij Pane Tracker Plugin

A Zellij plugin + MCP server that gives AI assistants visibility into all terminal panes.

## Quick Context

**What it does:** Exports pane metadata to JSON + MCP server for AI assistants to read/interact with panes

**Why:** Lets AI coding assistants (OpenCode, etc.) "see" and interact with other terminal panes

**Status:** ✅ **FULLY WORKING** - Plugin, companion script, and MCP server all operational

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Zellij Session                          │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌─────────────┐ │
│  │terminal_1│  │terminal_2│  │terminal_3│  │pane-tracker │ │
│  │(opencode)│  │(Pane #1) │  │(Pane #2) │  │  (plugin)   │ │
│  └──────────┘  └──────────┘  └──────────┘  └──────┬──────┘ │
└───────────────────────────────────────────────────┼────────┘
                                                    │
                                    writes metadata │
                                                    ▼
                              /tmp/zj-pane-names.json
                                                    │
                    ┌───────────────────────────────┼───────────────────┐
                    │                               │                   │
                    ▼                               ▼                   ▼
              ~/zjdump                        MCP Server           Direct read
         (cycles + dumps)                   (calls zjdump)      (cat JSON file)
```

## Key Files

| File | Purpose |
|------|---------|
| `src/main.rs` | Plugin source (Rust, compiles to WASM) |
| `mcp-server/index.ts` | MCP server (TypeScript/Bun) - needs fixes |
| `~/.config/zellij/plugins/zellij-pane-tracker.wasm` | Installed plugin |
| `/tmp/zj-pane-names.json` | Output file (pane metadata) |
| `~/zjdump` | **Primary tool** - dump any pane content with full scrollback |

## ~/zjdump - The Companion Script

**Location:** `~/zjdump` (standalone script, not a shell function)

**What it does:**
- Reads pane metadata from `/tmp/zj-pane-names.json`
- Cycles through tabs and panes to find target
- Dumps full scrollback with `--full` flag
- Returns focus to original position

**Usage:**
```bash
~/zjdump              # Dump current focused pane
~/zjdump 3            # Dump terminal_3 by ID
~/zjdump "Pane #2"    # Dump by display name
~/zjdump opencode     # Dump by pane name
```

**Performance:**
- Current pane: ~0.4s (fast path)
- Other pane: ~2s (cycles through tabs/panes)

**Output:** Writes to `/tmp/zjd-{id}.txt` and outputs to stdout

**OpenCode integration:**
```
User: "check pane 2"
OpenCode runs: ~/zjdump 2
```

## Pane Metadata JSON

**File:** `/tmp/zj-pane-names.json`

**Format:**
```json
{
  "panes": {
    "terminal_1": "opencode",
    "terminal_2": "Pane #1",
    "terminal_3": "Pane #2",
    "plugin_0": "(.) - file:/path/to/plugin.wasm"
  },
  "timestamp": 1765146556
}
```

**Updated by:** The Zellij plugin continuously on pane events

## MCP Server (WORKING)

The MCP server provides AI assistants with full Zellij integration via OpenCode's native tools.

**Version:** 0.6.1 (tab-scoped queries with named tab support)

**Pane Identification (Tab-Scoped):**
- `"Pane 1"` or `"1"` → searches **current tab first** (fast), then other tabs
- `"Tab 2 Pane 1"` → goes directly to **Tab #2**, searches only there
- `"shell Pane 1"` → goes directly to tab named **"shell"**
- `"opencode"` → finds pane by name across all tabs
- `"terminal_2"` → explicit terminal ID

**Tools available:**
| Tool | Status | Description |
|------|--------|-------------|
| `zellij_get_panes` | ✅ Working | List all panes with IDs and display names |
| `zellij_dump_pane` | ✅ Working | Get scrollback of any pane (default: last 100 lines) |
| `zellij_run_in_pane` | ✅ Working | Execute commands in other panes |
| `zellij_new_pane` | ✅ Working | Create new panes |
| `zellij_rename_session` | ✅ Working | Rename the Zellij session |

**zellij_dump_pane options:**
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `pane_id` | string | required | Pane identifier (see above) |
| `full` | boolean | false | If true, dump entire scrollback (can be slow/large) |
| `lines` | number | 100 | Number of lines from end to return (ignored if full=true) |

**Config:** `~/.config/opencode/opencode.json`

**Example usage in OpenCode:**
```
User: "check pane 2"
→ zellij_dump_pane("2")  # Searches current tab first

User: "what's in tab 2 pane 1"
→ zellij_dump_pane("Tab 2 Pane 1")  # Goes directly to second tab

User: "check shell pane 2"
→ zellij_dump_pane("shell 2")  # Goes to tab named "shell"

User: "get full output from pane 2"
→ zellij_dump_pane("2", full=true)  # Returns entire scrollback

User: "run tests in pane 2"
→ zellij_run_in_pane("2", "bun test")
```

## Development Workflow

**CRITICAL: After editing `mcp-server/index.ts`, ALWAYS kill the MCP process:**
```bash
pkill -f "bun.*zellij-pane-tracker/mcp-server"
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [theslyprofessor/zellij-pane-tracker](https://github.com/theslyprofessor/zellij-pane-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
