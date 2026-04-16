---
trigger: always_on
description: Before executing any user prompt, I **MUST** read the corresponding issue's chat file as defined in `CHAT.md`. After completing all actions for a prompt, I **MUST** append my activity log to that same file. This is non-negotiable for multi-agent coordination.
---

### **CRITICAL: Agent CHAT Protocol**

Before executing any user prompt, I **MUST** read the corresponding issue's chat file as defined in `CHAT.md`. After completing all actions for a prompt, I **MUST** append my activity log to that same file. This is non-negotiable for multi-agent coordination.

---

### **CRITICAL: Workspace Protocol**

You **MUST** operate exclusively from within your assigned `git worktree` directory (e.g., `gemini-worktree/`).

- **DO NOT** operate in the main project root directory; it is reserved for the human operator.
- All commands (`git`, file edits, etc.) must be executed from your worktree.
- This is a non-negotiable directive to ensure a clean and isolated development environment. It is the first step before the "Issue and Branching Protocol".

---
# Gemini Workflow

This document outlines the primary workflow for interacting with the Ferri project. I will use this as my guide for our sessions.

## Session Start

At the start of each session, I will:

1. Greet you with a fact about rust or python to help me learn
2.  **Ask you what you'd like to do.**

---

## What would you like to do?

**1. Check project status:** I will use `gh issue list` to analyze GitHub issues and cross-reference them with the actual implementation in the codebase to give you a summary of completed, in-progress, and pending tasks.

**2. Work on a ticket:** Tell me which ticket you want to work on (e.g., "Let's work on issue #8"). I will then focus on implementing the issue.

**3. Create a new ticket:** I will help you create a new GitHub issue with appropriate labels, milestones, and a clear description of the work required.

**4. Mark a ticket as done:** Tell me the issue number you've completed (e.g., "#5 is done"), and I will close the GitHub issue.

**5. Something else:** If you have a different task in mind, just let me know.

---

## Session Logging

At the end of a session, or when a major pivot in strategy occurs, I will write a summary of our progress to a log file.

* **Location:** `development_resources/logs/`
* **Filename:** `log-YYYY-MM-DD.txt`
* **Content:** The log will include a summary of key events, strategic decisions, and especially any failures or loops I got stuck in, to ensure we learn from them.

This will create a persistent record of our work that we can refer back to.

---

## Core Reference Files

To inform our work, I will always refer to:
* **`README.md`**: For understanding the project's architecture and goals.
* **GitHub Issues** (via `gh issue list` and `gh issue view`): As the single source of truth for all development tasks.
* **The source code**: To verify implementation status.

This should go without saying but always confirm if changing the .gitignore. It already happened before and it made things go crazy for a bit.

---
## Development Best Practices

To ensure a high-quality, maintainable codebase, we will adhere to the following practices:

* **Atomic Commits:** Each commit should represent a single, logical change. Commits should reference their corresponding issue number (e.g., `fix(auth): #21 - Fix login redirect bug`).
* **Test-Driven Development (TDD):** For new functionality, we will write a failing test *before* writing the implementation. This ensures the code is correct, testable, and meets requirements from the start.
* **Continuous Integration:** After implementing a change, I will always prompt you to run the project's build, lint, and test suites. I will not run them myself. This is to catch regressions early while keeping you in control of the execution environment.
* **Zero-Warning Policy:** Compiler warnings must be treated as errors and fixed immediately. A clean, warning-free build is required at all times.
* **User-Centric Testing:** When a feature is complete, I will install the binary and provide clear instructions for you to perform the final user acceptance testing (UAT).
* **Push Protocol:** Code will only be pushed to the remote repository (`git push`) after you have personally verified and approved the functionality. Frequent local commits will continue.

---
## Multi-Agent Collaboration

When working in a multi-agent environment, all development practices are superseded by the formal protocol defined in `AGENTS.md`. All agents, including this one, **must** adhere to that protocol to ensure synchronized, conflict-free work.

---

### **CRITICAL: Issue and Branching Protocol**

To ensure all work is tracked, isolated, and aligned with project management, the following protocol is **mandatory and non-negotiable**.

1.  **Issue First:** No work of any kind shall begin without a corresponding GitHub issue.
    * Before any action, I will generate the appropriate `gh issue create` command.
    * I will present this command to you for execution.
    * I will not proceed until the issue is created.

2.  **Branch from Develop:** All work must be done on a dedicated feature branch created from `develop`.
    * After an issue is created, I will ensure I'm on the `develop` branch with `git checkout develop`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ferri-tools) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
