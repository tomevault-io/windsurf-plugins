---
trigger: always_on
description: This guide outlines the typical process for using the Task Orchestrator to manage software development projects. The Task Orchestrator is defined as the mechanism for recording tasks in TODO.md, marking them off with [x], and updating the file with new tasks to complete.
---


# Task Orchestrator Development Workflow

This guide outlines the typical process for using the Task Orchestrator to manage software development projects. The Task Orchestrator is defined as the mechanism for recording tasks in TODO.md, marking them off with [x], and updating the file with new tasks to complete.

Note: TODO is pronounced "TO-DO" ;-P

## Primary Interaction: MCP Server vs. CLI

Task Orchestrator offers two primary ways to interact:

1.  **MCP Server (Recommended for Integrated Tools)**:
    - For AI agents and integrated development environments (like Cursor), interacting via the **MCP server is the preferred method**.
    - The MCP server exposes Task Orchestrator functionality through a set of tools (e.g., `get_tasks`, `add_subtask`).
    - This method offers better performance, structured data exchange, and richer error handling compared to CLI parsing.
    - Refer to [`mcp.mdc`](mdc:.cursor/rules/mcp.mdc) for details on the MCP architecture and available tools.
    - A comprehensive list and description of MCP tools and their corresponding CLI commands can be found in [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc).
    - **Restart the MCP server** if core logic in `scripts/modules` or MCP tool/direct function definitions change.

2.  **`task-orchestrator` CLI (For Users & Fallback)**:
    - The global `task-orchestrator` command provides a user-friendly interface for direct terminal interaction.
    - It can also serve as a fallback if the MCP server is inaccessible or a specific function isn't exposed via MCP.
    - Install globally with `npm install -g task-orchestrator-ai` or use locally via `npx task-orchestrator-ai ...`.
    - The CLI commands often mirror the MCP tools (e.g., `task-orchestrator list` corresponds to `get_tasks`).
    - Refer to [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc) for a detailed command reference.

## Standard Development Workflow Process

-   Start new projects by running `init` tool / `task-orchestrator init` or `parse_prd` / `task-orchestrator parse-prd --input='<prd-file.txt>'` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)) to generate initial tasks.json
-   Begin coding sessions with `get_tasks` / `task-orchestrator list` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)) to see current tasks, status, and IDs
-   Determine the next task to work on using `next_task` / `task-orchestrator next` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)).
-   Analyze task complexity with `analyze_complexity` / `task-orchestrator analyze-complexity --research` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)) before breaking down tasks
-   Review complexity report using `complexity_report` / `task-orchestrator complexity-report` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)).
-   Select tasks based on dependencies (all marked 'done'), priority level, and ID order
-   Clarify tasks by checking task files in tasks/ directory or asking for user input
-   View specific task details using `get_task` / `task-orchestrator show <id>` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)) to understand implementation requirements
-   Break down complex tasks using `expand_task` / `task-orchestrator expand --id=<id>` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)) with appropriate flags
-   Clear existing subtasks if needed using `clear_subtasks` / `task-orchestrator clear-subtasks --id=<id>` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)) before regenerating
-   Implement code following task details, dependencies, and project standards
-   Verify tasks according to test strategies before marking as complete (See [`tests.mdc`](mdc:.cursor/rules/tests.mdc))
-   **REQUIRED**: Commit all changes to git before marking tasks as complete (see "Commit Changes to Git" section under "Iterative Subtask Implementation")
-   Mark completed tasks with `set_task_status` / `task-orchestrator set-status --id=<id> --status=done` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)) only after committing changes
-   Update dependent tasks when implementation differs from original plan using `update` / `task-orchestrator update --from=<id> --prompt="..."` or `update_task` / `task-orchestrator update-task --id=<id> --prompt="..."` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc))
-   Add new tasks discovered during implementation using `add_task` / `task-orchestrator add-task --prompt="..."` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)).
-   Add new subtasks as needed using `add_subtask` / `task-orchestrator add-subtask --parent=<id> --title="..."` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)).
-   Append notes or details to subtasks using `update_subtask` / `task-orchestrator update-subtask --id=<subtaskId> --prompt='Add implementation notes here...\nMore details...'` (see [`task-orchestrator.mdc`](mdc:.cursor/rules/taskmaster.mdc)).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [declanshanaghy/dollar-game](https://github.com/declanshanaghy/dollar-game) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
