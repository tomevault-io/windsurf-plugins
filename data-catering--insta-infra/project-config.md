---
trigger: always_on
description: This rule guides your actions when you are implementing code changes based on an approved implementation plan (e.g., `docs/implementation/wails-ui-plan.md`). The goal is to ensure that all code you write or modify for `insta-infra` is simple, readable, clean, testable, and aligns with both the plan and existing project standards. This applies to both Go backend development and Web UI/React frontend development.
---

# insta-infra Code Implementation Best Practices (When Executing a Plan)

## 1. Introduction and Purpose

This rule guides your actions when you are implementing code changes based on an approved implementation plan (e.g., `docs/implementation/wails-ui-plan.md`). The goal is to ensure that all code you write or modify for `insta-infra` is simple, readable, clean, testable, and aligns with both the plan and existing project standards. This applies to both Go backend development and Web UI/React frontend development.

**CRITICAL:** This code serves end users directly. Every change must maintain system stability and pass all integration tests.

## 2. Code Change Strategy and Quality Assurance

*   **Batch File Edits Efficiently:**
    *   When you have multiple small edits to make on a single file (and the file is not excessively large), **combine all edits into a single operation** to minimize disruption and ensure consistency.
    *   For files under ~500 lines, batch all related changes together using a single `edit_file` or `search_replace` call.
    *   For larger files or complex changes spanning multiple areas, break into logical groups but minimize the number of edit operations.
*   **Pre-Change Validation:**
    *   Before making ANY code change, understand the existing logic thoroughly by reading the relevant code sections.
    *   Identify all potential impact points: functions that call the code you're changing, dependencies, and related components.
    *   **Never break existing logic** - this code is extremely important and serves real end users.
*   **Post-Change Verification:**
    *   After EVERY code change, **immediately run the relevant tests** to ensure nothing is broken.
    *   **Integration tests MUST always pass** - this is non-negotiable for end-user facing code.
    *   If any test fails, fix the issue immediately before proceeding with other changes.
*   **Incremental Safety:**
    *   Make changes incrementally when dealing with complex modifications.
    *   Test after each logical group of changes to catch issues early.
    *   If you're unsure about the impact of a change, ask for guidance rather than risk breaking functionality.

## 3. Plan Adherence and Communication

*   **State Current Task:** Before starting work on a task from the plan, explicitly state: "Now working on: [Task Description from Plan]".
*   **Follow the Plan:**
    *   Strictly adhere to the tasks, sub-tasks, and implementation details specified in the active implementation plan.
    *   Do not deviate significantly without confirming with the user.
*   **Verify Each Step:**
    *   After implementing a task (or a significant sub-task), **immediately execute the corresponding testing step** as defined in the plan.
    *   Report the outcome: "Testing for '[Task Description]': [Command Executed (e.g., `go test ./cmd/insta/...`, `make test`, `npm test --prefix ui` if UI tests exist, or manual UI check)] -> [Pass/Fail] - [Brief observation or output snippet]".
    *   If a test fails, attempt to debug and fix the implementation. If the fix requires a deviation from the plan or takes significant effort, inform the user.
*   **Communication is Key:**
    *   If you encounter ambiguities in the plan, blockers, or realize a task needs to be changed, communicate this to the user *before* proceeding with a modified approach.
    *   Request clarification if any part of the plan or existing codebase is unclear.
    *   Upon completion of all tasks, state: "Implementation plan fully executed."

## 4. Core Coding Principles

*   **Simplicity (KISS - Keep It Simple, Stupid):**
    *   Opt for the most straightforward solution that meets the requirements.
    *   Avoid unnecessary complexity, abstractions, or over-engineering.
*   **Readability:**
    *   Use clear, descriptive, and consistent names for variables, functions, classes, methods, and Go struct fields.
    *   Format code neatly and consistently. For Go, adhere to `gofmt`/`goimports`. For React/TypeScript, use Prettier (if configured, or maintain consistent formatting).
    *   Write code that is as self-documenting as possible.
*   **Clean Code:**
    *   **DRY (Don't Repeat Yourself):** Consolidate and reuse code where appropriate. Avoid duplicated logic.
    *   **SRP (Single Responsibility Principle):** Functions/methods/classes/Go packages should ideally do one thing well. Keep them focused and concise.
    *   Minimize side effects in functions where possible.
    *   Properly manage resources (e.g., close file handles, release locks, manage database connections).
*   **Testability:**
    *   Write code in a way that is easy to unit test. For Go, this means well-defined packages and functions. For React, this means creating testable components and using libraries like React Testing Library (if adopted).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [data-catering/insta-infra](https://github.com/data-catering/insta-infra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
