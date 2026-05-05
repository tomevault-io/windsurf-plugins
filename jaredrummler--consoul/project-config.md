---
trigger: always_on
description: This guide helps Gemini AI agents understand how to use Gira for project management and ticket tracking in the **Consoul** project. Gemini's large context window makes it ideal for analyzing project-wide patterns and managing complex tasks.
---

# GEMINI.md - Gira Usage Guide for Gemini

This guide helps Gemini AI agents understand how to use Gira for project management and ticket tracking in the **Consoul** project. Gemini's large context window makes it ideal for analyzing project-wide patterns and managing complex tasks.

## 🎯 What is Gira?

Gira is a Git-native project management tool that stores all project data as JSON files in `.gira/`, enabling version control for project management data and seamless AI collaboration.

## 🚦 Workflow States Quick Reference

**IMPORTANT**: Use these exact status names to avoid errors:

| Status | Description | Usage |
|--------|-------------|-------|
| `todo` | To Do tickets | Default status for new tickets |
| `in_progress` | In Progress tickets | Maximum 5 tickets (WIP limit) |
| `review` | Review tickets | Maximum 5 tickets (WIP limit) |
| `done` | Done tickets | Final status |

⚠️ **Common Status Names**:
The valid statuses for this project are: `todo`, `in_progress`, `review`, `done`.
Use `gira status list` to see all available statuses.


## 🔍 Leveraging Gemini's Strengths with Gira

### Large-Scale Project Analysis

Gemini's large context window is perfect for analyzing entire project structures and ticket relationships:

#### 1. Project-Wide Ticket Analysis
```bash
# Export all tickets for comprehensive analysis
gira export tickets --format json > all-tickets.json

# View all tickets with their relationships
gira ticket list --format json | jq '.'

# Analyze ticket patterns and dependencies
gira epic list --show-progress
```

#### 2. Batch Ticket Operations
```bash
# Process multiple tickets based on criteria
gira ticket list --status backlog --priority high --format json | \
  jq -r '.[].id' | while read -r ticket; do
    gira ticket show "$ticket"
  done

# Find related tickets across epics
for epic in $(gira epic list --format json | jq -r '.[].id'); do
    echo "Epic: $epic"
    gira epic show "$epic"
done
```

#### 3. Comprehensive Progress Analysis
```bash
# Generate project statistics
gira ticket list --format json | \
  jq '[.[] | select(.status == "done")] | length' # Count completed tickets

# Analyze velocity across sprints
gira sprint list --format json | \
  jq '.[] | {name: .name, completed: .completed_points}'
```

## 🤖 Agent-Friendly Commands for Gemini

### Non-Interactive Operations
```bash
# Skip confirmations with --force flag
gira ticket move TICKET-ID in_progress --force
gira ticket move TICKET-ID done --force

# Use echo for confirmation prompts
echo "y" | gira ticket move TICKET-ID in_progress

# Batch operations with JSON processing
gira ticket list --status backlog --format json | \
  jq -r '.[].id' | head -5 | \
  while read ticket; do echo "y" | gira ticket move "$ticket" ready; done
```

### Complete Workflow Examples
```bash
# Basic bug fix workflow
gira ticket create "Fix authentication issue" --type bug --priority high --status backlog
gira ticket move SOUL-1 in_progress --force
gira comment add SOUL-1 -c "Investigated root cause"
gira ticket move SOUL-1 done --force

# Epic creation and management
gira epic create "User Management" --description "Complete user system"
gira ticket create "User registration" --type feature --epic EPIC-001 --status backlog
gira ticket create "Password reset" --type feature --epic EPIC-001 --status backlog
gira epic show EPIC-001
```

## 📋 Essential Gira Commands for Gemini

### Viewing Project Information
```bash
# View the kanban board with all tickets
gira board

# List all tickets with detailed information
gira ticket list

# View epics and their progress
gira epic list --show-progress

# Check sprint status and velocity
gira sprint list
```

### Managing Complex Tasks
```bash
# View all tickets in an epic
gira epic show EPIC-001

# Create subtasks for large features
gira ticket create "Implement component A" --type task --epic EPIC-001
gira ticket create "Implement component B" --type task --epic EPIC-001

# Batch update tickets
for ticket in $(gira ticket list --status backlog --assignee me --format json | jq -r '.[].id'); do
    echo "y" | gira ticket move "$ticket" in_progress
done
```

## 🚀 Gemini-Specific Workflows

### 1. Multi-Ticket Analysis Workflow
```bash
# Export tickets for analysis
gira export tickets --format json > tickets.json

# Analyze patterns in ticket data
gira ticket list --format json | \
  jq 'group_by(.type) | map({type: .[0].type, count: length})'

# Find tickets without epics
gira ticket list --format json | \
  jq '.[] | select(.epic == null) | .id'
```

### 2. Progress Tracking Across Epics
```bash
# Generate epic progress report
for epic in $(gira epic list --format json | jq -r '.[].id'); do
    echo "Epic: $epic"
    gira epic show "$epic" | grep -E "(Progress|Status)"
done

# Track sprint velocity
gira sprint list --format json | \
  jq 'map({sprint: .name, velocity: .completed_points})'
```

### 3. Dependency Analysis
```bash
# Find blocked tickets
gira ticket list --status blocked

# Analyze ticket relationships
gira ticket list --format json | \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jaredrummler/consoul](https://github.com/jaredrummler/consoul) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
