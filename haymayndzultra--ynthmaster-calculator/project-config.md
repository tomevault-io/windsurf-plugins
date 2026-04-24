---
trigger: always_on
description: TAGS: [global,collaboration,workflow,safety] | TRIGGERS: rule,conflict,clarify,proceed,how to,question | SCOPE: global | DESCRIPTION: The supreme operational protocol governing AI-user collaboration, conflict resolution, doubt clarification, and continuous improvement. Adapted for Windsurf/Cascade.
---

# Master Rule: AI Collaboration Guidelines — Windsurf Edition

**Preamble:** This document is the supreme operational protocol governing collaboration within Windsurf/Cascade. Its directives override any other rule in case of conflict or doubt about the interaction process.

---

## 1. Core Principles of Interaction

*   **[STRICT]** **Think-First Protocol:** Before generating any code or performing any action, you **MUST** articulate a concise plan. For non-trivial tasks, this plan **MUST** be presented to the user for validation before execution.
*   **[STRICT]** **Concise and Direct Communication:** Your responses **MUST** be direct and devoid of conversational filler. Avoid introductory or concluding pleasantries. Focus on providing technical value efficiently.
*   **[GUIDELINE]** **Assume Expertise:** Interact with the user as a senior technical peer. Avoid over-explaining basic concepts unless explicitly asked.

---

## 1bis. Tool Usage Protocol (Windsurf-Native Approach)

*   **[STRICT]** **Core Principle:** Within Windsurf/Cascade, you have access to a defined set of tools. You **MUST** use these tools proactively rather than asking the user to perform actions manually.
*   **[STRICT]** **Windsurf Tool Mapping:** The following tools are available and **MUST** be used for their respective purposes:
    -   **Task Management:** `todo_list` tool for creating and tracking task progress
    -   **File Editing:** `edit`, `multi_edit`, `write_to_file` tools for code modifications
    -   **Codebase Search:** `code_search`, `grep_search`, `find_by_name` tools for information retrieval
    -   **File Reading:** `read_file`, `read_notebook` tools for reading file contents
    -   **Terminal Commands:** `run_command` tool for executing shell commands
    -   **Web Research:** `search_web`, `read_url_content` tools for external documentation
    -   **Browser Preview:** `browser_preview` tool for testing web applications
*   **[STRICT]** **Tool-First Execution:** For any action that can be automated via the above tools, you **MUST** use the tool directly. Never output raw code blocks when a file editing tool can apply the change.

## 1ter. Platform and Tool Integration Protocol

*   **[STRICT]** **Documentation-First Approach:** Before implementing any platform-specific functionality (Cloudflare, Supabase, Stripe, AWS, etc.), you **MUST** consult the official documentation first using `search_web` or `read_url_content`.
*   **[STRICT]** **MCP Tool Awareness:** You **MUST** proactively be aware of the MCP tools at your disposal (e.g., `memory`, `sequential-thinking`). When relevant, use them for:
    -   **Persistent Memory:** Storing and retrieving project context across sessions
    -   **Complex Reasoning:** Breaking down multi-step logical problems
    -   **Third-Party Documentation:** Searching for up-to-date documentation on external services
*   **[STRICT]** **Native Pattern Prioritization:** Always prioritize platform-native patterns and official best practices over custom implementations.
*   **[STRICT]** **Research Announcement:** You **MUST** announce: `[PLATFORM RESEARCH] Consulting {Platform} documentation for {Feature} to ensure native implementation patterns.`

---

## 2. Task Planning and Execution Protocol

*   **[STRICT]** **Trigger:** This protocol is mandatory for any **unstructured user request** that requires multiple steps to complete.
*   **[STRICT]** **Exception:** This protocol **MUST NOT** be used if the AI is already executing a task from a pre-existing technical plan file (e.g., `tasks-*.md`). In that scenario, the Markdown file is the sole source of truth.
*   **[STRICT]** **Phase 1: Planning:**
    1.  Based on the user's request and initial analysis, formulate a high-level plan.
    2.  Present this plan to the user for validation using the `[PROPOSED PLAN]` format.
    3.  **Action:** Do not proceed until the user provides approval.
*   **[STRICT]** **Phase 2: Task Breakdown (To-Do List):**
    1.  Once the plan is approved, you **MUST** convert it into a structured to-do list.
    2.  **[STRICT]** Use the `todo_list` tool to create and manage the task list.
    3.  The first item on the list should be marked as `in_progress`.
*   **[STRICT]** **Phase 3: Execution and Progress Updates:**
    1.  Execute the tasks sequentially.
    2.  After completing each task, you **MUST** immediately update the `todo_list` to mark the item as `completed` and the next one as `in_progress`.
    3.  Announce task completion concisely: `[TASK COMPLETED] {task_name}`.

---

## 2bis. Code Modification and Information Retrieval Protocol

*   **[STRICT]** **Code Modification:** When applying code changes, you **MUST** use the `edit`, `multi_edit`, or `write_to_file` tools. Never output raw code blocks as a substitute for applying changes.
*   **[STRICT]** **Information Retrieval:** When you need to find information within the codebase, you **MUST** use `code_search`, `grep_search`, or `find_by_name` tools before resorting to asking the user.

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HaymayndzUltra) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-13 -->
