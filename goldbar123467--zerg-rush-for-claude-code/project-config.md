---
trigger: always_on
description: This project uses a **Zerg Rush-style agent swarm** optimized for speed, parallelism, and disposability.
---

# Zerg Rush Agent Swarm

## Overview

This project uses a **Zerg Rush-style agent swarm** optimized for speed, parallelism, and disposability.

Agents are **short-lived**, **low-context**, and **timeboxed**.
No long-running agents. No deep deliberation. No architecture drift.

**Spawn -> Bite -> Die -> Repeat.**

---

## Agent Mail Protocol (MANDATORY)

Every agent session MUST:
1. Register on startup: `mcp__mcp-agent-mail__register_agent`
2. Check inbox: `mcp__mcp-agent-mail__fetch_inbox`
3. Reserve files before editing: `mcp__mcp-agent-mail__file_reservation_paths`
4. Release reservations when done: `mcp__mcp-agent-mail__release_file_reservations`

### Project Key
```
/home/ubuntu/projects/zerg-swarm
```

### Message Conventions
Subject prefixes: `[TASK]` `[DONE]` `[PARTIAL]` `[BLOCKED]` `[FAILED]`

---

## Agent Roles

### OVERLORD (Opus)
- Breaks large goals into **microtasks** fitting the 4-min / 100-line rule
- Assigns tasks in **waves of 4-5 zerglings**
- Maintains `/SWARM/STATE.json`
- Rejects oversized tasks and splits them further
- Merges results and creates follow-up tasks

### ZERGLING (Sonnet)
- Completes **exactly one task**
- Obeys all hard limits
- Writes results and **stops immediately**

---

## Zergling Hard Constraints (NON-NEGOTIABLE)

| Constraint | Limit |
|------------|-------|
| Timebox | 4 minutes (hard stop) |
| Max new lines | 100 |
| Files touched | 1 (max 2 if 2nd is test/docs) |
| New dependencies | NONE |
| Architectural decisions | NONE |
| Refactors outside task | NONE |
| Exploration beyond listed files | NONE |

If limits are exceeded, return **PARTIAL**.

---

## RAG Brain Protocol

Every agent session SHOULD use RAG Brain for shared memory:

1. **On spawn**: `mcp__rag-brain__recall` - Retrieve prior context and decisions
2. **On complete**: `mcp__rag-brain__remember` - Store important decisions and outcomes
3. **Always**: `mcp__rag-brain__feedback` - Provide feedback on useful memories

### Usage Examples

```
# Recall context before starting work
recall: "zerg-swarm architecture decisions"

# Remember a decision made
remember: "Chose atomic write pattern for STATE.json to prevent corruption"

# Provide feedback on a memory that was helpful
feedback: memory_id=123, helpful=true
```

### When to Use

- **recall**: At session start, before major decisions
- **remember**: After completing tasks, making architectural decisions
- **feedback**: When a recalled memory helped (or didn't)

---

## MCP Server

The Zerg Swarm MCP Server provides programmatic access to swarm coordination.

### Starting the Server
```bash
cd /home/ubuntu/projects/zerg-swarm
pip install -e .
python -m zerg_swarm_mcp
# Server starts on http://127.0.0.1:8766
```

### Available Tools

| Tool | Description |
|------|-------------|
| `swarm_status` | Get current swarm state |
| `swarm_reset` | Reset state to initial values |
| `task_list` | List tasks by lane |
| `task_get` | Read a task card |
| `task_create` | Create new task card |
| `zergling_register` | Register active zergling |
| `zergling_unregister` | Remove zergling |
| `lock_acquire` | Reserve files for editing |
| `lock_release` | Release file locks |
| `wave_status` | Get wave statistics |
| `wave_increment` | Advance wave counter |
| `health_check` | System diagnostics |

### Configuration
Server runs on `http://127.0.0.1:8766` by default. Override with environment variables:
- `ZERG_HOST` - Server host (default: 127.0.0.1)
- `ZERG_PORT` - Server port (default: 8766)
- `ZERG_SWARM_ROOT` - Path to SWARM directory

---

## File Locking Protocol

Before editing ANY file:

```
1. Reserve:  mcp__mcp-agent-mail__file_reservation_paths
2. Edit:     Make changes
3. Release:  mcp__mcp-agent-mail__release_file_reservations
```

**NEVER edit files reserved by another agent.**

---

## Task Card Format

All tasks live in `/SWARM/TASKS/` and follow this structure:

```markdown
# Task: [LANE_PREFIX-NNN]

## Metadata
- Wave: X
- Zergling: (assigned at runtime)
- Status: PENDING | IN_PROGRESS | DONE | PARTIAL | BLOCKED

## Context
Files to read (ONLY these):
- path/to/file.py

## Objective
Single-sentence goal.

## Deliverables
- [ ] Output 1
- [ ] Output 2

## Constraints
- Max 100 lines
- Max 2 files
```

### Task ID Naming Convention

Task IDs use lane-specific prefixes:
- **KERNEL lane**: K001, K002, K003, ...
- **ML lane**: M001, M002, M003, ...
- **QUANT lane**: Q001, Q002, Q003, ...
- **DEX lane**: D001, D002, D003, ...
- **INTEGRATION lane**: INT-001, INT-002, INT-003, ...
- **DOC lane**: DOC-001, DOC-002, DOC-003, ...
- **META lane**: META-001, META-002, META-003, ...

File locations:
```
TASKS/KERNEL/K001.md → INBOX/K001_RESULT.md
TASKS/ML/M001.md → INBOX/M001_RESULT.md
TASKS/INTEGRATION/INT-001.md → INBOX/INT-001_RESULT.md
```

---

## State Management

The OVERLORD maintains `/SWARM/STATE.json`:

```json
{
  "wave": 0,
  "active_zerglings": [],
  "completed_tasks": [],
  "pending_tasks": [],
  "last_updated": "<timestamp>"
}
```

---

## Status Codes

| Code | Meaning |
|------|---------|
| `DONE` | Task completed successfully |
| `PARTIAL` | Limits hit, work incomplete |
| `BLOCKED` | Cannot proceed, needs input |
| `FAILED` | Error, task abandoned |

---

## Directory Structure

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/goldbar123467) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
