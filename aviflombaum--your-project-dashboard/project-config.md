---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Project Dashboard is a Rails 8.2 application that automatically discovers, analyzes, and tracks all active git repositories in your local development environment. It provides a centralized view via a CLI interface using Rake tasks, extracting metadata about tech stack, deployment status, and project state.

**Architecture:** SQLite database + ActiveRecord models + Plain Ruby classes (`lib/`) + Rake tasks interface. No web UI yet (planned for v1.1).

## Tech Stack

- **Ruby:** 3.4+
- **Rails:** 8.2 (main branch)
- **Database:** SQLite3
- **Frontend (future):** Tailwind 4, Hotwire (Turbo + Stimulus)
- **Background Jobs:** Solid Queue
- **Cache:** Solid Cache
- **Asset Pipeline:** Propshaft + Importmap

## Essential Commands

### Setup
```bash
bin/setup                    # Initial setup
bin/rails db:migrate         # Run migrations
```

### Scanning Projects
```bash
bin/rake projects:scan                           # Scan and save to DB
DRY_RUN=true bin/rake projects:scan             # Preview without saving
SCAN_ROOT_PATH=/custom/path bin/rake projects:scan  # Custom directory
SCAN_CUTOFF_DAYS=180 bin/rake projects:scan     # 6 months instead of 8
bin/rake projects:config                         # Show configuration
```

### Rails Console
```bash
bin/rails console           # Access models and query data
# Example queries:
# Project.order(last_commit_date: :desc).first(10)
# Project.where("metadata ->> 'inferred_type' = ?", "rails-app")
```

### Testing
```bash
bin/rails test              # Run test suite
bin/rails test:system       # Run system tests
```

### Code Quality
```bash
bin/rubocop                 # Ruby linting (omakase style)
bin/brakeman                # Security scanning
bundle audit                # Check for gem vulnerabilities
```

### Development Server
```bash
bin/dev                     # Start Rails server + Tailwind watch
bin/rails server            # Rails server only
```

## Core Architecture

### Data Flow
```
Rake Task (lib/tasks/projects.rake)
    ↓
ProjectScanner (lib/project_scanner.rb)
    ├─ Finds all .git directories recursively
    ├─ Filters by commit date (default: 8 months)
    └─ For each repo:
        ↓
    ProjectData (lib/project_data.rb)
        ├─ Extracts git metadata (commits, authors, dates)
        ├─ Detects tech stack (Gemfile, package.json, etc.)
        ├─ Infers project type (rails-app, node-app, etc.)
        ├─ Finds reference files (README, CLAUDE.md, TODO.md)
        ├─ Assesses current state (active, paused, WIP)
        └─ Returns structured metadata
        ↓
    Project Model (app/models/project.rb)
        └─ Upserts to SQLite (find_or_initialize_by path)
```

### Key Components

**`lib/project_scanner.rb`**
- Recursive directory traversal to find .git repos
- Filters repos by commit recency
- Skips irrelevant directories (node_modules, hidden dirs)
- Orchestrates scanning loop and database persistence
- Prints progress indicators and summary reports

**`lib/project_data.rb`**
- Extracts all metadata for a single repository
- Runs git commands safely (captures stderr)
- Detects tech stack by checking signature files (Gemfile, package.json, etc.)
- Infers project type, deployment status, and current state
- Parses documentation files for descriptions
- Graceful error handling per extraction step

**`app/models/project.rb`**
- ActiveRecord model with validations
- Upsert logic via `create_or_update_from_data(project_data)`
- JSON metadata column for flexible schema

**Database Schema:**
```ruby
create_table :projects do |t|
  t.string :path              # Unique, indexed (repository path)
  t.string :name              # Project name (directory basename)
  t.string :last_commit_date  # Indexed for sorting
  t.text :last_commit_message
  t.json :metadata            # All extracted data (tech_stack, commits, etc.)
  t.timestamps
end
```

### Metadata Structure

The `metadata` JSON column stores:
- `last_commit_author` - Most recent committer
- `recent_commits` - Array of last 10 commits (date + message)
- `commit_count_8m` - Commits in last 8 months
- `contributors` - All unique authors
- `reference_files` - Hash of docs by category (root, ai, cursor, docs)
- `description` - Extracted from README/CLAUDE.md
- `current_state` - "active (committed 2 days ago), 3 open tasks"
- `tech_stack` - Array: ["ruby", "rails", "node"]
- `inferred_type` - rails-app | node-app | python-app | docs | script | unknown
- `deployment_status` - Indicators from Dockerfile, deploy scripts, etc.
- `nested_repos` - Subdirectories with their own .git
- `errors` - Array of non-fatal errors during extraction

## Configuration

### Environment Variables
- `SCAN_ROOT_PATH` - Root directory to scan (default: `~/Development`)
- `SCAN_CUTOFF_DAYS` - Skip projects older than N days (default: `240` = 8 months)
- `DRY_RUN` - Set to `"true"` to scan without saving (default: `false`)

### Autoloading
- `lib/` directory is autoloaded (except `tasks/`)
- `ProjectScanner` and `ProjectData` are available in rake tasks and console

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aviflombaum/your-project-dashboard](https://github.com/aviflombaum/your-project-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
