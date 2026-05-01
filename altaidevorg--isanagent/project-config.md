---
trigger: always_on
description: This document serves as the primary system architectural context for any future LLMs continuing to evolve the `isanagent` framework. If you are an AI tasked with writing a new tool, skill, integration, or feature for `isanagent`, please read this carefully to avoid producing anti-patterns or breaking the Actor Memory Model.
---

# Future AI Development Blueprint (isanagent)

This document serves as the primary system architectural context for any future LLMs continuing to evolve the `isanagent` framework. If you are an AI tasked with writing a new tool, skill, integration, or feature for `isanagent`, please read this carefully to avoid producing anti-patterns or breaking the Actor Memory Model.

## 🧠 System Architecture Primer

`isanagent` completely decouples standard AI sequential blocking loops into a natively concurrent **Actor System**. 
The core data structure traveling the entire network natively is `isanagent::bus::BusMessage`:

```rust
pub enum BusMessage {
    Inbound(InboundMessage),
    Outbound(OutboundMessage),
    Telemetry(TelemetryEvent),
    Log(LogEvent),
    LoggerControl(LoggerControlMessage),
    Cancel(String), // chat_id — explicit stop; clears FIFO queued inbounds for that chat
}
```

While a **main** reasoning loop is already active for a `chat_id`, additional **`Inbound`** messages for that chat are held in a **per-chat FIFO** and started only after the current turn finishes. **`BusMessage::Cancel(chat_id)`** (for example terminal **`/cancel`** / **`/stop`**) is the explicit stop path: it cancels the in-flight loop, optionally cascades to sub-agents when configured, and **drops** any queued prompts for that chat. New user text does **not** implicitly cancel an in-flight turn.

The fundamental philosophy here is **Wait-Free Threading**. Do not use `std::sync::Arc<std::sync::Mutex<rusqlite::Connection>>`. All critical I/O, especially database storage, must route through an opaque lock-free asynchronous Actor (`MemoryActor` wrapping `SqliteMemory` for instance).

### Workspace & Sandboxing
The agent is explicitly designed to run inside a sandbox. `IsanagentWorkspace` manages two boundaries:
1. `workspace_dir` (Outer Rim): Holds `config.toml`, generated logs, and `.system_generated` internal sqlite caches.
2. `sandbox_dir` (Inner Rim): This is the designated execution field, usually `workspace_dir/.agents`. This is where the Agent expects to load `AGENTS.md` and read `skills/`.

If you are writing a new `Tool` that mutates the disk or reads files, **DO NOT** let the Agent pass absolute system paths cleanly. You MUST wrap the injection using `crate::utils::resolve_path(&sandbox_dir, &agent_path`). Doing so naturally bounds all agent `../` directory escapes to the sandbox boundary.

Harness todo lists (`todo_write`) are stored in the workspace SQLite DB (same file as session memory: `<workspace_dir>/.system_generated/agent_memory.db`, table `harness_todos`). Reads and writes go through `MemoryMessage::ReplaceHarnessTodos` and `MemoryMessage::LoadHarnessTodos` on the same `SqliteMemoryActor` as session memory—never a separate mutex-wrapped connection.

User clarification (`ask_user`) sends an outbound message tagged with metadata `isanagent_clarification` and blocks the tool until the **next inbound** on the same session (`channel`, `chat_id`, optional `thread_id`). The agent routes that inbound to the waiting tool instead of starting a new reasoning task, so the model receives the reply as the tool result and continues the same turn.

Git worktrees (`git_worktree`): **off** in a minimal `config.toml` until you set `[harness.git_worktree] enabled = true`. The onboarding template under `assets/onboarding/config.toml` turns this **on** for new workspaces. Worktree paths use the same `resolve_path` sandbox rules as other filesystem tools unless `allow_path_outside_sandbox = true`, which permits canonical paths outside the sandbox (for example a host temp directory). See `docs/harness-implementation-plan.md` Phase 4.

Sub-agents (`subagent_spawn`, `task_*`, `subagent_plan_execute`, `task_history_list`): **off** in a minimal config until `[harness.subagents] enabled = true`. The onboarding template enables this for new workspaces. Sub-agents run a second `run_reasoning_loop` with a synthetic chat id (`subagent-…`) and optional tool allowlist. `cancel_children_on_parent_cancel` (default true) controls whether an **explicit** parent cancel (`BusMessage::Cancel` / API cancel / terminal **`/cancel`**) also cancels those child tasks (not triggered by queued follow-up user messages). Completed runs are recorded in the workspace SQLite table **`subagent_tasks`** (same DB as session memory). See `docs/harness-implementation-plan.md` Phase 5.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [altaidevorg/isanagent](https://github.com/altaidevorg/isanagent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
