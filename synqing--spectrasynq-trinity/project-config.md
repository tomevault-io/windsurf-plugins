---
trigger: always_on
description: MANDATORY protocol for executing implementation plans with independent tasks - dispatches fresh subagent for each task with code review between tasks, enabling fast iteration with quality gates. MANDATORY parallel execution for independent tasks.
---



# Subagent-Driven Development

**Protocol Version:** 2.0.0  
**Status:** MANDATORY  
**Last Updated:** 2025-01-XX

Create and execute plan by dispatching fresh subagent per task or issue, with code and output review after each or batch of tasks.

**Core principle:** Fresh subagent per task + review between or after tasks = high quality, fast iteration.

**MANDATORY REQUIREMENT:** Parallel execution MUST be used when tasks are independent and can be executed concurrently.

Executing Plans through agents:

- Same session (no context switch)
- Fresh subagent per task (no context pollution)
- Code review after each or batch of task (catch issues early)
- Faster iteration (no human-in-loop between tasks)

## Supported types of execution

### Sequential Execution

When you have a tasks or issues that are related to each other, and they need to be executed in order, investigating or modifying them sequentially is the best way to go.

Dispatch one agent per task or issue. Let it work sequentially. Review the output and code after each task or issue.

**When to use:**

- Tasks are tightly coupled
- Tasks should be executed in order

### Parallel Execution (MANDATORY)

**MANDATORY PROTOCOL:** When you have multiple unrelated tasks or issues (different files, different subsystems, different bugs), investigating or modifying them sequentially wastes time. Each task or investigation is independent and can happen in parallel.

**MANDATORY ACTION:** Dispatch one agent per independent problem domain. Let them work concurrently.

**MANDATORY - Parallel execution MUST be used when:**

- Tasks are mostly independent
- Overall review can be done after all tasks are completed
- No shared state conflicts exist
- Different file sets or subsystems are involved
- 3+ independent tasks exist

**MANDATORY - Task Decomposition:**
- Complex tasks MUST be divided into parallelizable components
- Each component must be independent and self-contained
- Components must have clear, specific goals

## Sequential Execution Process

### 1. Load Plan

Read plan file, create TodoWrite with all tasks.

### 2. Execute Task with Subagent

For each task:

**Dispatch fresh subagent:**

```
Task tool (general-purpose):
  description: "Implement Task N: [task name]"
  prompt: |
    You are implementing Task N from [plan-file].

    Read that task carefully. Your job is to:
    1. Implement exactly what the task specifies
    2. Write tests (following TDD if task says to)
    3. Verify implementation works
    4. Commit your work
    5. Report back

    Work from: [directory]

    Report: What you implemented, what you tested, test results, files changed, any issues
```

**Subagent reports back** with summary of work.

### 3. Review Subagent's Work

**Dispatch code-reviewer subagent:**

```
Task tool (superpowers:code-reviewer):
  Use template at requesting-code-review/code-reviewer.md

  WHAT_WAS_IMPLEMENTED: [from subagent's report]
  PLAN_OR_REQUIREMENTS: Task N from [plan-file]
  BASE_SHA: [commit before task]
  HEAD_SHA: [current commit]
  DESCRIPTION: [task summary]
```

**Code reviewer returns:** Strengths, Issues (Critical/Important/Minor), Assessment

### 4. Apply Review Feedback

**If issues found:**

- Fix Critical issues immediately
- Fix Important issues before next task
- Note Minor issues

**Dispatch follow-up subagent if needed:**

```
"Fix issues from code review: [list issues]"
```

### 5. Mark Complete, Next Task

- Mark task as completed in TodoWrite
- Move to next task
- Repeat steps 2-5

### 6. Final Review

After all tasks complete, dispatch final code-reviewer:

- Reviews entire implementation
- Checks all plan requirements met
- Validates overall architecture

### 7. Complete Development

After final review passes:

- Announce: "I'm using the finishing-a-development-branch skill to complete this work."
- **REQUIRED SUB-SKILL:** Use superpowers:finishing-a-development-branch
- Follow that skill to verify tests, present options, execute choice

### Example Workflow

```
You: I'm using Subagent-Driven Development to execute this plan.

[Load plan, create TodoWrite]

Task 1: Hook installation script

[Dispatch implementation subagent]
Subagent: Implemented install-hook with tests, 5/5 passing

[Get git SHAs, dispatch code-reviewer]
Reviewer: Strengths: Good test coverage. Issues: None. Ready.

[Mark Task 1 complete]

Task 2: Recovery modes

[Dispatch implementation subagent]
Subagent: Added verify/repair, 8/8 tests passing

[Dispatch code-reviewer]
Reviewer: Strengths: Solid. Issues (Important): Missing progress reporting

[Dispatch fix subagent]
Fix subagent: Added progress every 100 conversations

[Verify fix, mark Task 2 complete]

...

[After all tasks]
[Dispatch final code-reviewer]
Final reviewer: All requirements met, ready to merge

Done!
```

### Red Flags

**Never:**

- Skip code review between tasks
- Proceed with unfixed Critical issues
- Dispatch multiple implementation subagents in parallel (conflicts)
- Implement without reading plan task

**If subagent fails task:**

- Dispatch fix subagent with specific instructions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/synqing) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
