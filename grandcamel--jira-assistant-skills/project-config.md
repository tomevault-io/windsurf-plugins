---
trigger: always_on
description: A modular, production-ready Claude Code skills framework for JIRA REST API automation.
---

# JIRA Assistant Skills

A modular, production-ready Claude Code skills framework for JIRA REST API automation.

## Project Overview

This project provides 14 specialized skills for interacting with JIRA via natural language:

| Skill | Purpose |
|-------|---------|
| `jira-assistant` | Hub/router with progressive disclosure |
| `jira-issue` | Core CRUD operations on issues |
| `jira-lifecycle` | Workflow/transition management |
| `jira-search` | JQL queries, saved filters, bulk operations |
| `jira-collaborate` | Comments, attachments, watchers |
| `jira-agile` | Epics, sprints, backlog, story points |
| `jira-relationships` | Issue linking, dependencies, cloning |
| `jira-time` | Time tracking, worklogs, estimates |
| `jira-jsm` | Jira Service Management |
| `jira-bulk` | Bulk operations with dry-run support |
| `jira-dev` | Git branch names, commit parsing, PR descriptions |
| `jira-fields` | Custom field management |
| `jira-ops` | Cache management, request batching |
| `jira-admin` | Project and permission administration |

## Architecture

### Directory Structure

```
.claude-plugin/
├── plugin.json                # Plugin manifest
└── marketplace.json           # Marketplace registry

.claude/
├── settings.example.json      # Example config (copy to settings.local.json)
└── settings.local.json        # Personal credentials (gitignored)

commands/                      # Slash commands (at project root)
config/                        # Configuration examples
skills/                        # Skills (autodiscovered at project root)
    ├── jira-assistant/        # Hub router
    ├── jira-issue/            # Issue CRUD
    ├── jira-lifecycle/        # Workflow transitions
    ├── jira-search/           # JQL queries
    ├── jira-collaborate/      # Comments, attachments
    ├── jira-agile/            # Sprints, boards
    ├── jira-relationships/    # Issue links
    ├── jira-time/             # Time tracking
    ├── jira-jsm/              # Service Management
    ├── jira-bulk/             # Bulk operations
    ├── jira-dev/              # Developer integration
    ├── jira-fields/           # Custom fields
    ├── jira-ops/              # Operations
    ├── jira-admin/            # Administration
    └── shared/                # Shared config and tests
        ├── config/
        ├── docs/
        └── tests/

jira-as/                       # PyPI library (source)
├── src/jira_as/
│   ├── cli/                   # CLI implementation
│   │   ├── main.py            # Entry point
│   │   ├── cli_utils.py       # Shared utilities
│   │   └── commands/          # Command groups
│   ├── mock/                  # Mock client (10 mixins)
│   └── *.py                   # Core modules
└── tests/                     # 952 unit tests
```

Note: All plugin components (commands/, skills/) are at project root.
Paths in `.claude-plugin/plugin.json` are relative to project root.

### Shared Library Pattern

All scripts import from the [jira-as](https://pypi.org/project/jira-as/) PyPI package:

```python
from jira_as import (
    get_jira_client,
    JiraError,
    ValidationError,
    validate_issue_key,
    format_issue,
)
```

### Shared Library Components

The `jira-as` package provides:

| Module | Purpose |
|--------|---------|
| `jira_client` | HTTP client with retry logic and session management |
| `config_manager` | Multi-source configuration |
| `error_handler` | Exception hierarchy |
| `validators` | Input validation (issue keys, JQL, URLs) |
| `formatters` | Output formatting (tables, JSON, CSV) |
| `adf_helper` | Atlassian Document Format conversion |
| `time_utils` | JIRA time format parsing |
| `cache` | SQLite-based caching with TTL |
| `credential_manager` | Keychain integration |
| `transition_helpers` | Fuzzy workflow matching |
| `user_helpers` | User resolution to account IDs |
| `jsm_utils` | Service Management SLA utilities |

### CLI Entry Point

The project provides a unified CLI via the `jira-as` command:

```bash
# Install in development mode
pip install -e jira-as/

# Verify installation
jira-as --version
```

#### CLI Usage

```bash
# Get help
jira-as --help
jira-as issue --help

# Issue operations
jira-as issue get PROJ-123
jira-as issue create PROJ --type Bug --summary "Fix login"
jira-as issue update PROJ-123 --priority High
jira-as issue delete PROJ-123

# Search and JQL
jira-as search query "project = PROJ AND status = Open"
jira-as search query "assignee = currentUser()" --max-results 50

# Workflow transitions
jira-as lifecycle transitions PROJ-123
jira-as lifecycle transition PROJ-123 "In Progress"

# Agile operations
jira-as agile sprints --board 1
jira-as agile move-to-sprint PROJ-123 --sprint 42

# Time tracking
jira-as time log PROJ-123 --time "2h 30m" --comment "Code review"
jira-as time worklogs PROJ-123

# Bulk operations
jira-as bulk transition "project = PROJ AND status = Open" "In Progress" --dry-run

# Administration
jira-as admin projects
jira-as admin users --project PROJ
```

#### Available Command Groups

| Group | Description |
|-------|-------------|
| `issue` | Issue CRUD operations |
| `search` | JQL queries and saved filters |
| `lifecycle` | Workflow transitions and status changes |
| `agile` | Sprint, board, and backlog management |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [grandcamel/JIRA-Assistant-Skills](https://github.com/grandcamel/JIRA-Assistant-Skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
