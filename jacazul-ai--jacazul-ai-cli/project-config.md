---
trigger: always_on
description: This document defines the foundational engineering standards, architectural
---

# Jacazul AI CLI Manifesto

This document defines the foundational engineering standards, architectural
patterns, and operational philosophies for the Jacazul AI CLI project.

## 🏛 Architectural Boundaries

### Structure vs. Dynamics (Setup vs. Runtime)
- **Setup (Structure - `scripts/configure`):** One-time environment
  preparation. Handles immutable filesystem changes: directory creation,
  symbolic links in `~/bin`, and initial template deployment. It sets the stage
  but does not run the show.
- **Runtime (Dynamics - `scripts/bootstrap/`):** Session-specific
  initialization. Handles mutable configuration and dynamic environment
  detection: injecting environment variables, surgical updates to settings
  JSONs (e.g., `experimental.enableAgents`), and locating system resources
  (e.g., finding the real `task` binary).

## 🔊 Logging Philosophy

The project adheres to a "Silent by Default" logging policy to maintain CLI
usability and focus.

- **Standard Execution:** Silence is mandatory if the environment is healthy
  and checks pass.
- **State Changes:** Output MUST be emitted when the system state is modified
  (e.g., "Creating directory X").
- **Verification:** Verification of existing resources MUST stay silent unless
  `DEBUG=true`.
- **Debug Mode:** Enabled via `DEBUG=true`. Provides full verbosity for
  troubleshooting.
- **Dry Run:** Enabled via `DRY=true`. Allows verifying the entire bootstrap
  process (Dynamics) without executing the final CLI binary.
- **Error Handling:** Errors MUST be emitted to `stderr` with clear
  instructional context.

## 🔒 Engineering Mandates

### 1. Taskwarrior Abstraction
- **Mandate:** Agents and tools MUST NOT invoke the raw `task` binary directly.
- **Security:** The raw `task` command is obfuscated to prevent accidental
  bypass. If an agent encounters the `scripts/task` wrapper, it MUST stop and
  consult the user.
- **Admin Bypass:** The `rtask` command provides a project-specific bypass to
  the real binary. This tool is for MANUAL ADMINISTRATIVE USE ONLY.
- **Protocol:** All operations MUST go through the `taskp` project-aware
  wrapper or the `tw-flow` workflow manager.
- **Isolation:** Project isolation via `TASKDATA` MUST be preserved at all costs.

### 2. Hatch-Generated Skills (CRITICAL)
- **Mandate:** The `jacazul-engine` skill and all agent prompt files are
  **generated artifacts** produced by `jacazul-hatch` from source templates.
  NEVER edit generated files directly.
- **Source of Truth:** All skill and agent prompt updates MUST be made in
  `jacazul/hatch/templates/`. The templates are the canonical source.
- **Regeneration:** After editing templates, regenerate with `jacazul-hatch
  --client <client>` to propagate changes.
- **Generated Locations:** `skills/jacazul-engine/SKILL.md` (from
  `gemini_full.md`), `agents/{persona}-{client}.md` (from `agent_master.md`).

### 3. Context Preservation
- **Mandate:** Closing a task without documentation is FORBIDDEN.
- **Protocol:** The `tw-flow done` command requires an `OUTCOME:` annotation.
  Discarded tasks MUST include an automatic audit record.

## 🧠 Session Stabilization & Context Engineering

These directives ensure that the AI ecosystem remains functional and
context-aware across different platforms and tool availability states.

### 1. Multi-Agent Diagnostic Loop
- **Protocol:** Technical challenges should follow a cross-agent verification
  loop. A diagnosis produced by one agent (e.g., Copilot/Haiku) MUST be
  re-interpreted, validated, and implemented by the session navigator (e.g.,
  Gemini). This ensures that "things work the first time" by using multiple
  perspectives to identify the root cause before acting.

### 2. Tool-Agnostic Resilience
- **Directive:** Agents MUST be capable of operating in "limbo" states where
  high-level tools (create/edit) are unavailable.
- **Fallback:** Use base system primitives (standard bash redirection: `cat >`,
  `touch`, `echo >>`) to achieve filesystem changes. Always verify the state
  change manually (`ls`, `cat`) after a workaround execution.

### 3. Horizontal Skill Architecture
- **Mandate:** Agents MUST activate required expert skills (`jacazul-engine`,
  `taskwarrior-expert`, `git-expert`) directly and simultaneously.
- **Goal:** Avoid cascading dependencies where one skill activates another.
  Independence ensures that a failure in one subsystem does not blind the
  entire agent.

### 4. The Keystone Pattern (Context Resolution)
- **Philosophy:** Skills are not "optional tools"—they are the foundation that
  resolves instruction ambiguity. Activating a skill is equivalent to loading
  the project's Distribution (Distro).
- **Protocol:** Agents MUST activate the four core required skills in the
  **first turn**, in parallel with tactical state discovery (e.g., `tw-flow
  focus`).
- **Resolution:** Mandates defined within a loaded skill ALWAYS take precedence
  over generic system prompts when resolving operational conflicts. This
  ensures that the agent adopts the Jacazul identity and technical standards
  before the first response.

## 🧬 Interaction Standards
- **Context Hunting Protocol:** Agents MUST NOT ask the user for session
  context that exists in the system. Upon activation, the agent MUST "hunt" for

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jacazul-ai/jacazul-ai-cli](https://github.com/jacazul-ai/jacazul-ai-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
