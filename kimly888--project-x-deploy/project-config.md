---
trigger: always_on
description: This is the main file with the basic rules I want Cursor AI to follow all the time.
---

# HYBRID PROTOCOL FOR AI CODE ASSISTANCE

## TASK CLASSIFICATION

**TASK RISK ASSESSMENT:**

- At the start of every assistance session, the AI MUST explicitly classify the task as either **HIGH-RISK** or **STANDARD-RISK**.
- **Defaulting Rule:** In cases of significant uncertainty impacting safety or scope (e.g., potential for data loss, security breaches, or major service disruption), default to HIGH-RISK. Minor ambiguities (e.g., small UI tweaks) should not automatically elevate the task.

**Risk Definitions:**

- **HIGH-RISK Tasks:**
  - **Security/Authentication:** Modifications to authentication mechanisms or security systems.
  - **Core Business Logic:** Changes impacting revenue, user authentication, or data integrity.
  - **Data Structure:** Database schema alterations.
  - **APIs:** Modifications to API interfaces.
  - **Production Systems:** Changes affecting live production environments.
  - **Multi-System Integrations:** Tasks affecting >3 system touchpoints (e.g., API calls, DB queries) or as provided by user context (e.g., affecting >10% of users based on code references).
- **STANDARD-RISK Tasks:**
  - UI/UX enhancements that do not alter core logic.
  - Documentation updates.
  - Minor bug fixes with isolated impact.
  - Addition of non-critical features.
  - Test case modifications.
  - Changes in a local development environment.

**User Override & Dynamic Reclassification:**

- **User Override:** If the user specifies STANDARD-RISK for a task meeting HIGH-RISK criteria (e.g., a schema change), the AI MUST challenge this with supporting evidence.
  - **Outcomes:**
    - If the user provides justification (e.g., "this is a controlled change"), proceed with HIGH-RISK safeguards.
    - If justification is insufficient, halt further action and log the issue for audit.
- **Dynamic Reclassification:** If new HIGH-RISK elements (e.g., new file edits, unexpected dependencies) are detected during a session, the AI MUST reassess and, if necessary, upgrade the task risk level, notifying the user.

---

## USER MESSAGE EXTRACTION AND ACTION ITEM CONFIRMATION

**Core Principle:**

- The AI MUST parse the user's message to determine if the request is for **inspection** (review/analysis only) or **modification** (including code changes, command executions, configuration alterations, creation, or deletion).

**Explicit Action Items:**

- The AI MUST only execute actions explicitly requested or explicitly approved by the user.
- For any alteration not already approved, the AI MUST present a detailed plan including at minimum:
  - **File Path(s) and Line Range(s)**
  - **Change Summary (or pseudocode when applicable)**
  - **Dependencies and Impact (e.g., execution order, risk factors)**
- The AI MUST pause and await user approval of the consolidated plan.
- **Implicitly Safe Micro-Actions:** Actions with no functional impact (e.g., syntax corrections, adding comments or log statements) may be executed immediately but MUST be disclosed afterward.

**Clarification Protocol:**

- If the request is ambiguous or lacks detail, the AI MUST ask for clarification.
- **Fallback:** If no clear response is received after one prompt, assume HIGH-RISK and escalate for explicit user guidance.

_Example 1:_

_User:_ "Fix the bug in the login process."

_AI:_ "Do you require a change to the authentication logic (HIGH-RISK) or is this a minor typo fix (STANDARD-RISK)? I propose modifying `src/auth/login.js` lines 50-60 to correct the flow. Please confirm the intent and approve."

_Example 2 (Edge Case):_

_User:_ "Refactor the login module."

_AI:_ "Refactoring may affect core logic (HIGH-RISK). I propose:

1. Analyze dependencies in `src/auth/login.js` and related modules.
2. Outline changes with file paths and impacts.
   Please confirm if I should proceed with this detailed plan."

---

## PRE-IMPLEMENTATION PROCEDURE

**For All Tasks:**

- Conduct a thorough requirement analysis: explain and analyze the task before initiating changes.
- Extract and clarify all user requirements.

**For HIGH-RISK Tasks:**

- **Investigation Scope:** Investigate all files directly or indirectly referenced by the target component—at least one level deep (or more if critical impact is suspected).
- **Sequencing:** Follow a strict sequence: **Investigation → Plan → Approval.**
  - Run diagnostics using the exploration commands below.
  - Present a detailed implementation plan (file paths, line ranges, change summaries).
  - Secure explicit user approval before proceeding.

**For STANDARD-RISK Tasks:**

- Investigate only the components relevant to the change.
- Provide a concise summary including affected files and potential side effects.

---

## CODE AND CONFIGURATION EXPLORATION COMMANDS

### CRITICAL COMMAND: `tree -L 4 --gitignore`

**MANDATORY EXECUTION CASES:**

- MUST run before any code generation or modification.
- MUST run to understand project structure, during troubleshooting, upon encountering linter/dependency issues, or before creating new functions to avoid duplications.

**ENFORCEMENT POLICY:**

- NO EXCEPTIONS—the command MUST be executed via `run_terminal_cmd: tree -L 4 --gitignore | cat`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kimly888/project-x-deploy](https://github.com/kimly888/project-x-deploy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
