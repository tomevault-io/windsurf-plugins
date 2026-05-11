---
trigger: always_on
description: This document guides AI agents (Claude, GPT, Cursor, etc.) on how to use Tracer effectively.
---

# Tracer for AI Agents

This document guides AI agents (Claude, GPT, Cursor, etc.) on how to use Tracer effectively.

## Quick Start

```bash
# Initialize tracer in your project
tracer init

# Create your first issue
tracer create "Implement user authentication" -p 1 -t feature

# See what's ready to work on
tracer ready --json
```

## Core Workflow for AI Agents

### 1. Find Ready Work

At the start of any session, check for ready work:

```bash
# Get unblocked work in JSON format
tracer ready --json

# Filter by priority
tracer ready --priority 1 --json

# Limit results
tracer ready --limit 5 --json
```

### 2. Claim and Start Work

```bash
# Update issue to in_progress
tracer update <issue-id> --status in_progress --json

# Example:
tracer update bd-1 --status in_progress
```

### 3. Create Issues During Work

As you discover new work, file it immediately:

```bash
# Create a bug you found
tracer create "Fix edge case in validation" -t bug -p 0 --json

# Link it back to parent work with discovered-from
tracer dep add <new-issue-id> <parent-issue-id> --type discovered-from
```

### 4. Add Dependencies

When you realize work blocks other work:

```bash
# bd-2 is blocked by bd-1
tracer dep add bd-2 bd-1 --type blocks

# For parent-child relationships (epics)
tracer dep add bd-3 bd-1 --type parent-child
```

### 5. Complete Work

```bash
# Close the issue
tracer close <issue-id> --reason "Implemented and tested"

# Example with multiple issues
tracer close bd-1 bd-2 bd-3 --reason "All completed"
```

### 6. Check Status

```bash
# View issue details
tracer show <issue-id> --json

# See overall statistics
tracer stats --json

# Check what's blocked
tracer blocked --json
```

## JSON Output Format

All commands support `--json` for programmatic parsing:

### Ready Work Response

```json
[
  {
    "id": "bd-1",
    "title": "Implement feature X",
    "status": "open",
    "priority": 1,
    "issue_type": "feature",
    "created_at": "2025-10-15T10:00:00Z",
    "updated_at": "2025-10-15T10:00:00Z"
  }
]
```

### Issue Creation Response

```json
{
  "id": "bd-42",
  "title": "New issue",
  "status": "open",
  "priority": 2,
  "issue_type": "task",
  "created_at": "2025-10-15T10:30:00Z",
  "updated_at": "2025-10-15T10:30:00Z"
}
```

## Best Practices for AI Agents

### 1. Always Check Ready Work First

Before starting any new work, check what's unblocked:

```bash
WORK=$(tracer ready --limit 1 --json)
if [ "$(echo $WORK | jq length)" -gt 0 ]; then
  ISSUE_ID=$(echo $WORK | jq -r '.[0].id')
  echo "Working on: $ISSUE_ID"
fi
```

### 2. File Issues as You Discover Them

Don't let discovered work get lost:

```bash
# Discovered a bug while working on bd-5
NEW_ID=$(tracer create "Fix null pointer in parser" -t bug -p 0 --json | jq -r '.id')
tracer dep add $NEW_ID bd-5 --type discovered-from
```

### 3. Use Appropriate Dependency Types

- **blocks**: "bd-2 cannot start until bd-1 is done"
- **parent-child**: "bd-2 is a subtask of epic bd-1"
- **discovered-from**: "bd-2 was found while working on bd-1"
- **related**: "bd-2 and bd-1 are connected but don't block"

### 4. Update Status Regularly

Keep the tracker in sync with reality:

```bash
# Starting work
tracer update bd-1 --status in_progress

# Hit a blocker
tracer update bd-1 --status blocked

# Back to work
tracer update bd-1 --status in_progress

# Done
tracer close bd-1 --reason "Completed successfully"
```

### 5. Create Epics for Large Features

Break down complex work:

```bash
# Create epic
EPIC=$(tracer create "User authentication system" -t epic -p 0 --json | jq -r '.id')

# Create child tasks
T1=$(tracer create "Design auth schema" -t task -p 1 --json | jq -r '.id')
tracer dep add $T1 $EPIC --type parent-child

T2=$(tracer create "Implement login endpoint" -t task -p 1 --json | jq -r '.id')
tracer dep add $T2 $EPIC --type parent-child
tracer dep add $T2 $T1 --type blocks  # T2 blocked by T1

T3=$(tracer create "Add session management" -t task -p 1 --json | jq -r '.id')
tracer dep add $T3 $EPIC --type parent-child
tracer dep add $T3 $T1 --type blocks  # T3 blocked by T1
```

## Advanced Usage

### Query and Filter

```bash
# List all open bugs
tracer list --status open --type bug --json

# Find issues by priority
tracer list --priority 0 --json

# Search by assignee
tracer list --assignee "agent-1" --json
```

### Dependency Management

```bash
# View dependency tree
tracer dep tree bd-1 --max-depth 10

# Detect cycles (issues blocking each other)
tracer dep cycles

# Remove a dependency
tracer dep remove bd-2 bd-1
```

### Batch Operations

```bash
# Close multiple issues
tracer close bd-1 bd-2 bd-3 --reason "Sprint complete"

# Create issues with dependencies inline
tracer create "Task B" -p 1 --deps "blocks:bd-1"
```

## Integration Example

Complete agent workflow:

```bash
#!/bin/bash

# 1. Check for ready work
WORK=$(tracer ready --limit 1 --json)

if [ "$(echo $WORK | jq length)" -eq 0 ]; then
  echo "No ready work found"
  exit 0
fi

# 2. Get issue details
ISSUE_ID=$(echo $WORK | jq -r '.[0].id')
ISSUE_TITLE=$(echo $WORK | jq -r '.[0].title')

echo "Working on: $ISSUE_ID - $ISSUE_TITLE"

# 3. Claim the work
tracer update $ISSUE_ID --status in_progress

# 4. Do the work...
# (your implementation here)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Abil-Shrestha/tracer](https://github.com/Abil-Shrestha/tracer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
