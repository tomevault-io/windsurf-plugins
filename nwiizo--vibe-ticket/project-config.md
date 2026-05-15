---
trigger: always_on
description: A vibe-ticket managed project with spec-driven development support.
---

# vibe-ticket Project: vibe-ticket

A vibe-ticket managed project with spec-driven development support.

## Quick Reference

| Feature | Command | Slash Command | MCP Tool |
|---------|---------|---------------|----------|
| Kanban board view | `vibe-ticket board` | - | `vibe-ticket_board` |
| Review ticket | `vibe-ticket review <ticket>` | - | `vibe-ticket_review` |
| Approve ticket | `vibe-ticket approve <ticket>` | - | `vibe-ticket_approve` |
| Request changes | `vibe-ticket request-changes <ticket>` | - | `vibe-ticket_request_changes` |
| Handoff ticket | `vibe-ticket handoff <ticket> <assignee>` | - | `vibe-ticket_handoff` |
| Create spec from requirements | `vibe-ticket spec specify "..."` | `/specify "..."` | `vibe-ticket_spec_specify` |
| Generate plan | `vibe-ticket spec plan --tech-stack ...` | `/plan --tech-stack ...` | `vibe-ticket_spec_plan` |
| Create tasks | `vibe-ticket spec tasks --parallel` | `/tasks --parallel` | `vibe-ticket_spec_generate_tasks` |
| Validate spec | `vibe-ticket spec validate` | `/validate` | `vibe-ticket_spec_validate` |
| Bulk update | `vibe-ticket bulk update --status ...` | - | `vibe-ticket_bulk_update` |
| Bulk tag | `vibe-ticket bulk tag "tags" ...` | - | `vibe-ticket_bulk_tag` |
| Bulk close | `vibe-ticket bulk close ...` | - | `vibe-ticket_bulk_close` |
| Bulk archive | `vibe-ticket bulk archive ...` | - | `vibe-ticket_bulk_archive` |
| Create filter | `vibe-ticket filter create ...` | - | `vibe-ticket_filter_create` |
| Apply filter | `vibe-ticket filter apply <name>` | - | `vibe-ticket_filter_apply` |
| Create alias | `vibe-ticket alias create ...` | - | `vibe-ticket_alias_create` |
| Run alias | `vibe-ticket alias run <name>` | - | `vibe-ticket_alias_run` |
| Time log | `vibe-ticket time log <ticket> <duration>` | - | `vibe-ticket_time_log` |
| Time start/stop | `vibe-ticket time start/stop` | - | `vibe-ticket_time_start/stop` |
| Time report | `vibe-ticket time report` | - | `vibe-ticket_time_report` |
| Create hook | `vibe-ticket hook create ...` | - | `vibe-ticket_hook_create` |
| List hooks | `vibe-ticket hook list` | - | `vibe-ticket_hook_list` |
| Interactive select | `vibe-ticket interactive select` | - | - |
| Interactive multi | `vibe-ticket interactive multi` | - | - |

## Overview

This project uses vibe-ticket for ticket management with advanced spec-driven development capabilities. This document provides guidance for Claude Code when working with this codebase.

## Common vibe-ticket Commands

### Getting Started
```bash
# Create your first ticket
vibe-ticket new fix-bug --title "Fix login issue" --priority high

# List all tickets
vibe-ticket list

# Start working on a ticket (creates worktree by default)
vibe-ticket start fix-bug
# This creates: ./vibe-ticket-vibeticket-fix-bug/

# Start without worktree (branch only)
vibe-ticket start fix-bug --no-worktree

# Show current status
vibe-ticket check
```

### Working with Tickets
```bash
# Show ticket details
vibe-ticket show <ticket>

# Update ticket
vibe-ticket edit <ticket> --status review

# Add tasks to ticket
vibe-ticket task add "Write unit tests"
vibe-ticket task add "Update documentation"

# Complete tasks
vibe-ticket task complete 1

# Close ticket
vibe-ticket close <ticket> --message "Fixed the login issue"
```

### Review Workflow (NEW!)
```bash
# Mark ticket for review
vibe-ticket review <ticket> --notes "Ready for review"

# Approve ticket and mark as done
vibe-ticket approve <ticket> --message "Looks good!"

# Request changes on a ticket
vibe-ticket request-changes <ticket> --changes "Please add tests"

# Hand off ticket to another agent/person
vibe-ticket handoff <ticket> <assignee> --notes "Handing off to specialist"
```

### Kanban Board View (NEW!)
```bash
# Show tickets in kanban board format
vibe-ticket board

# Filter by assignee
vibe-ticket board --assignee "claude-code"

# Show only active tickets
vibe-ticket board --active-only

# Compact view
vibe-ticket board --compact
```

### Search and Filter
```bash
# Search tickets
vibe-ticket search "login"

# Filter by status
vibe-ticket list --status doing

# Filter by priority
vibe-ticket list --priority high
```

### Bulk Operations
```bash
# Update multiple tickets at once
vibe-ticket bulk update --status doing --priority high tag1,tag2

# Tag multiple tickets
vibe-ticket bulk tag "important,urgent" ticket1 ticket2 ticket3

# Close multiple tickets
vibe-ticket bulk close ticket1 ticket2 --message "Batch close"

# Archive old tickets
vibe-ticket bulk archive --before 2024-01-01
```

### Saved Filters
```bash
# Create a reusable filter
vibe-ticket filter create urgent-bugs --status todo --priority high --tags bug

# List saved filters
vibe-ticket filter list

# Apply a filter
vibe-ticket filter apply urgent-bugs

# Delete a filter
vibe-ticket filter delete urgent-bugs
```

### Custom Aliases
```bash
# Create command shortcuts
vibe-ticket alias create today "list --status doing"
vibe-ticket alias create urgent "list --priority high --priority critical"

# List all aliases
vibe-ticket alias list

# Run an alias
vibe-ticket alias run today

# Delete an alias
vibe-ticket alias delete today
```

### Time Tracking
```bash
# Start a timer for active work
vibe-ticket time start my-ticket

# Stop the current timer

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nwiizo/vibe-ticket](https://github.com/nwiizo/vibe-ticket) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
