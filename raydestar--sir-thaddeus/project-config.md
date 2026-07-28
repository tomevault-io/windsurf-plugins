---
trigger: always_on
description: > **Purpose:** Keep the project from drifting into “something else” as velocity increases.
---


# Cursor Rules — Sir Thaddeus (Local‑First Copilot Guardrails)

> **Purpose:** Keep the project from drifting into “something else” as velocity increases.
>
> **How Cursor should behave:** When a change request violates (or risks violating) these guardrails, **do not refuse**. Instead, **pause and push back** with a short check-in:
>
> **Pushback template:**
> *“Are you sure? You asked me to push back when we drift. This change touches: [Invariant(s)]. If we proceed, we should do it by: [safe alternative].”*

---

## 0) North Star

Sir Thaddeus is a **local‑first, explicit‑permission copilot runtime**.

* **Runs on the user’s machine** (Windows).
* **Desktop UI is optional** (tray/headless is first‑class).
* **LLM is local** (LM Studio / OpenAI‑compatible server).
* **Tools execute through a strict boundary** (MCP stdio tool server).
* **Audit log is always on**.

**If a feature makes it feel like a cloud agent, a spyware assistant, or an auto‑executing bot — it’s drift.**

---

## 1) Design Invariants (Non‑Negotiables)

### I1 — Agent has *no authority*

The agent may **request**, **route**, and **summarize**, but it does not decide policy.

* No “confidence-based” auto-approval.
* No hidden escalations.
* No silent capability expansion.

**Cursor pushback when:** agent logic becomes a policy engine (permissions, trust scores, “smart” auto behavior).

### I2 — Tools are behind a trust boundary (MCP)

All side effects occur through **MCP tools** in a **separate process**.

* No “direct calls” to system APIs from the agent loop for convenience.
* No bypassing MCP just because it’s local.

**Cursor pushback when:** code adds new side effects outside MCP.

### I3 — Explicit permission for side effects

Any action that changes the system, files, network state, or reveals sensitive data must be **explicitly permitted**.

* Default stance: **deny** until approved.
* Denial must be explicit and logged.

**Cursor pushback when:** someone suggests “just do it” or “auto-run” behaviors.

### I4 — Auditability is first-class

Everything meaningful is written to an **append-only audit log**.

* Include: timestamp, session/run id, tool name, inputs (redacted as needed), outputs (redacted), decision/permission outcome.

**Cursor pushback when:** features introduce invisible actions or unlogged tool execution.

### I5 — UI is not the identity

UI is a shell. Runtime is the product.

* Removing WPF should not collapse the architecture.

**Cursor pushback when:** logic migrates into UI code-behind or UI becomes required for core flows.

### I6 — Local-first means no surprise networking

Networking is explicit and bounded.

* No telemetry by default.
* No background “helpful” uploads.
* Any outbound request must be via a tool with policy + audit.

**Cursor pushback when:** libraries/services introduce network calls implicitly.

---

## 2) Architecture Rules (Keep the separation clean)

### A1 — Frontend (apps/desktop-runtime)

Allowed:

* Tray, overlay, hotkeys, PTT trigger, TTS output
* UI rendering of agent events

Not allowed:

* Tool execution
* Permission decisions
* Policy logic

### A2 — Agent (packages/agent)

Allowed:

* Conversation loop
* Tool routing (MCP client)
* State machine transitions
* Prompt construction

Not allowed:

* Direct system modifications
* Direct filesystem/shell calls (except launching MCP server process)

### A3 — LLM client (packages/llm-client)

Allowed:

* Transport only (OpenAI-compatible calls)

Not allowed:

* Tool logic
* State logic

### A4 — MCP server (apps/mcp-server)

Allowed:

* Implement tools
* Enforce allowlists / guardrails per tool
* Be stateless per tool call where possible

Not allowed:

* Agent logic
* UI coupling

---

## 3) Tooling Rules (Prevent “agent cleverness creep”)

### T1 — Tools must be **declarative + bounded**

Every tool must declare:

* inputs schema
* output schema
* max size/limits (time, bytes, entries)
* safety constraints

### T2 — Tools must be **idempotent** or carry an idempotency key

MCP calls may be retried.

* Tools should not double-apply side effects.
* If side effects exist, require an **idempotency key** and store a short-lived dedupe record.

**Cursor pushback when:** adding side-effecting tools without retry safety.

### T3 — Strict allowlists for execution

`SystemExecute`:

* Use an allowlist of commands and argument patterns.
* No raw shell execution.
* Prefer structured tools over “execute arbitrary command.”

### T4 — “Observation” tools are safer than “Action” tools

Prioritize:

* FileList, FileRead (bounded)
* BrowserNavigate (bounded)
* ScreenCapture (explicit permission + redaction)

Over:

* arbitrary write/delete
* arbitrary system calls

---

## 4) Permission Model Rules (Before you expand tools)

### P1 — One universal enforcement point

Pick a single mechanism to enforce permissions and apply it to **all MCP tool calls**.

* No “legacy path” half-enforced.

### P2 — Permission tokens must be time-boxed + scope-boxed

* token: {tool, scope, expiresAt, reason}
* short TTL
* minimum scope

### P3 — Permissions are user-visible and auditable

* prompts are clear
* outcomes logged

**Cursor pushback when:** permission becomes implicit, inferred, or hidden.

---

## 5) Data & Memory Rules


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [raydeStar/sir-thaddeus](https://github.com/raydeStar/sir-thaddeus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
