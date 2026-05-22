---
trigger: always_on
description: Guide for using meta-development script (scripts/dev.js) to manage task-driven development workflows
---

---
description: Guide for using meta-development script (scripts/dev.js) to manage task-driven development workflows
globs: ["tasks/tasks.json", "tasks/*.md", "scripts/dev.{js,cjs}"]
alwaysApply: true
---

- **Global CLI Commands**
  - Task Master now provides a global CLI through the `task-master` command
  - All functionality from `scripts/dev.js` is available through this interface
  - Install globally with `npm install -g claude-task-master` or use locally via `npx`
  - Use `task-master <command>` instead of `node scripts/dev.js <command>`
  - Examples:
    - `task-master list` instead of `node scripts/dev.js list`
    - `task-master next` instead of `node scripts/dev.js next`
    - `task-master expand --id=3` instead of `node scripts/dev.js expand --id=3`
  - All commands accept the same options as their script equivalents
  - The CLI provides additional commands like `task-master init` for project setup

- **Development Workflow Process**
  - Start new projects by running `task-master init` or `node scripts/dev.js parse-prd --input=<prd-file.txt>` to generate initial tasks.json
  - Begin coding sessions with `task-master list` to see current tasks, status, and IDs
  - Analyze task complexity with `task-master analyze-complexity --research` before breaking down tasks
  - Select tasks based on dependencies (all marked 'done'), priority level, and ID order
  - Clarify tasks by checking task files in tasks/ directory or asking for user input
  - View specific task details using `task-master show <id>` to understand implementation requirements
  - Break down complex tasks using `task-master expand --id=<id>` with appropriate flags
  - Clear existing subtasks if needed using `task-master clear-subtasks --id=<id>` before regenerating
  - Implement code following task details, dependencies, and project standards
  - **Maintain Analysis Documentation**: Whencomplex areas like circuits ([analysis.md](mdc:docs/analysis.md)), smart contracts, or core architecture, frequently update the relevant analysis/documentation file (e.g., `docs/analysis.md`) with findings, decisions, and current understanding. This ensures documentation stays synchronized with development progress.
  - **Run Tests After Edits**: After making significant code changes, especially to core components like smart contracts (`*.sol`), circuits (`*.circom`), or critical frontend logic, **run the relevant test suite** (e.g., `npx hardhat test`) to catch regressions early. Address any test failures before proceeding or committing changes.
  - Verify tasks according to test strategies before marking as complete
  - Mark completed tasks with `task-master set-status --id=<id> --status=done`
  - Update dependent tasks when implementation differs from original plan
  - Generate task files with `task-master generate` after updating tasks.json
  - Maintain valid dependency structure with `task-master fix-dependencies` when needed
  - Respect dependency chains and task priorities when selecting work
  - Report progress regularly using the list command


- **Post-Task Integration & Review (After Completing All Subtasks of a Parent Task ID)**
    - **Mark Parent Task Done**: Once all subtasks for a parent task ID are marked 'done', mark the parent task itself as 'done'.
        ```bash
        # Example: Mark parent task 1 as done
        task-master set-status --id=1 --status=done
        ```
    - **Integration Testing**:
        - **Goal**: Verify that the functionalities implemented across all subtasks of the completed parent task work together correctly.
        - **Process**:
            - Identify or create integration tests relevant to the completed parent task (e.g., testing deposit, proof generation, and withdrawal flow together for the ZK Mixer core).
            - Run these tests (e.g., `npx hardhat test test/integration/Task1Integration.test.js` or a relevant script).
            - Address any failures before proceeding.
        - **AI Assistance**: Ask the AI to help identify, generate, or run relevant integration tests for the completed task ID.
            ```
            Task 1 and all its subtasks are complete. Can you help create or run integration tests for the ZK Mixer core architecture?
            ```
    - **Final Rule Review & Adaptation**:
        - **Goal**: Re-evaluate project rules based on the learnings and patterns from the *entire* completed parent task.
        - **Process**:
            - Review relevant rule files (e.g., `solidity_rules.mdc`, `dev_workflow.mdc`) considering the overall implementation of the completed task.
            - Identify any new patterns, necessary adjustments, or potential new rules based on the completed feature set.
        - **AI Assistance**: Prompt the AI to assist in this review.
            ```
            Task 1 (Setup Core Architecture) is fully complete. Let's review solidity_rules.mdc and dev_workflow.mdc to see if any adaptations are needed based on this phase.
            ```
        - This complements the subtask-specific rule review (which should still be done per subtask) by providing a holistic perspective after a major feature/phase is complete.


- **Task Complexity Analysis**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Zyra-V21/ZKUzumaki](https://github.com/Zyra-V21/ZKUzumaki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
