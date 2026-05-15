---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

@doc/glossary.md

## Project Overview

An MCP (Model Context Protocol) server that provides task-based workflow management for agents. Tasks are organized into configurable categories, each with its own execution instructions.

## Documentation Structure

**Documentation Policy:**

- **`doc/*.md` files**: Should be definitive and exhaustive, covering all details, edge cases, configuration options, and troubleshooting for their respective topics
- **`README.md`**: Should address only the most common use cases and provide clear links to the exhaustive documentation in `doc/*.md` files

When updating documentation:
- Add comprehensive details to `doc/*.md` files
- Keep `README.md` concise, covering quick start and common scenarios
- Ensure `README.md` links to relevant `doc/*.md` files for deeper coverage
- Avoid duplicating detailed content between README.md and doc files

## Architecture

**Task Storage Structure:**
- `.mcp-tasks/tasks.ednl` - All incomplete tasks stored as EDN records
- `.mcp-tasks/complete.ednl` - All completed tasks archive
- `.mcp-tasks/category-prompts/<category-name>.md` - Category-specific execution prompts
- `.mcp-tasks/prompt-overrides/<name>.md` - Custom workflow prompt overrides (optional)

**Prompt Templating:**

Prompts support Selmer-based templating to reduce duplication:
- `{% include "infrastructure/file.md" %}` - Include reusable fragments from `resources/prompts/infrastructure/`
- `{{variable}}` - Variable substitution (used internally)
- Override files in `.mcp-tasks/prompt-overrides/` take precedence over built-in prompts

The `mcp-tasks.templates` namespace wraps Selmer to provide include resolution with proper override handling. See `doc/customization.md` for complete templating syntax and usage documentation.

**Task File Format:**
Tasks are stored in EDNL (EDN Lines) format where each line is a valid EDN map representing a Task record. The Task schema (defined in `src/mcp_tasks/schema.clj`) includes:

```clojure
{:id              ;; int - unique task identifier
 :parent-id       ;; int or nil - optional parent task reference
 :status          ;; :open | :in-progress | :done | :closed | :blocked
 :title           ;; string - task title
 :description     ;; string - detailed task description
 :design          ;; string - design notes
 :category        ;; string - execution category (simple, medium, large, etc.)
 :type            ;; :task | :bug | :feature | :story | :chore
 :meta            ;; map - arbitrary string key-value metadata
 :relations       ;; vector of Relation maps
 :shared-context  ;; vector of strings (optional) - inter-task communication
 :session-events  ;; vector of SessionEvent maps (optional) - captured interactions
}
```

**Relation Schema:**
Task relationships are defined as:

```clojure
{:id         ;; int - relation identifier
 :relates-to ;; int - target task ID
 :as-type    ;; :blocked-by | :related | :discovered-during
}
```

**Dependencies:**
- Depends on local `mcp-clj` server library at `../mcp-clj/projects/server`
- Ensure the sibling mcp-clj repository is available for development

## Task Dependencies and Blocking

The system supports task dependencies through `:blocked-by` relationships, enabling proper task ordering and dependency management.

**Blocking Logic:**

A task is **blocked** if ANY of its `:blocked-by` relations reference an incomplete task (status `:open`, `:in-progress`, or `:blocked`). A task is **unblocked** if ALL of its `:blocked-by` relations reference completed tasks (status `:closed`, `:done`, or `:deleted`), or if it has no `:blocked-by` relations. Note that `:done` tasks do NOT block dependent tasks—they have finished implementation and are awaiting merge.

**Status Field vs. blocked-by Relations:**

The system distinguishes between two types of blocking:

- **`:status :blocked`** - Manual blocking for external factors (e.g., waiting on stakeholder decision, external dependency). This is a user-set status indicating the task cannot proceed for reasons outside the task system.

- **`:blocked-by` relations** - Automated dependency-based blocking. The system computes whether a task is blocked by checking the completion status of tasks referenced in its `:blocked-by` relations.

A task can be both manually blocked (`:status :blocked`) and dependency-blocked (has incomplete `:blocked-by` relations). When computing dependency blocking, tasks with `:status :blocked` are treated as incomplete, meaning they can block other tasks that depend on them.

**Creating Dependencies:**

When creating tasks with dependencies:
1. Create all tasks first using `add-task`
2. Use `update-task` to add `:blocked-by` relations after tasks exist
3. Example: Task B depends on Task A → add relation `{:id 1, :relates-to <task-a-id>, :as-type :blocked-by}` to Task B's `:relations` vector

**Querying Blocked Status:**

The `select-tasks` tool automatically computes blocking status for each returned task:
- `:is-blocked` - Boolean indicating if the task is currently blocked
- `:blocking-task-ids` - Vector of task IDs that are blocking this task (empty if unblocked)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hugoduncan/mcp-tasks](https://github.com/hugoduncan/mcp-tasks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
