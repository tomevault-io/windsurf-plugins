---
trigger: always_on
description: AgentDeck provides a local-first, unified AI engineering control plane for autonomous coding agents. It orchestrates background agent processes, captures real-time Chain of Thought (CoT) reasoning, routes Model Context Protocol (MCP) tool calls, enforces security approval gates, and streams live IDE chat and telemetry over an encrypted Tailscale WireGuard mesh.
---

# Agent Integration & Control Plane Specification

AgentDeck provides a local-first, unified AI engineering control plane for autonomous coding agents. It orchestrates background agent processes, captures real-time Chain of Thought (CoT) reasoning, routes Model Context Protocol (MCP) tool calls, enforces security approval gates, and streams live IDE chat and telemetry over an encrypted Tailscale WireGuard mesh.

---

## 1. Unified Architecture & Adapter Abstraction

All agent integrations implement the standard `AgentAdapter` trait in Rust (`src/agents/trait.rs`), which decouples process lifecycle, structured stream parsing, and capability negotiation from the transport layer.

```
┌──────────────────────────────────────────────────────────────────┐
│                      AgentDeck Mobile Client                     │
│    (Live Chat, CoT Stream, Tool Diffs, Approvals, Remote Host)   │
└─────────────────────────────────┬────────────────────────────────┘
                                  │ Tailscale Mesh (100.x.y.z:8765)
                                  ▼
┌──────────────────────────────────────────────────────────────────┐
│                      agentdeckd Daemon Engine                    │
│ ┌──────────────────────────────────────────────────────────────┐ │
│ │ AgentManager (Process Lifetime, Stdin/Stdout, Crash Recovery)│ │
│ └───────────────────────────────┬──────────────────────────────┘ │
│                                 │
│      ┌──────────────────────────┼──────────────────────────┐     │
│      ▼                          ▼                          ▼     │
│ ┌───────────────┐        ┌───────────────┐        ┌────────────┐ │
│ │ Antigravity   │        │ Claude Code   │        │ Gemini CLI │ │
│ │ Adapter (agy) │        │ Adapter       │        │ & Ollama   │ │
│ └───────┬───────┘        └───────┬───────┘        └─────┬──────┘ │
│         │                        │                      │        │
│         ▼                        ▼                      ▼        │
│ ┌──────────────────────────────────────────────────────────────┐ │
│ │ Model Context Protocol (MCP) & Built-in Tool Dispatch Bridge │ │
│ │ (File Edits, Shell Exec, Browser Automation, Search, Git)    │ │
│ └───────────────────────────────┬──────────────────────────────┘ │
│                                 │
│ ┌───────────────────────────────┴──────────────────────────────┐ │
│ │ Security & Approvals Engine (Interactive Push Auth Gates)    │ │
│ └──────────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────────┘
```

### Core Responsibilities of an `AgentAdapter`
1. **Lifecycle Management**: Spawn, supervise, keep-alive, pause, resume, and terminate agent subprocesses across host OS environments (macOS, Linux, Windows).
2. **Chain of Thought (CoT) Interception**: Stream continuous model reasoning, planning milestones, and subagent delegation directly to mobile clients before actions execute.
3. **MCP Tool Routing & Diff Generation**: Parse and execute Model Context Protocol (MCP) and native tool requests, capturing file diffs (`+` / `-`), command outputs, and exit codes.
4. **Approval Gate Integration**: Intercept destructive or sensitive tool actions (e.g. `rm -rf`, force push, credential access) and hold execution until user approves via mobile push.
5. **State & Conversation Continuity**: Support conversational resumption (`--continue` or session ID references) across disconnections and reboots.

---

## 2. Supported AI Coding Agents

### 1. Google Antigravity (`agy` / Antigravity IDE Engine) — First-Class Priority Integration

Google Antigravity is the primary autonomous software engineering agent supported by AgentDeck.

- **CLI Binary**: `agy` (Discovered via `PATH` or `~/.gemini/antigravity/bin/agy`)
- **Execution Mode**: Structured JSON streaming (`--output-format stream-json`)
- **Continuation Flag**: `agy --continue` or `agy --conversation <id>`
- **Live IDE Brain Path**: `~/.gemini/antigravity-ide/brain/<conversation-id>/.system_generated/logs/transcript.jsonl`

#### Antigravity Chain of Thought & Reasoning
- Emits real-time `THINKING` steps containing model analysis, problem decomposition, and execution planning.
- Mobile client displays an animated reasoning indicator with collapsible markdown thoughts and stage checkpoints.
- Supports reasoning effort configuration (`low`, `medium`, `high`).

#### Remote Antigravity Auth & Account Switching
- Reads and updates active Google OAuth profiles in `~/.gemini/google_accounts.json` and `~/.gemini/oauth_creds.json`.
- Mobile users can switch between stored Google accounts with 1-tap, link new accounts, test auth status, and inspect remaining Gemini / Claude token quotas in real time.

#### Antigravity Tool Protocol & File Edit Streaming
AgentDeck maps all Antigravity tool invocations into structured mobile widgets:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [darknecrocities/Agentdeck](https://github.com/darknecrocities/Agentdeck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
