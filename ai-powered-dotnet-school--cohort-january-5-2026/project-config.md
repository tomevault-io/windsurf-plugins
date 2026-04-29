---
trigger: always_on
description: **Your Core Task: Generate a multi-commit task plan file, saved as `docs/tasks/<YYYY-MM-DD-HH-MM-task-name>.md`. This plan must be meticulously detailed, based *primarily* on information from the project's `docs/` directory, and explicitly incorporate comprehensive testing and logging for each step.**
---

# Create a Detailed, Verifiable Task Plan

**Your Core Task: Generate a multi-commit task plan file, saved as `docs/tasks/<YYYY-MM-DD-HH-MM-task-name>.md`. This plan must be meticulously detailed, based *primarily* on information from the project's `docs/` directory, and explicitly incorporate comprehensive testing and logging for each step.**

## File Existence Check

Verify key documentation files before planning:

```bash
ls . docs
```

Use the results to tailor the task plan.

## I. Pre-computation: Capture Timestamp

1.  **Obtain Current Timestamp:** Execute a terminal command to get the current date and time in `YYYY-MM-DD-HH-MM` format. (Example for Linux/macOS: `date +"%Y-%m-%d-%H-%M"`).
2.  **Formulate Filename:** Use this timestamp and a descriptive task name (derived from user request or PRD) to construct the full filename, e.g., `docs/tasks/2023-10-27-14-35-implement-user-auth.md`.

## II. Information Gathering (CRITICAL PRE-REQUISITE)

- **Mandatory Context Review:** Before writing any part of the plan, you **MUST** find and thoroughly read all relevant files in the project's `docs/` directory. This is not optional.
- **Prioritized Documents:** Pay special attention to (if they exist):
    - `docs/PRD.md` (Product Requirements Document)
    - `docs/TECH_STACK.md`
    - `docs/openapi.yaml` (or similar API specifications)
    - `docs/logging.md`, `docs/LOGGING_GUIDE.md` (or any docs related to logging practices)
    - `docs/TEST_STRATEGY.md`, `docs/TESTING_GUIDELINES.md` (or any docs related to testing)
    - `NOTES.md` (if present at project root or in `docs/`)
- **Purpose:** This information is *essential* for creating an accurate, relevant, and genuinely useful task plan that aligns with project standards.

## III. Core Directive: Construct the Task Plan File

Create the content for the `docs/tasks/<YYYY-MM-DD-HH-MM-task-name>.md` file. This file outlines a step-by-step plan, broken down into a series of distinct commits (typically 2-5 commits per task). Each commit in the plan **must** be verifiable and **must** explicitly incorporate both testing and logging best practices as detailed below.

### A. Strict Operational Constraints

- **File Operations:**
    - You are **ONLY** permitted to write to the specific task plan file you are creating (e.g., `docs/tasks/YYYY-MM-DD-HH-MM-task-name.md`).
    - Reading project documentation from `docs/` (as specified above) is **MANDATORY** to inform the plan.
- **Communication Protocol:**
    - **NO Conversational Output:** You are forbidden from generating any conversational output, commentary, preamble, or summaries *before* or *during* the creation of the task plan file content.
    - **Output is the File Content:** Your *entire* output for this specific rule invocation must be *only* the complete, raw Markdown content of the `docs/tasks/<YYYY-MM-DD-HH-MM-task-name>.md` file.
- **User Interaction:**
    - You receive the initial task description from the user.
    - If, after a **thorough review** of all available `docs/` materials, the task description remains insufficient to create a coherent and specific plan, you **MUST** indicate this *within* the task plan file itself using the HTML comment format: `<!-- TODO: [Specify missing information and reference document, e.g., 'Need API endpoint for user creation (see docs/openapi.yaml)'] -->`. Do not invent details.

### B. Testing & Observability (Mandatory for Each Commit)

1.  **Primary Verification = Automated Tests:** This is the preferred method.
    - **First Choice: Unit Tests.** Aim for small, isolated, fast tests targeting specific functions or modules.
    - **Second Choice: Integration Tests.** For broader scope, interactions between components, API contract testing, or E2E flows.
    - **Fallback (Only if Automated Test is Genuinely Infeasible): Explicit runtime logging or debug output checks.** This must be justified.

2.  **Logging is ALWAYS Required:** Even when automated tests exist, each commit's implementation **MUST** also include relevant, contextual logging or debug statements. This provides additional runtime visibility and aids in troubleshooting.
    - **Toggleable Logging:** Logging should be configurable (e.g., via an environment variable like `LOG_LEVEL=debug`, a feature flag, or a build-time switch) so it can be enabled/disabled without code modification.

3.  **Structured & Centralized Logs:**
    - Prefer structured log output (e.g., JSON lines or clear `key="value"` pairs).
    - Adhere to the project's established logging library and configuration (e.g., `pino`, `winston`). Reference the specific logger configuration file if known (e.g., `src/utils/logger.ts`).
    - Ensure logs are directed to the project's standard log aggregation system, if applicable.

4.  **Verification Details in Commit Plan (Hierarchy):**
    - **If Unit Test:** Specify the exact command to run the test(s) (e.g., `pnpm test --filter user-service.test.ts`) AND describe the key assertion(s) or link to an expected output/snapshot.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ai-powered-dotnet-school) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
