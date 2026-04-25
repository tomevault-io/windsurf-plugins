---
trigger: always_on
description: - **Global CLI Commands**
---

- **Global CLI Commands**
  - Task Master now provides a global CLI through the `task-manager` command
  - All functionality from `scripts/dev.js` is available through this interface
  - Install globally with `npm install -g claude-task-manager` or use locally via `npx`
  - Use `task-manager <command>` instead of `node scripts/dev.js <command>`
  - Examples:
    - `task-manager list` instead of `node scripts/dev.js list`
    - `task-manager next` instead of `node scripts/dev.js next`
    - `task-manager expand --id=3` instead of `node scripts/dev.js expand --id=3`
  - All commands accept the same options as their script equivalents
  - The CLI provides additional commands like `task-manager init` for project setup

- **Development Workflow Process**
  - Start new projects by running `task-manager init` or `node scripts/dev.js parse-prd --input=<prd-file.txt>` to generate initial tasks.json
  - Begin coding sessions with `task-manager list` to see current tasks, status, and IDs
  - Analyze task complexity with `task-manager analyze-complexity --research` before breaking down tasks
  - Select tasks based on dependencies (all marked 'done'), priority level, and ID order
  - Clarify tasks by checking task files in tasks/ directory or asking for user input
  - View specific task details using `task-manager show <id>` to understand implementation requirements
  - Break down complex tasks using `task-manager expand --id=<id>` with appropriate flags
  - Clear existing subtasks if needed using `task-manager clear-subtasks --id=<id>` before regenerating
  - Implement code following task details, dependencies, and project standards
  - Verify tasks according to test strategies before marking as complete
  - Mark completed tasks with `task-manager set-status --id=<id> --status=done`
  - Update dependent tasks when implementation differs from original plan
  - Generate task files with `task-manager generate` after updating tasks.json
  - Maintain valid dependency structure with `task-manager fix-dependencies` when needed
  - Respect dependency chains and task priorities when selecting work
  - Report progress regularly using the list command

- **Task Complexity Analysis**
  - Run `node scripts/dev.js analyze-complexity --research` for comprehensive analysis
  - Review complexity report in scripts/task-complexity-report.json
  - Or use `node scripts/dev.js complexity-report` for a formatted, readable version of the report
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
  - Legacy Syntax: `node scripts/dev.js parse-prd --input=<prd-file.txt>`
  - CLI Syntax: `task-manager parse-prd --input=<prd-file.txt>`
  - Description: Parses a PRD document and generates a tasks.json file with structured tasks
  - Parameters:
    - `--input=<file>`: Path to the PRD text file (default: sample-prd.txt)
    - `--num-tasks=<number>`: Number of tasks to generate (if not specified, AI will determine based on complexity)
    - `--knowledge-base=<path>, -k`: Path to business knowledge documents to use as context
  - Example: `task-manager parse-prd --input=requirements.txt`
  - Notes: Will overwrite existing tasks.json file. Use with caution.

- **Command Reference: update**
  - Legacy Syntax: `node scripts/dev.js update --from=<id> --prompt="<prompt>"`
  - CLI Syntax: `task-manager update --from=<id> --prompt="<prompt>"`
  - Description: Updates tasks with ID >= specified ID based on the provided prompt
  - Parameters:
    - `--from=<id>`: Task ID from which to start updating (required)
    - `--prompt="<text>"`: Explanation of changes or new context (required)
  - Example: `task-manager update --from=4 --prompt="Now we are using Express instead of Fastify."`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/skindhu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
