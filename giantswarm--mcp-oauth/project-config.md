---
trigger: always_on
description: Guide for task-driven development workflows for the mcp-oauth library
---


# The Development Workflow

This document outlines the standard, iterative process for all development work on the `mcp-oauth` library. Following this workflow is **mandatory** to ensure consistency, quality, and effective collaboration across the project.

## The Core Development Loop

All work, from new features to bug fixes, follows this fundamental three-phase cycle:

**⚠️ REMEMBER: This repository uses `main` as the default branch. NEVER commit directly to `main`!**

1.  **Phase 1: Planning & Alignment**
    -   Select an issue from the backlog.
    -   Switch to the `main` branch and pull the latest changes
    -   **Create a dedicated feature branch from `main`** (NEVER work directly on `main`).
    -   Explore the codebase to create a detailed implementation strategy.

2.  **Phase 2: Iterative Implementation**
    -   **Log your plan first.** Document your intended changes in the relevant task *before* writing code.
    -   Implement the changes, adhering to the project's architectural guidelines.
    -   Continuously log progress, discoveries, and setbacks as you work to create a rich history of the task.

3.  **Phase 3: Finalization & Committing**
    -   Write and pass all required tests.
    -   Format your code according to project standards.
    -   Update any relevant project rules based on your changes.
    -   Commit your work using a structured, conventional commit message. Push your branch and create a PR.

## Phase 1: Planning & Alignment

Before writing a single line of implementation code, you must have a clear plan and be working on the correct branch.

### 1.1. Select an Issue & State Your Intent

1.  **List Open Issues:** Use `mcp_github_list_issues` to see available tasks for `giantswarm/mcp-oauth`.
2.  **Choose an Issue:** Select the highest-priority issue you are able to work on.
3.  **Get Details:** Use `mcp_github_get_issue` to retrieve its full details.
4.  **Announce Your Plan:** In the chat, you **MUST** summarize the issue (title, number) and your intended high-level approach.
    -   **Example:** *"I am starting work on issue #37: Add GitHub OAuth Provider. My plan is to start by creating a new provider package following the Google provider pattern..."*

### 1.2. Create a Dedicated Git Branch

**🚨 CRITICAL: NEVER COMMIT DIRECTLY TO THE `main` BRANCH! 🚨**

The repository uses `main` as the default branch (NOT `master`). All work MUST be done in feature branches.

1.  **Check Your Current Branch:** Run `git rev-parse --abbrev-ref HEAD`.
2.  **STOP if on `main`:** If you are on `main`, you MUST create a new branch immediately. DO NOT make any commits to `main`.
3.  **Branch Naming Convention:** Branch names **MUST** follow this format:
    > `<type>/issue-<number>-<kebab-case-title>`
4.  **Branch Types:** `feature`, `fix`, `refactor`, `docs`, `test`, `chore`.
5.  **Example:** `git checkout -b feature/issue-42-add-github-provider`

**Workflow Summary:**
- ✅ Switch to `main` and pull latest changes
- ✅ Always create a branch from `main`
- ✅ Make all commits to your feature branch
- ✅ Push your branch and create a Pull Request
- ✅ Merge to `main` only via Pull Request
- ❌ NEVER `git commit` while on `main`
- ❌ NEVER `git push origin main`

### 1.3. Explore and Plan

This is a critical step to ensure your implementation is well-considered.

1.  **Explore the Codebase:** Identify the specific files, functions, and lines of code that need to be added, removed, or changed.
2.  **Consider Backward Compatibility:** This is a library used by other projects. Will your changes break existing functionality? Do you need to version the change?
3.  **Formulate a Detailed Plan:** Based on your exploration, create a precise implementation plan. What is the exact diff you intend to apply? What potential challenges do you foresee? This plan will be logged in the next step.

## Phase 2: Iterative Implementation & Logging

This cycle is the heart of the workflow. It's designed to build a rich, contextual history of the implementation, which is invaluable for debugging, collaboration, and future reference.

### 2.1. Log Your Plan *Before* Coding

1.  Before you start implementing, log the detailed plan you created in step 1.3.
2.  Always update the description of the existing issue with your plan
3.  Verify that the plan was successfully logged by viewing the issue details again.

### 2.2. Implement & Log Progress

1.  **Set Issue Status:** Mark the issue as `in-progress`
2.  **Write Code:** Begin writing code according to your plan and the project's [architecture.mdc](mdc:.cursor/rules/architecture.mdc).
3.  **Log Continuously:** As you work, you will learn things. **You must log them.** Regularly update the issue to append new findings.
    -   ✅ **What Worked:** Confirmed approaches, "fundamental truths."
    -   ❌ **What Didn't Work:** Dead ends, failed experiments, and why.
    -   ⚙️ **Specifics:** Successful code snippets, configurations, or commands.
    -   🗣️ **Decisions:** Any choices made, especially if confirmed with the user.

## Phase 3: Finalization & Committing

### 3.1. Pre-Commit Quality Check

Before **every** commit, you **MUST** perform these checks:

1.  **Format Code:**
    ```bash
    goimports -w .
    go fmt ./...
    ```

2.  **Lint Code:**
    ```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giantswarm/mcp-oauth](https://github.com/giantswarm/mcp-oauth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
