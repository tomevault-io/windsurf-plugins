---
trigger: always_on
description: Guide for using Task Master to manage task-driven development workflows
---

# Task Master Development Workflow
This guide outlines the typical process for using Task Master to manage software development projects.

# The Development Workflow

This document outlines the standard, iterative process for all development work. Following this workflow is **mandatory** to ensure consistency, quality, and effective collaboration across the project.

## The Core Development Loop

All work, from new features to bug fixes, follows this fundamental three-phase cycle:

1.  **Phase 1: Planning & Alignment**
    -   Select an issue from the backlog.
    -   Create a dedicated branch for your work.
    -   Explore the codebase to create a detailed implementation strategy.

2.  **Phase 2: Iterative Implementation**
    -   **Log your plan first.** Document your intended changes in the relevant task *before* writing code.
    -   Implement the changes, adhering to the project's architectural guidelines.
    -   Continuously log progress, discoveries, and setbacks as you work to create a rich history of the task.

3.  **Phase 3: Finalization & Committing**
    -   Write and pass all required tests.
    -   Format your code according to project standards.
    -   Update any relevant project rules based on your changes.
    -   Commit your work using a structured, conventional commit message.
    -   Close the corresponding issue.

---

## Phase 1: Planning & Alignment

Before writing a single line of implementation code, you must have a clear plan and be working on the correct branch.

### 1.1. Select an Issue & State Your Intent

1.  **List Open Issues:** Use `mcp_github_list_issues` to see available tasks for `giantswarm/mcp-opsgenie`.
2.  **Choose an Issue:** Select the highest-priority issue you are able to work on.
3.  **Get Details:** Use `mcp_github_get_issue` to retrieve its full details.
4.  **Announce Your Plan:** In the chat, you **MUST** summarize the issue (title, number) and your intended high-level approach.
    -   **Example:** *"I am starting work on issue #37: Refactor Capability API. My plan is to start by defining the new interfaces in the API package..."*

### 1.2. Create a Dedicated Git Branch

**NEVER** commit directly to the `main` branch.

1.  **Check Your Current Branch:** Run `git rev-parse --abbrev-ref HEAD`.
2.  **Create a New Branch:** If you are on `main`, create a new branch immediately. Branch names **MUST** follow this format:
    > `<type>/issue-<number>-<kebab-case-title>`
3.  **Branch Types:** `feature`, `fix`, `refactor`, `docs`, `test`, `chore`.
4.  **Example:** `git checkout -b feature/issue-42-add-prometheus-provider`

### 1.3. Explore and Plan

This is a critical step to ensure your implementation is well-considered.

1.  **Explore the Codebase:** Identify the specific files, functions, and lines of code that need to be added, removed, or changed.
2.  **Formulate a Detailed Plan:** Based on your exploration, create a precise implementation plan. What is the exact diff you intend to apply? What potential challenges do you foresee? This plan will be logged in the next step.

---

## Phase 2: Iterative Implementation & Logging

This cycle is the heart of the workflow. It's designed to build a rich, contextual history of the implementation, which is invaluable for debugging, collaboration, and future reference.

### 2.1. Log Your Plan *Before* Coding

1.  Before you start implementing, log the detailed plan you created in step 1.3.
2.  Use the `mcp_taskmaster-ai_update_subtask` tool. The prompt should be a complete summary of your findings and your intended approach.
3.  Verify that the plan was successfully logged by viewing the task details again.

### 2.2. Implement & Log Progress

1.  **Set Task Status:** Mark the task as `in-progress` (e.g., `mcp_taskmaster-ai_set_task_status --id=<subtaskId> --status=in-progress`).
2.  **Write Code:** Begin writing code according to your plan and the project's [architecture.mdc](mdc:.cursor/rules/architecture.mdc).
3.  **Log Continuously:** As you work, you will learn things. **You must log them.** Regularly use `mcp_taskmaster-ai_update_subtask` to append new findings.
    -   ✅ **What Worked:** Confirmed approaches, "fundamental truths."
    -   ❌ **What Didn't Work:** Dead ends, failed experiments, and why.
    -   ⚙️ **Specifics:** Successful code snippets, configurations, or commands.
    -   🗣️ **Decisions:** Any choices made, especially if confirmed with the user.

---

## Phase 3: Finalization & Committing

### 3.1. Pre-Commit Quality Check

Before **every** commit, you **MUST** perform these checks:

1.  **Format Code:**
    ```bash
    goimports -w .
    go fmt ./...
    ```
2.  **Run All Tests:**
    ```bash
    make test
    ```
    -   **DO NOT** commit if any tests are failing. Fix the tests first.
    -   Code must meet the test coverage minimums defined in [architecture.mdc](mdc:.cursor/rules/architecture.mdc).
3.  **Update TUI Snapshots (if applicable):**
    -   If you made changes to the TUI, update the golden files: `NO_COLOR=true go test ./internal/tui/view/... -update`.
    -   Review the changes to ensure they are intentional.

### 3.2. Review & Update Rules

1.  Review your implementation and the chat history.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giantswarm/mcp-opsgenie](https://github.com/giantswarm/mcp-opsgenie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
