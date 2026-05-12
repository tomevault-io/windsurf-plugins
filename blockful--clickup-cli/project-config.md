---
trigger: always_on
description: A Go CLI wrapping the **complete ClickUp API** (134/135 endpoints, 99.3% coverage). All output is JSON. No interactive prompts. Designed for AI agents.
---

# AGENTS.md — AI Agent Guide for clickup-cli

## What This Is

A Go CLI wrapping the **complete ClickUp API** (134/135 endpoints, 99.3% coverage). All output is JSON. No interactive prompts. Designed for AI agents.

## Setup

```bash
# Authenticate (one-time, saves to ~/.clickup-cli.yaml)
clickup auth login --token pk_YOUR_TOKEN

# Verify
clickup auth whoami
```

## Command Pattern

```
clickup <resource> <verb> [flags]
```

All flags are `--kebab-case`. Required flags are noted in `--help`. IDs are always strings.

## Output Format

**Success** — raw JSON from ClickUp API:
```json
{"id": "abc123", "name": "My Task", "status": {"status": "open"}, ...}
```

**Error** — structured:
```json
{"error": "task not found", "code": "NOT_FOUND", "status": 404}
```

**Exit codes**: 0 = success, non-zero = error. Always parse stdout for the JSON response.

Use `--format text` only for human debugging. Default JSON is what you should parse.

---

## Complete Command Reference

### Workspace Discovery

```bash
clickup workspace list                          # → [{"id":"1234","name":"My Workspace",...}]
clickup workspace plan --workspace 1234         # Get plan details
clickup workspace seats --workspace 1234        # Get seat usage
clickup space list --workspace 1234             # List spaces
clickup space get --id 5678                     # Get single space
clickup folder list --space 5678                # List folders in space
clickup folder get --id 9012                    # Get single folder
clickup list list --folder 9012                 # Lists in folder
clickup list list --space 5678                  # Folderless lists in space
clickup list get --id 456                       # Get single list
```

### Task CRUD

```bash
# List tasks in a list
clickup task list --list 123 --include-closed --subtasks --page 0

# Get single task
clickup task get --id abc123
clickup task get --id abc123 --include-markdown --include-subtasks

# Create task
clickup task create --list 123 --name "Ship feature" \
  --assignee 456 --assignee 789 \
  --priority 2 \
  --status "in progress" \
  --due-date 1700000000000 \
  --tag "urgent" --tag "frontend" \
  --markdown-content "## Description\nRich **markdown** here"

# Update task
clickup task update --id abc123 \
  --name "New name" \
  --status "done" \
  --assignees-add 789 \
  --assignees-rem 456 \
  --priority 1

# Delete task
clickup task delete --id abc123

# Search across workspace
clickup task search --workspace 1234 \
  --assignee 567 \
  --status "in progress" \
  --tag "urgent" \
  --include-closed \
  --include-markdown
```

### Task Advanced Operations

```bash
# Add/remove task from additional lists
clickup task add-to-list --id abc123 --list 456
clickup task remove-from-list --id abc123 --list 456

# Merge tasks
clickup task merge --id abc123 --merge-with def456 --merge-with ghi789

# Time in status
clickup task time-in-status --id abc123
clickup task time-in-status --task-ids abc123 --task-ids def456  # bulk

# Dependencies
clickup task dependency add --task abc123 --depends-on def456
clickup task dependency add --task abc123 --dependency-of ghi789
clickup task dependency remove --task abc123 --depends-on def456

# Links
clickup task link add --task abc123 --links-to def456
clickup task link remove --task abc123 --links-to def456
```

### Custom Task IDs

When your workspace uses custom task IDs (e.g., `PROJ-123`), add `--custom-task-ids` and `--team-id`:

```bash
clickup task get --id "PROJ-123" --custom-task-ids --team-id 1234567
clickup task update --id "PROJ-123" --custom-task-ids --team-id 1234567 --status "done"
clickup comment create --task "PROJ-123" --custom-task-ids --team-id 1234567 --text "Done!"
```

This pattern works on: `task get/update/delete`, `task add-to-list/remove-from-list/merge/time-in-status`, `task dependency/link`, `comment create`, `custom-field set/remove`, `attachment create`, `guest add-to-task/remove-from-task`, `time-entry legacy` commands.

### Comments

```bash
# Create comment on task
clickup comment create --task abc123 --text "Status update: done"

# Create comment on list or view
clickup comment create --list 456 --text "List-level note"
clickup comment create --view-id viewid --text "View comment"

# List comments
clickup comment list --task abc123
clickup comment list --list 456

# Update/delete
clickup comment update --id 789 --text "Updated text"
clickup comment delete --id 789

# Threaded replies
clickup comment reply list --comment-id 789
clickup comment reply create --comment-id 789 --text "Reply text"
```

### Custom Fields

```bash
# Discover fields (scoped to list, folder, space, or workspace)
clickup custom-field list --list 123
clickup custom-field list --space 456
clickup custom-field list --workspace 1234

# Set a field value
clickup custom-field set --task abc123 --field field-uuid --value '"some string"'
clickup custom-field set --task abc123 --field field-uuid --value '123'
clickup custom-field set --task abc123 --field field-uuid --value '{"option_id": "uuid"}'

# Remove a field value
clickup custom-field remove --task abc123 --field field-uuid
```

### Tags

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [blockful/clickup-cli](https://github.com/blockful/clickup-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-11 -->
