---
trigger: always_on
description: Guide for using Task Master to manage task-driven development workflows
---

# Task Master Development Workflow

This guide outlines the typical process for using Task Master to manage software development projects.

## Primary Interaction: MCP Server vs. CLI

Task Master offers two primary ways to interact:

1.  **MCP Server (Recommended for Integrated Tools)**:
    - For AI agents and integrated development environments (like Cursor), interacting via the **MCP server is the preferred method**.
    - The MCP server exposes Task Master functionality through a set of tools (e.g., `get_tasks`, `add_subtask`).
    - This method offers better performance, structured data exchange, and richer error handling compared to CLI parsing.
    - Refer to [`mcp.mdc`](mdc:.cursor/rules/mcp.mdc) for details on the MCP architecture and available tools.
    - A comprehensive list and description of MCP tools and their corresponding CLI commands can be found in [`taskmaster.mdc`](mdc:.cursor/rules/taskmaster.mdc).
    - **Restart the MCP server** if core logic in `scripts/modules` or MCP tool/direct function definitions change.

2.  **`task-master` CLI (For Users & Fallback)**:
    - The global `task-master` command provides a user-friendly interface for direct terminal interaction.
    - It can also serve as a fallback if the MCP server is inaccessible or a specific function isn't exposed via MCP.
    - Install globally with `npm install -g task-master-ai` or use locally via `npx task-master-ai ...`.
    - The CLI commands often mirror the MCP tools (e.g., `task-master list` corresponds to `get_tasks`).
    - Refer to [`taskmaster.mdc`](mdc:.cursor/rules/taskmaster.mdc) for a detailed command reference.

## Standard Development Workflow Process

-   Start new projects by running `initialize_project` tool / `task-master init` or `parse_prd` / `task-master parse-prd --input='<prd-file.txt>'` (see [`taskmaster.mdc`](mdc:.cursor/rules/taskmaster.mdc)) to generate initial tasks.json
-   Begin coding sessions with `get_tasks` / `task-master list` (see [`taskmaster.mdc`](mdc:.cursor/rules/taskmaster.mdc)) to see current tasks, status, and IDs
-   Determine the next task to work on using `next_task` / `task-master next` (see [`taskmaster.mdc`](mdc:.cursor/rules/taskmaster.mdc)).
-   Analyze task complexity with `analyze_project_complexity` / `task-master analyze-complexity --research` (see [`taskmaster.mdc`](mdc:.cursor/rules/taskmaster.mdc)) before breaking down tasks
-   Review complexity report using `complexity_report` / `task-master complexity-report` (see [`taskmaster.mdc`](mdc:.cursor/rules/taskmaster.mdc)).
-   Select tasks based on dependencies (all marked 'done'), priority level, and ID order
-   Clarify tasks by checking task files in tasks/ directory or asking for user input
-   View specific task details using `get_task` / `task-master show <id>` (see [`taskmaster.mdc`](mdc:.cursor/rules/taskmaster.mdc)) to understand implementation requirements
-   Break down complex tasks using `expand_task` / `task-master expand --id=<id> --force --research` (see [`taskmaster.mdc`](mdc:.cursor/rules/taskmaster.mdc)) with appropriate flags like `--force` (to replace existing subtasks) and `--research`.
-   Clear existing subtasks if needed using `clear_subtasks` / `task-master clear-subtasks --id=<id>` (see [`taskmaster.mdc`](mdc:.cursor/rules/taskmaster.mdc)) before regenerating
-   Implement code following task details, dependencies, and project standards
-   Verify tasks according to test strategies before marking as complete (See [`tests.mdc`](mdc:.cursor/rules/tests.mdc))
-   Mark completed tasks with `set_task_status` / `task-master set-status --id=<id> --status=done` (see [`taskmaster.mdc`](mdc:.cursor/rules/taskmaster.mdc))
-   Update dependent tasks when implementation differs from original plan using `update` / `task-master update --from=<id> --prompt="..."` or `update_task` / `task-master update-task --id=<id> --prompt="..."` (see [`taskmaster.mdc`](mdc:.cursor/rules/taskmaster.mdc))
-   Add new tasks discovered during implementation using `add_task` / `task-master add-task --prompt="..." --research` (see [`taskmaster.mdc`](mdc:.cursor/rules/taskmaster.mdc)).
-   Add new subtasks as needed using `add_subtask` / `task-master add-subtask --parent=<id> --title="..."` (see [`taskmaster.mdc`](mdc:.cursor/rules/taskmaster.mdc)).
-   Append notes or details to subtasks using `update_subtask` / `task-master update-subtask --id=<subtaskId> --prompt='Add implementation notes here...\nMore details...'` (see [`taskmaster.mdc`](mdc:.cursor/rules/taskmaster.mdc)).
-   Generate task files with `generate` / `task-master generate` (see [`taskmaster.mdc`](mdc:.cursor/rules/taskmaster.mdc)) after updating tasks.json
-   Maintain valid dependency structure with `add_dependency`/`remove_dependency` tools or `task-master add-dependency`/`remove-dependency` commands, `validate_dependencies` / `task-master validate-dependencies`, and `fix_dependencies` / `task-master fix-dependencies` (see [`taskmaster.mdc`](mdc:.cursor/rules/taskmaster.mdc)) when needed
-   Respect dependency chains and task priorities when selecting work
-   Report progress regularly using `get_tasks` / `task-master list`

## Task Complexity Analysis


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/6531503070) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
