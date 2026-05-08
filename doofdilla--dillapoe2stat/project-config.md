---
trigger: always_on
description: **Path of Exile 2 farming tracker** with phase-based architecture for inventory snapshots, loot valuation, and session analytics. Hotkey-driven Windows tool with OBS overlay support for streamers.
---

# DillaPoE2Stat - AI Coding Agent Instructions

## Project Overview

**Path of Exile 2 farming tracker** with phase-based architecture for inventory snapshots, loot valuation, and session analytics. Hotkey-driven Windows tool with OBS overlay support for streamers.

**Tech Stack:** Python 3.10+, Windows-only (`keyboard`, `win11toast`), OAuth API integration, Flask web server

**Core Entry Point:** `poe_stats_refactored_v2.py` - main tracker with global hotkey bindings

## ⚠️ CRITICAL: Git Commit Rules (READ THIS FIRST!)

**EVERY COMMIT MUST HAVE GITMOJI - NO EXCEPTIONS!**

### How to Commit (Windows)

**❌ NEVER do this:**
```bash
git commit -m "feat: Add feature"  # Missing gitmoji! PowerShell strips emojis!
```

**✅ ALWAYS do this:**
```bash
& "C:\Program Files\Git\bin\bash.exe" -c "
  cd /c/temp/dillapoe2stat && 
  printf 'feat: \xE2\x9C\xA8 Add feature\n\n- Detail 1\n- Detail 2\n' > commit_msg.txt && 
  git commit -F commit_msg.txt && 
  rm commit_msg.txt
"
```

**Common gitmoji hex codes (MEMORIZE THESE):**
- ✨ `feat:` new feature → `\xE2\x9C\xA8`
- 🐛 `fix:` bug fix → `\xF0\x9F\x90\x9B`
- 📝 `docs:` documentation → `\xF0\x9F\x93\x9D`
- 🔧 `chore:` config/tooling → `\xF0\x9F\x94\xA7`
- 🚀 `perf:` performance → `\xF0\x9F\x9A\x80`
- 🔖 `chore:` version bump → `\xF0\x9F\x94\x96`

**If you forget gitmoji:**
1. STOP immediately
2. `git reset --soft HEAD~1`
3. Re-commit with proper gitmoji using Git Bash

**Why:** PowerShell cannot handle UTF-8 emojis - they get stripped before Git sees them!

## Available MCP Servers

This project has access to specialized MCP servers for enhanced AI assistance:

### Microsoft Learn MCP Server (`microsoft_docs_*`)
- **Purpose:** Trusted, up-to-date information from official Microsoft documentation
- **Use for:** Windows API questions, Python on Windows, Azure integration, .NET interop
- **Tools available:**
  - `microsoft_docs_search` - Search official docs (returns 10 concise chunks)
  - `microsoft_code_sample_search` - Find code examples from Microsoft Learn
  - `microsoft_docs_fetch` - Get full documentation pages in markdown
- **When to use:** Win32 API questions (like HWND_TOPMOST, WS_EX_TRANSPARENT), ctypes with Windows, Python best practices
- **Workflow:** Search → Get overview → Fetch full page if needed

### MCP Memory Server (`mcp_memory_*`)
- **Purpose:** Persistent knowledge graph across chat sessions
- **See:** "Memory Management" section below for detailed usage guidelines

## Critical Architecture Patterns

### Phase-Based Flow Controller (v0.3.4+)

**THE MOST IMPORTANT PATTERN** - All map tracking uses `MapFlowController` with explicit phases:

```python
# PRE-MAP (F2): 4 phases
Phase 1: Snapshot → Phase 2: Parse → Phase 3: Update State → Phase 4: Notify

# POST-MAP (F3): 9 phases  
Phase 1-3: Snapshot → Diff → Value
Phase 4: Capture session BEFORE (critical for comparison)
Phase 5: Update session (ONLY place to call add_completed_map())
Phase 6-9: Notify → Log → Display → Reset
```

**Why this matters:**
- **Session double-counting bug prevention:** Only Phase 5 can add maps to session
- **Accurate notifications:** Phase 4 captures "session before this map" for ex/h comparison
- Read `docs/SESSION_FLOW.md` for complete flow diagrams and common pitfalls

**When modifying tracking flow:**
1. Never call `session_manager.add_completed_map()` outside `_phase_update_post_state()`
2. Use `_phase_capture_session_before()` for baseline metrics before updating
3. Each phase returns explicit dataclass result (`DiffResult`, `ValueResult`, `SessionSnapshot`)

### Manager-Based Modular Architecture

```
PoEStatsTracker (main orchestrator)
  ├── MapFlowController (phases)
  │     └── InventorySnapshotService (API + rate limiting)
  ├── GameState (central state: map, session, drops)
  ├── SessionManager (session lifecycle, stats)
  ├── InventoryAnalyzer (diff, categorize)
  ├── DisplayManager (console output, themes)
  ├── NotificationManager (Windows toasts)
  ├── WaystoneAnalyzer (experimental waystone parsing)
  └── AutoMapDetector (Client.txt monitoring)
```

**Managers are singletons** - initialize once in `PoEStatsTracker.__init__`, pass to flow controller

## Data Persistence & Versioning

### JSON Lines Format (Not JSON)

```python
# runs.jsonl - ONE JSON object per line (append-only)
{"run_id": "abc123", "format_version": "2.1", "delirious": 0, ...}
{"run_id": "def456", "format_version": "2.1", "delirious": 15, ...}

# sessions.jsonl - Event log
{"event_type": "session_start", "session_id": "xyz", ...}
{"event_type": "session_end", "session_id": "xyz", "total_maps": 5, ...}
```

**Important:** `runs.jsonl` and `sessions.jsonl` are **NOT tracked in Git** (local data only)
- Listed in `.gitignore` to prevent accidental commits
- Sample files provided: `runs.jsonl.sample`, `sessions.jsonl.sample`
- New users: Copy `.sample` files to create initial data files

**Data Format Versions:**
- `2.0`: Added per-item valuations (chaos/exalted/divine)
- `2.1`: Added `delirious` field for waystone tracking
- Use `upgrade_runs_data.py` to migrate old data (creates timestamped backups)

**When adding new fields:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DoofDilla/dillapoe2stat](https://github.com/DoofDilla/dillapoe2stat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
