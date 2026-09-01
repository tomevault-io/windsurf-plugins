---
trigger: always_on
description: This document contains critical information about working with this codebase. Follow these guidelines precisely.
---

# Overview
This document contains critical information about working with this codebase. Follow these guidelines precisely.

## Bash Tool Usage
- **NEVER include comments in bash commands.** Do not prefix commands with `# explanation` comments. Just run the command directly. Use the `description` parameter to explain what the command does instead.
- **For testing Python code**, create a temporary `.py` file (e.g., `test_something.py`) and run it with `python test_something.py`. Do NOT use heredoc syntax like `python3 << 'EOF'`.

## Image Analysis
When you need to analyze, describe, or extract information from an image file, use the `/describe-image` skill. This invokes a local vision LLM to provide detailed descriptions of image contents.

This project will use React for any GUI. All controls, buttons, etc. will use an event driven architecture, so when a button is pressed, the UI remains responsive while the process executes.

Standalone command line tools such as those that process or generate data files should include argument processing, and the usage should be documented in a readme file.

When writing a project plan, always create a TODO.md with the tasks and phases. Mark things as complete in the readme as the process runs.

Each time a major feature or phase is developed, make sure to thoroughly test it, then commit it to a local git repo (do not push).

## Overseer Integration

You have access to Overseer project management tools. Use them to stay focused and prevent scope drift.
## Required: Use Overseer MCP for Task Management

**You MUST use the Overseer MCP tools instead of the built-in TodoWrite tool for all task tracking in this project.**

### Before Starting Any Work

1. **Check active tasks first**: Call `mcp__overseer__read_active_tasks` to see what's currently being worked on
2. **Check for scope drift**: Call `mcp__overseer__check_drift` with the user's request to verify the work aligns with tracked tasks
   - If no match is found, ask the user if they want to create a new task before proceeding
   - If a weak match (40-80%), confirm which task this relates to

### During Work

3. **Create tasks for new work**: Use `mcp__overseer__create_task` to track new features, bugs, or chores
   - Always specify `type`: `feature`, `bug`, `debt`, or `chore`
   - Include `context` with implementation notes
   - Add `linked_files` for relevant source files

4. **Update task status**: Use `mcp__overseer__update_task_status` to mark tasks as:
   - `active` - currently being worked on
   - `blocked` - waiting on something
   - `done` - completed

### After Completing Work

5. **Log work sessions**: Call `mcp__overseer__log_work_session` with:
   - A brief `summary` of what was accomplished
   - The `task_id` if work was for a specific task
   - List of `files_touched` that were modified

### Jira Integration (if configured)

- Use `mcp__overseer__pull_jira_issues` to see assigned Jira issues
- Use `mcp__overseer__link_jira_issue` to connect local tasks to Jira
- Use `mcp__overseer__sync_jira_status` to push status updates to Jira
- Use `mcp__overseer__push_task_to_jira` to create a Jira issue from a local task

**Automatic Jira sync workflow:**
1. When you create a new task locally, ask the user if they want it pushed to Jira
2. When marking a task as `done`, automatically call `sync_jira_status` if the task has a `jira_key`
3. For significant features or bugs, proactively suggest pushing to Jira for team visibility


## Safety
When deleting files, make sure to stash or commit them to the repository first, so they can be recovered if something goes awry.

## Core Development Rules for Python

1. Package Management
   - Use pip, maintain an up to date requirements.txt file
   - Create a venv and a shell script to automatically make/start in it.

2. Code Quality
   - Type hints required for all code
   - Public APIs must have docstrings
   - Functions must be focused and small
   - Follow existing patterns exactly
   - Line length: 88 chars maximum

3. Testing Requirements
   - Framework: `uv run pytest`
   - Async testing: use anyio, not asyncio
   - Coverage: test edge cases and errors
   - New features require tests
   - Bug fixes require regression tests

4. Code Style
    - PEP 8 naming (snake_case for functions/variables)
    - Class names in PascalCase
    - Constants in UPPER_SNAKE_CASE
    - Document with docstrings
    - Use f-strings for formatting

- For commits fixing bugs or adding features based on user reports add:
  ```bash
  git commit --trailer "Reported-by:<name>"
  ```
  Where `<name>` is the name of the user.

- For commits related to a Github issue, add
  ```bash
  git commit --trailer "Github-Issue:#<number>"
  ```
- NEVER ever mention a `co-authored-by` or similar aspects. In particular, never
  mention the tool used to create the commit message or PR.

## Development Philosophy

- **Simplicity**: Write simple, straightforward code
- **Readability**: Make code easy to understand
- **Performance**: Consider performance without sacrificing readability
- **Maintainability**: Write code that's easy to update
- **Testability**: Ensure code is testable
- **Reusability**: Create reusable components and functions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kevinbentley/ronin-nvr](https://github.com/kevinbentley/ronin-nvr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
