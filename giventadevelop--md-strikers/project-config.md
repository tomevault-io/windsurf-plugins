---
trigger: always_on
description: Guide for using meta-development script (scripts/dev.js) to manage task-driven development workflows
---


- **Development Workflow Process**
  - Start new projects by running `node scripts/dev.js parse-prd --input=<prd-file.txt>` to generate initial tasks.json
  - Begin coding sessions with `node scripts/dev.js list` to see current tasks, status, and IDs
  - Analyze task complexity with `node scripts/dev.js analyze-complexity --research` before breaking down tasks
  - Select tasks based on dependencies (all marked 'done'), priority level, and ID order
  - Clarify tasks by checking task files in tasks/ directory or asking for user input
  - View specific task details using `node scripts/dev.js show --id=<id>` to understand implementation requirements
  - Break down complex tasks using `node scripts/dev.js expand --id=<id>` with appropriate flags
  - Clear existing subtasks if needed using `node scripts/dev.js clear-subtasks --id=<id>` before regenerating
  - Implement code following task details, dependencies, and project standards
  - Verify tasks according to test strategies before marking as complete
  - Mark completed tasks with `node scripts/dev.js set-status --id=<id> --status=done`
  - Update dependent tasks when implementation differs from original plan
  - Generate task files with `node scripts/dev.js generate` after updating tasks.json
  - Maintain valid dependency structure with `node scripts/dev.js fix-dependencies` when needed
  - Respect dependency chains and task priorities when selecting work
  - Report progress regularly using the list command

- **Task Complexity Analysis**
  - Run `node scripts/dev.js analyze-complexity --research` for comprehensive analysis
  - Review complexity report in scripts/task-complexity-report.json
  - Focus on tasks with highest complexity scores (8-10) for detailed breakdown
  - Use analysis results to determine appropriate subtask allocation
  - Note that reports are automatically used by the expand command

- **Task Breakdown Process**
  - For tasks with complexity analysis, use `node scripts/dev.js expand --id=<id>`
  - Otherwise use `node scripts/dev.js expand --id=<id> --subtasks=<number>`
  - Add `--research` flag to leverage Perplexity AI for research-backed expansion
  - Use `--prompt="<context>"` to provide additional context when needed
  - Review and adjust generated subtasks as necessary
  - Use `--all` flag to expand multiple pending tasks at once
  - If subtasks need regeneration, clear them first with `clear-subtasks` command

- **Implementation Drift Handling**
  - When implementation differs significantly from planned approach
  - When future tasks need modification due to current implementation choices
  - When new dependencies or requirements emerge
  - Call `node scripts/dev.js update --from=<futureTaskId> --prompt="<explanation>"` to update tasks.json

- **Task Status Management**
  - Use 'pending' for tasks ready to be worked on
  - Use 'done' for completed and verified tasks
  - Use 'deferred' for postponed tasks
  - Add custom status values as needed for project-specific workflows

- **Task File Format Reference**
  ```
  # Task ID: <id>
  # Title: <title>
  # Status: <status>
  # Dependencies: <comma-separated list of dependency IDs>
  # Priority: <priority>
  # Description: <brief description>
  # Details:
  <detailed implementation notes>
  
  # Test Strategy:
  <verification approach>
  ```

- **Command Reference: parse-prd**
  - Syntax: `node scripts/dev.js parse-prd --input=<prd-file.txt>`
  - Description: Parses a PRD document and generates a tasks.json file with structured tasks
  - Parameters: 
    - `--input=<file>`: Path to the PRD text file (default: sample-prd.txt)
  - Example: `node scripts/dev.js parse-prd --input=requirements.txt`
  - Notes: Will overwrite existing tasks.json file. Use with caution.

- **Command Reference: update**
  - Syntax: `node scripts/dev.js update --from=<id> --prompt="<prompt>"`
  - Description: Updates tasks with ID >= specified ID based on the provided prompt
  - Parameters:
    - `--from=<id>`: Task ID from which to start updating (required)
    - `--prompt="<text>"`: Explanation of changes or new context (required)
  - Example: `node scripts/dev.js update --from=4 --prompt="Now we are using Express instead of Fastify."`
  - Notes: Only updates tasks not marked as 'done'. Completed tasks remain unchanged.

- **Command Reference: generate**
  - Syntax: `node scripts/dev.js generate`
  - Description: Generates individual task files in tasks/ directory based on tasks.json
  - Parameters: None
  - Example: `node scripts/dev.js generate`
  - Notes: Overwrites existing task files. Creates tasks/ directory if needed.

- **Command Reference: set-status**
  - Syntax: `node scripts/dev.js set-status --id=<id> --status=<status>`
  - Description: Updates the status of a specific task in tasks.json
  - Parameters:
    - `--id=<id>`: ID of the task to update (required)
    - `--status=<status>`: New status value (required)
  - Example: `node scripts/dev.js set-status --id=3 --status=done`
  - Notes: Common values are 'done', 'pending', and 'deferred', but any string is accepted.

- **Command Reference: list**
  - Syntax: `node scripts/dev.js list`
  - Description: Lists all tasks in tasks.json with IDs, titles, and status
  - Parameters: None

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
