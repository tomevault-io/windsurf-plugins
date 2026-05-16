---
trigger: always_on
description: This file documents Claude's enhanced contextual memory capabilities through the **smarter-claude** system.
---

# Claude Contextual Memory System

This file documents Claude's enhanced contextual memory capabilities through the **smarter-claude** system.

## Cross-Platform Settings

**`settings.json` is generated locally and never tracked in git.**

The install scripts automatically generate the correct `settings.json` for your platform:
- **macOS/Linux**: `setup.sh` generates `settings.json` with Unix paths (`~/.claude/`)
- **Windows**: `setup.ps1` generates `settings.json` with Windows paths (`%USERPROFILE%\.claude\`)

**This means:**
- `settings.json` is in `.gitignore` - never commit it
- Each platform gets the correct paths automatically
- No manual copying or path fixes needed

**If hooks fail with "Failed to spawn":**

Re-run the setup script to regenerate settings.json:

```bash
# macOS/Linux
bash ~/.claude/setup.sh

# Windows (PowerShell)
powershell -ExecutionPolicy Bypass -File $env:USERPROFILE\.claude\setup.ps1
```

## Project Context Check

Before starting any task, run:
`ls -la .claude/smarter-claude/smarter-claude.db`

If this file exists, you have access to rich contextual memory.

## CRITICAL: Database-First Policy

BEFORE using git log, git show, or any historical analysis:
1. ALWAYS check if `.claude/smarter-claude/smarter-claude.db` exists
2. If it exists, query it FIRST for file changes and context
3. Only use git as a fallback or for verification

**Trigger phrases that MUST use database first:**
- "what files were changed"
- "what did we work on"
- "recent changes"
- "which files were modified"
- "what was the last thing we did"
- "what did we work on recently"
- "recent activity"
- "previous work"
- "file history"
- "changes made"

## Overview

Claude now has access to a sophisticated contextual memory database that automatically tracks:
- User intents and requests
- File modifications with WHY context
- Task delegation and subagent usage  
- Execution summaries and workflow insights

## Database Schema

The contextual memory uses a 4-table SQLite schema designed for fast context retrieval:

### 1. `cycles` - Core Request Tracking
```sql
CREATE TABLE cycles (
    cycle_id INTEGER PRIMARY KEY,
    session_id TEXT NOT NULL,
    user_intent TEXT,           -- The original user request
    phase_number INTEGER,       -- Project phase tracking  
    task_number INTEGER,        -- Task within phase
    start_time TIMESTAMP,
    end_time TIMESTAMP,
    primary_activity TEXT       -- file_modification, testing, git-operation, etc.
);
```

**Purpose**: Tracks each request cycle with the original user intent as the primary driver.

### 2. `file_contexts` - File Changes with WHY Context
```sql
CREATE TABLE file_contexts (
    id INTEGER PRIMARY KEY,
    cycle_id INTEGER REFERENCES cycles(cycle_id),
    file_path TEXT NOT NULL,    -- What file was changed
    agent_type TEXT,            -- main_agent, subagent
    operation_type TEXT,        -- edit, write, multiedit
    change_reason TEXT,         -- WHY the change was made
    edit_count INTEGER,         -- Number of edits
    timestamp TIMESTAMP
);
```

**Purpose**: Captures not just WHAT files were changed, but WHY they were changed.

### 3. `llm_summaries` - Generated Insights
```sql
CREATE TABLE llm_summaries (
    id INTEGER PRIMARY KEY, 
    cycle_id INTEGER REFERENCES cycles(cycle_id),
    intent_sequence INTEGER,   -- For multi-intent cycles
    summary_text TEXT,         -- Generated summary content
    summary_type TEXT,         -- user_intent, execution_summary, workflow_insights
    confidence_level TEXT     -- high, medium, low
);
```

**Purpose**: Stores generated insights and summaries for complex request cycles.

### 4. `subagent_tasks` - Delegation Context
```sql
CREATE TABLE subagent_tasks (
    id INTEGER PRIMARY KEY,
    cycle_id INTEGER REFERENCES cycles(cycle_id), 
    task_description TEXT,     -- What was delegated
    files_modified TEXT,       -- JSON array of files
    status TEXT,              -- completed, failed, in_progress
    completion_time TIMESTAMP
);
```

**Purpose**: Tracks task delegation to specialized agents with their outcomes.

## Convenience Views for Simplified Queries

To make database queries more accessible, create these views:

```sql
-- Recent file changes (last 7 days) with WHY context
CREATE VIEW recent_file_changes AS
SELECT c.user_intent, fc.file_path, fc.change_reason, 
       fc.operation_type, fc.timestamp, c.cycle_id,
       c.primary_activity
FROM cycles c 
JOIN file_contexts fc ON c.cycle_id = fc.cycle_id
WHERE c.start_time > datetime('now', '-7 days')
ORDER BY fc.timestamp DESC;

-- Recent activity summary with change reasoning
CREATE VIEW recent_activity AS
SELECT c.user_intent, c.primary_activity, c.start_time,
       GROUP_CONCAT(fc.file_path || ' (' || fc.change_reason || ')') as files_and_reasons
FROM cycles c 
LEFT JOIN file_contexts fc ON c.cycle_id = fc.cycle_id
WHERE c.start_time > datetime('now', '-7 days')
GROUP BY c.cycle_id
ORDER BY c.start_time DESC;

-- File modification history with full context
CREATE VIEW file_history AS
SELECT fc.file_path, c.user_intent, fc.change_reason,
       fc.operation_type, fc.timestamp, c.primary_activity,
       CASE 
         WHEN fc.change_reason IS NOT NULL THEN fc.change_reason

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [okets/.claude](https://github.com/okets/.claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
