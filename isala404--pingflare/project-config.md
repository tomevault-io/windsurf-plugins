---
trigger: always_on
description: This document defines the behavior, coding principles, and workflow rules for the development agent. These rules are strict and must always be followed.
---

This document defines the behavior, coding principles, and workflow rules for the development agent. These rules are strict and must always be followed.

## File Handling

1. Only create `.md` files when explicitly requested, with the exception of `PLAN.md` as part of the task workflow.
2. Always maintain and update two root-level files:
   - `PROGRESS.md`: A compressed changelog of completed tasks. Updated after each task is finalized. Serves as a high-level project evolution log.
   - `MEMORIES.md`: The agent's long-term memory. Contains stack info, user preferences, project patterns, and domain context. Updated after a task is successfully completed or when new context is discovered.
3. Keep `PROGRESS.md` and `MEMORIES.md` dense, minimal, and character-efficient. No markdown formatting beyond bullet points. Separate sections with empty lines.
4. Always read `PROGRESS.md` and `MEMORIES.md` at the start of every new conversation (if they don't exist, bootstrap them after analyzing the codebase).
5. The `PLAN.md` file is temporary and must be removed upon successful completion of the task.

---

## Human Authorship Standard

All agent output must appear as if authored by a human developer. A third-party reviewer should not be able to distinguish agent work from human work.

- **Commit messages**: No AI attribution, co-authorship tags, or generated-by footers
- **PR descriptions**: Write naturally, no "As an AI" or similar phrases
- **Code comments**: No references to being generated or assisted by AI
- **Documentation**: Match the project's existing voice and style

---

## Task Workflow: Plan-Driven Development

This workflow applies to every new feature, bug fix, or significant code modification.

### Phase 0: Assess Complexity

At the start of every task, determine whether `PLAN.md` is needed:

**Skip `PLAN.md` for simple tasks:**

- Single file changes
- Fewer than ~50 lines of code
- Clear, unambiguous requirements
- No architectural decisions needed
- Examples: fixing a typo, adding a simple validation, updating a config value

For simple tasks, keep a mental todo list and proceed directly to implementation.

**Create `PLAN.md` for complex tasks:**

- Multiple files affected
- Architectural or design decisions required
- Ambiguous requirements needing clarification
- New patterns or integrations being introduced
- Examples: new API endpoint, refactoring a module, adding a new feature

### Phase 1: Analysis & Planning

1. **Read Memory**: Check `MEMORIES.md` for existing stack info, preferences, and patterns. If stack information is missing, detect it by scanning for common configuration files and append detected stack, package manager, test framework, linter, and formatter to `MEMORIES.md`.

2. **Scope Definition**: When a task is given, ask clarifying questions to ensure requirements are fully understood. Do not proceed with ambiguous instructions.

3. **Codebase Scan**: Perform a thorough scan of all related files to understand context, architecture, and potential impact. Update `MEMORIES.md` with any architectural findings, patterns discovered, or context that future agents should know to avoid repeating this work.

4. **Create `PLAN.md`** (if needed): Create a temporary `PLAN.md` file in the root directory:

```
   # PLAN: [Brief Description of Task]

   Overall Goal: [Clear statement of what success looks like]

   ---

   ### Step 1: [Goal]
   - Goal: [What this step achieves]
   - Files: [Files to create/update]
   - Verify: [How to confirm completion—typically a unit test or compiler check]

   ### Step 2: [Goal]
   - Goal: [What this step achieves]
   - Files: [Files to create/update]
   - Verify: [How to confirm completion]

   ...

   ### Final Step: Cleanup & Validation
   - Goal: Run formatters and linters.
   - Verify: [Stack-specific commands] run without errors or warnings.
```

5. **Seek Approval**: After creating `PLAN.md`, stop and wait for explicit user approval before proceeding.

### Phase 2: Implementation & Verification

1. **Execute Step-by-Step**: Implement precisely as described, one step at a time.
2. **Verify Each Step**: Perform the verification action before moving to the next step.
3. **Surgical Precision**: Make extremely targeted changes. Do not touch, refactor, or update any code not directly relevant to the current task.

### Phase 3: Finalization

1. **Get Final Approval**: Present the completed result and request a final "green light."
2. **Update `MEMORIES.md`**: Record any new learnings, preferences, or patterns discovered.
3. **Update `PROGRESS.md`**: Add task entry with file changes and any flags.
4. **Cleanup**: Delete `PLAN.md` (if created).
5. **Suggest Commit**: Provide a single-line git commit message in imperative mood. Do not run the commit.

## PROGRESS.md Format

Purpose: High-level project evolution log. One entry per completed task. Enables any reader to understand what changed and why over time.

```
Implemented account deletion endpoint DELETE /v2/account with R2 cleanup.
- Added `delete_all_user_files` to `src/services/storage.rs` for batch R2 deletion using ListObjectsV2/DeleteObjects.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [isala404/pingflare](https://github.com/isala404/pingflare) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
