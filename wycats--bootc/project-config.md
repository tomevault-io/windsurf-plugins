---
trigger: always_on
description: When completing a task that involves modifying the OS image (e.g., merging a PR with `Containerfile` changes):
---

# Copilot Instructions

## Bootc Update Lifecycle

When completing a task that involves modifying the OS image (e.g., merging a PR with `Containerfile` changes):

1. **Verify Build**: Ensure the GitHub Action has successfully built and pushed the image (`gh run watch/list`).
2. **Stage Update**: Run `sudo bootc upgrade` to fetch the new image.
   - **Crucial**: Do NOT tell the user "it is safe to reboot" or "the changes are live" until you have successfully run this command.
3. **Handover**: Once the upgrade is staged, inform the user that a reboot is required to apply the changes.

**Every PR that touches the image requires ALL THREE steps.** If multiple PRs are merged in sequence, only the FINAL `sudo bootc upgrade` matters — but it MUST be run after the last build completes. Do not assume a previous `bootc upgrade` covers a later PR.

**The word "reboot" must not appear in any message to the user unless `sudo bootc upgrade` has been run in the current session AND its output confirmed success (look for "Queued for next boot").**

## Tool Usage: ask_questions

Use the `ask_questions` tool to stop and align with the user when:

- **Ambiguity exists**: You are unsure of the user's specific intent or preference.
- **High-Risk Actions**: Before performing destructive operations if not explicitly requested.
- **Workflow Breaks**: When switching context significantly.

**Do NOT use `ask_questions`**:

- To ask for permission to proceed with a task you've already been assigned.
- To confirm low-risk actions clearly implied by the user's request.

## Git Workflow

When asked to commit and/or create a PR:

1. **Always create a feature branch** - Never commit directly to `main`
2. **Use logical commits** - Split changes into meaningful commits (e.g., separate commits for new features, bug fixes, documentation)
3. **One PR per feature set** - Group related changes into a single PR with multiple commits

## Context Management Strategy

Treat the Main Chat context window as a scarce resource to be conserved. Apply the following heuristics to determine whether to execute a task directly or delegate it to a subagent, and how to transfer information effectively.

### Decision Heuristics

1. **Prioritize Isolation (Subagents)**
   Delegate tasks to subagents whenever the work is **modular**—meaning it can be fully defined by a standalone prompt. This "fire-and-forget" approach prevents the Main Chat from being polluted with intermediate steps, verbose output, or temporary data processing.
2. **Prioritize Continuity (Main Agent)**
   Execute tasks directly only when they possess **high contextual entanglement**—where the task relies so heavily on the specific history, nuance, or "state" of the current conversation that accurately serializing that context into a fresh prompt would be inefficient or lossy.
3. **Prefer "Pass by Reference"**
   When delegating, avoid "Passing by Value" (reading file contents into the chat to paste into a subagent prompt). Instead, "Pass by Reference" by directing the subagent to specific file paths or directories. If the data exists only in memory or a command output, write it to a temporary artifact (e.g., `.context/temp_diff.txt`) and point the subagent to that artifact.

### Context Strategy Examples

#### Example 1: High Modularity (Delegate)

**User Request:** "I need a set of unit tests for the `DateFormatter` utility we just pasted. Cover all edge cases for leap years and timezones."
**Model Reasoning:**

- **Input Context:** The target code (`DateFormatter`) is distinct and currently available.
- **Context Entanglement:** Low. The requirements are standard ("cover edge cases") and do not rely on a complex history of user preferences.
- **Token Economics:** Writing comprehensive tests is verbose. Doing this in the main chat would push valuable history out of the context window.
  **Decision:** Delegate to Subagent.
  **Instruction:** "Generate a comprehensive test suite for the provided `DateFormatter` code, ensuring coverage for leap years and timezone edge cases."

#### Example 2: High Entanglement (Execute Directly)

**User Request:** "That looks better, but it contradicts the 'Sentinels' pattern we discussed at the start of the session. Refactor the `User` struct to align with that instead."
**Model Reasoning:**

- **Input Context:** "Sentinels pattern" refers to a specific, agreed-upon definition from earlier in the chat, likely containing nuance not found in public documentation.
- **Context Entanglement:** High. To delegate this, I would have to synthesize and explain the user's specific definition to the subagent. The cost of accurately serializing this context approaches the cost of just doing the work.
- **Risk:** A subagent might hallucinate a standard definition of "Sentinels" rather than the user's specific version.
  **Decision:** Execute by Main Agent.

#### Example 3: Pass-by-Reference via File System (Delegate)

**User Request:** "Read through the entire `legacy_migration/` folder and identify any deprecated API calls in the Python files."
**Model Reasoning:**

- **Input Context:** The user is pointing to a large directory on disk.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wycats/bootc](https://github.com/wycats/bootc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
