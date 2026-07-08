---
trigger: always_on
description: This file is for Claude Code and other coding agents working inside this repository. For product context, first read `README.md`, `AGENTS.md`, `docs/文档索引与权威口径.md`, `docs/当前状态与下一步路线.md`, `docs/AgentHub-HiClaw-lite开源内核重构方案.md`, and `docs/HiClaw架构调研与AgentHub底层重构方案.md`.
---

# AgentHub Development Guide

This file is for Claude Code and other coding agents working inside this repository. For product context, first read `README.md`, `AGENTS.md`, `docs/文档索引与权威口径.md`, `docs/当前状态与下一步路线.md`, `docs/AgentHub-HiClaw-lite开源内核重构方案.md`, and `docs/HiClaw架构调研与AgentHub底层重构方案.md`.

## Agent skills

### Issue tracker

Issues and PRDs are tracked in GitHub Issues for `metrogg/AgentHub`. See `docs/agents/issue-tracker.md`.

### Triage labels

Use the default five-role triage vocabulary: `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, and `wontfix`. See `docs/agents/triage-labels.md`.

### Domain docs

This is a single-context repo for skill consumption; the authoritative domain context currently lives in the existing project docs rather than a root `CONTEXT.md`. See `docs/agents/domain.md`.

## Product Definition

AgentHub is an open-source Coze/Kimi-style AI work platform shell evolving toward a HiClaw-lite open collaboration kernel. The expected behavior is:

1. The user starts from a group chat.
2. Manager / Orchestrator behaves like a team lead: it understands the request, decides whether to reply, clarify, request members, assign work, review, or summarize.
3. Planner is not the brain. Structured decomposition is only a planning skill/action that Manager may call.
4. Multiple agents receive concrete tasks in their own child conversations.
5. The main group chat shows Manager progress, member reports, artifacts, and final synthesis.
6. Users can open child conversations to inspect each agent's real execution trace.

Do not implement fixed scenario templates as the core path. The platform must stay general-purpose first.
Role presets may be used as a manual creation library, but they must not auto-seed a workspace, define a default team, or override model-generated assignments.
Role prompts should follow the composition model: shared collaboration protocol + role background + bound skills + runtime task context + output contract. Group goals may drive member recommendations, but not fixed execution templates. If an existing group lacks needed capability, Orchestrator may propose adding a new agent; this must be visible and user-approved by default.
Preinstalled agent templates and lightweight expert-team recommendations are agent configuration assets, not fixed execution teams. You may borrow structure from Claude Code subagents, BMAD, SuperClaude, awesome-cursor-skills, and MCP server ecosystems, but first adapt for license, safety boundaries, quality, and AgentHub schemas. Do not build a separate "my experts" system or full expert marketplace yet, and do not directly copy unaudited prompts or enable third-party MCP servers by default.

## Current Kernel Direction

The new architecture target is **AgentHub Product Shell + HiClaw-lite Open Kernel**:

- Keep AgentHub's own product UI: group chat, task rooms, experts, task board, artifact cards, settings, trace/eval.
- Use Matrix as the internal collaboration source of truth: Room, timeline, participant, and mention are first-class.
- Learn Manager Runtime from HiClaw OpenClaw: Manager is a real coordinator runtime, not a one-shot Planner function.
- Learn Worker Runtime from HiClaw, while keeping AgentHub's coding-agent advantage: Claude Code, OpenCode, Codex, and Gemini are core Worker bases.
- Use local filesystem SharedStorage as the default lightweight contract store, but keep S3-compatible object key semantics so MinIO/S3 can be swapped in later.
- Borrow lightweight implementation tactics from ClawTeam: CLI profiles/adapters, git worktree isolation, task claim locks, LeaderWatcher-style snapshot diff, profile doctor/test, and server-side board snapshots.
- Abstract AI Gateway. Short term: LocalGateway/LiteLLM. Long term: Higress-style model/MCP/credential governance.
- A2A is no longer the first-stage internal communication backbone. Keep it for external interoperability or optional task semantic envelopes inside Matrix events.
- This is still development. Old sessions, tasks, database rows, and workspace/storage data are not architecture constraints. Prefer clearing/rebuilding old data over preserving old execution paths.

The four highest-priority kernel modules are:

1. Manager coordinator: runtime, persona config, skills, state, worker registry, Room communication, heartbeat/patrol.
2. Worker runtime: real participant identity, model binding, skills/MCP scope, heartbeat, sleep/wake/stop, runtime lease.
3. Matrix communication: real Matrix homeserver Room/timeline/participant/mention instead of session metadata or local tables pretending to be rooms. Prefer Tuwunel; keep Synapse/Conduit compatible.
4. Shared storage: filesystem-first ArtifactStore/SharedStorage with canonical `shared/tasks/{taskId}/...` object refs and S3-compatible object key semantics.

Use `docs/hiclaw-wiki.agent.final.md` and local `hiclaw源码参考/` as the main HiClaw reference materials.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [metrogg/AgentHub](https://github.com/metrogg/AgentHub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
