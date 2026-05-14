---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Code Conductor is an AI agent coordination system designed to orchestrate multiple AI coding agents (like Claude Code, Conductor, Warp) working on the same codebase. It provides GitHub-native task management with automatic conflict prevention through git worktrees.

**IMPORTANT**: This is a template repository. When you encounter Code Conductor files in a project, they have been imported to enable automated agent coordination. Work autonomously based on GitHub Issues with the `conductor:task` label and the guidance in these files.

## Documentation Map

**CRITICAL**: When starting work on any task, check if `.conductor/documentation-map.yaml` exists. This file contains:
- Comprehensive project analysis and structure
- Technology stack details and dependencies  
- List of completed vs. pending features
- Implementation status and critical paths
- Architectural decisions and constraints

If this file exists, load it to understand the project context before beginning work. This map is created by the `[INIT]` discovery task and provides essential context for all subsequent tasks.

## Key Development Commands

### Running Tests
```bash
# Run all tests
python -m pytest tests/ -v

# Run specific test file
python tests/test_basic.py
python tests/test_stack_detection.py
```

### Linting and Formatting
```bash
# Check code formatting (without making changes)
black --check .conductor/scripts/ setup.py

# Apply formatting
black .conductor/scripts/ setup.py

# Run linting
flake8 .conductor/scripts/ setup.py --max-line-length=88 --extend-ignore=E203,W503
```

### Validation Commands
```bash
# Validate conductor configuration
python .conductor/scripts/validate-config.py

# Check system dependencies
python .conductor/scripts/dependency-check.py

# Run health check
python .conductor/scripts/health-check.py
```

## Architecture Overview

### Core Components

1. **Setup System** (`setup.py`)
   - Interactive/auto configuration wizard
   - Detects technology stack automatically
   - Configures agent roles based on project type
   - Creates GitHub workflows for automation

2. **Task Management** (GitHub Issues)
   - GitHub Issues with `conductor:task` label serve as tasks
   - Issues have unique numbers, descriptions, success criteria
   - GitHub's atomic operations prevent race conditions
   - Native integration with GitHub Projects and Actions

3. **Agent Roles** (`.conductor/roles/`)
   - `dev.md` - Default generalist role for most tasks
   - Specialized roles: `devops`, `security`, `frontend`, `mobile`, `ml-engineer`, `data`
   - `code-reviewer` - AI-powered PR reviews (always included)
   - Hybrid model: prefer `dev` role unless task requires specialization

4. **Agent Coordination** (`.conductor/scripts/`)
   - `conductor` - Universal agent command (primary interface)
   - `task-claim.py` - Task assignment via GitHub Issue assignment
   - `health-check.py` - Monitor agent heartbeats
   - `cleanup-stale.py` - Remove abandoned work
   - Git worktrees provide isolation between agents

5. **GitHub Integration**
   - Issues become tasks via `conductor:task` label
   - Actions run health checks every 15 minutes
   - AI code reviews on all PRs
   - Status dashboard via `conductor:status` issue

### Key Design Patterns

1. **Atomic Operations**: GitHub's issue assignment API ensures atomic task claiming
2. **Worktree Isolation**: Each agent works in separate git worktree (`worktrees/agent-{role}-{task_id}`)
3. **Heartbeat System**: Agents update timestamps; stale work auto-cleaned after timeout
4. **File Conflict Prevention**: Worktree isolation ensures agents work on separate branches
5. **Self-Healing**: GitHub Actions monitor health, clean stale work, process issues

### Configuration Structure

```yaml
# .conductor/config.yaml
project_name: string
documentation: 
  main: string (path to main docs)
  additional: [array of paths]
technology_stack:
  languages: [detected languages]
  frameworks: [detected frameworks]
  tools: [detected build tools]
roles:
  default: "dev"
  specialized: [list of specialized roles]
github_integration:
  enabled: boolean
  issue_to_task: boolean
  pr_reviews: boolean
worktree_retention_days: number (default 7)
```

## Development Workflow

When modifying code-conductor itself:

1. Make changes in appropriate files:
   - Core scripts: `.conductor/scripts/`
   - Role definitions: `.conductor/roles/`
   - Setup logic: `setup.py`

2. Run validation after changes:
   ```bash
   python .conductor/scripts/validate-config.py
   black .conductor/scripts/ setup.py
   flake8 .conductor/scripts/ setup.py --max-line-length=88
   python -m pytest tests/ -v
   ```

3. Test setup flow:
   ```bash
   # Create test environment
   mkdir /tmp/test-conductor && cd /tmp/test-conductor
   git init
   # Copy conductor files and run setup
   python setup.py --auto
   ```

<!-- CONDUCTOR:START -->
## 🤖 AI Agent Quick Start

**IMPORTANT**: Always run from the project root directory where Code Conductor is configured!

### 🚀 The Universal Kickoff Prompt

Start every Code Conductor session with:

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ryanmac/code-conductor](https://github.com/ryanmac/code-conductor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
