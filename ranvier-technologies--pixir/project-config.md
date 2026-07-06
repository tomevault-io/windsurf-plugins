---
trigger: always_on
description: Pixir is an OTP-native, local-first coding-agent harness. The runtime spine is:
---

# AGENTS.md - Pixir Harness

Pixir is an OTP-native, local-first coding-agent harness. The runtime spine is:

```text
Session -> Turn -> Provider -> Tools
```

The append-only Log is truth, the terminal CLI is the default presenter, and ACP over
stdio is the main client transport.

## Progressive Discovery

Do not read the whole repo up front. Load context in this order and stop once the task
is grounded enough:

1. **Start here:** this file gives the architecture map, invariants, commands, and beta
   stance.
2. **Human quickstart:** `README.md` and `docs/open-beta-quickstart.md` explain the
   source-install developer-preview path.
3. **Vocabulary:** `CONTEXT.md` defines Harness, Session, Agent, Skill, Workflow,
   Subagent, Event, Log, History, Workspace, Tool, Host Boundary Crossing, Provider,
   Credential, and PATCHMD. Use those meanings exactly.
4. **Architecture decisions:** read only the ADRs relevant to your change:
   - `0001` Session is the process; Agent is role/configuration.
   - `0002` ChatGPT subscription OAuth + OpenAI Responses API.
   - `0003` stateless Turns; local Log is source of truth.
   - `0004` unified Event envelope and canonical vs ephemeral events.
   - `0005` agent ergonomics: dry-run, help, structured errors, I/O discipline.
   - `0006` permission model.
   - `0007` encrypted reasoning item persistence/replay.
   - `0008` UI-agnostic Conversation driver.
   - `0009` ACP transport over stdio.
   - `0010` Agent Skills with progressive disclosure.
   - `0011` BEAM-native Subagents.
   - `0012` structural Workflows over Subagents.
   - `0013` Skill-backed Workflow Templates.
   - `0014` Workflow Checkpoint Bundles and honest partial outcomes.
   - `0015` PATCHMD customization maintenance.
   - `0016` open beta scope as source-install developer preview.
   - `0017` minimal Harness core and Presenter boundary.
   - `0018` durable History compaction and replay repair.
   - `0019` Provider usage, prompt-cache observability, and WebSocket continuation.
   - `0020` versioned Prompt Contract, cache-key family, and compaction triggers.
   - `0021` Session Resources and Image Attachments.
   - `0022` Provider-hosted Web Search evidence, not local Tool execution.
   - `0023` Skill Context Hydration as explicit, canonical, late-bound context.
   - `0024` Session Fork and Branch Summaries.
   - `0025` Hex package scope.
   - `0026` Runtime terminal-state and replay contract.
   - `0027` external command execution as a bounded host boundary.
   - `0028` Workspace Strategies and future virtual overlays.
   - `0029` Virtual Overlay changes export `virtual_diff` artifacts.
   - `0030` explicit apply and merge-back semantics for `virtual_diff`.
   - `0031` Git worktrees as lease-owned strategy for intended repo changes.
   - `0032` minimal Workflow Events for durable run decisions.
   - `0033` typed checkpoint outputs as harness-owned projections.
   - `0034` Delegate service runtime residency before async start.
   - `0035` Write-capable Sessions require an external evidence mirror.
   - `0036` Idle-timeout recovery does not auto-resume ambiguous work.
5. **Nearest local instructions:** read the closest subtree `AGENTS.md` before editing
   code, docs, tests, benchmarks, or ADRs.

## Screaming Architecture

The repo should reveal what Pixir is by its module names:

- `Pixir.Event` / `Pixir.Events` - Event envelope and pub/sub bus.
- `Pixir.Log` - append-only NDJSON under `.pixir/sessions/`.
- `Pixir.Session` / `Pixir.SessionSupervisor` - one process per conversation.
- `Pixir.Turn` - provider/tool loop for one user Turn.
- `Pixir.Conversation` - UI-agnostic multi-turn driver.
- `Pixir.Provider` - OpenAI Responses API dialect, stateless with `store: false`.
- `Pixir.Provider.{Connection,TransportPolicy,WebSocketClient}` - WebSocket-preferred
  transport policy and HTTP/SSE fallback. This is optimization plumbing; it never
  replaces local Logs.
- `Pixir.Provider.{Cache,HostedTools}` - prompt-cache metadata and Provider-hosted tools
  such as Web Search. Hosted tools are Provider evidence, not Pixir local Tool calls.
- `provider_usage` Events - durable token/cache evidence for each Provider call; never
  replayed as model context.
- `Pixir.Auth` - ChatGPT subscription OAuth and API-key fallback.
- `Pixir.Tools.*` / `Pixir.Tools.Executor` - permissioned, workspace-confined tools
  and the first enforcement boundary for Host Boundary Crossing.
- `Pixir.SessionResources` - durable local resources such as Image Attachments and ACP
  resource links; Provider image/file inputs are projections of these resources, not the
  resources themselves.
- `Pixir.Skills` / `Pixir.Agents` / `Pixir.Subagents` / `Pixir.Workflows` -
  installed practices, role configs, supervised child Sessions, and structural
  orchestration.
- `Pixir.SessionTree` - read-only projection of Session/Subagent hierarchy from Logs.
- `Pixir.Compaction` - durable `history_compaction` checkpoints and replay repair.
- `Pixir.CLI` / `Pixir.Renderer` - terminal presenter.
- `Pixir.ACP.*` - ACP stdio presenter. stdout is JSON-RPC only.

## Beta Stance

Pixir Harness is public as an early source-install developer preview:

- Supported: source build, CLI, ACP stdio, OpenAI Responses provider, ChatGPT

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ranvier-Technologies/pixir](https://github.com/Ranvier-Technologies/pixir) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
