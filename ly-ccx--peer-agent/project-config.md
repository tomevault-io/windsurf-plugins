---
trigger: always_on
description: This file governs the entire repository. All future code changes in this repository must follow these rules unless the user explicitly asks to change the architecture decision itself.
---

# Peer Agent Development Rules

This file governs the entire repository. All future code changes in this repository must follow these rules unless the user explicitly asks to change the architecture decision itself.

## Architecture Baseline

Peer Agent follows the 端云能力代理设计原则:

```text
模型负责认知。
本地负责能力。
界面负责表达。
契约负责边界。
证据负责治理。
```

`模型` names a role, not a deployment location: cognition may come from a cloud provider or from a future on-device runtime. Capability execution, authorization, and Evidence remain local either way.

Architecture knowledge lives in the companion knowledge base `peer-knowledge` (not in this code repo). The detailed governance baseline is in `peer-knowledge/knowledge/architecture/20-architecture-governance.md`. Existing architecture context is in:

- `peer-knowledge/knowledge/architecture/00-engineering-philosophy.md`
- `peer-knowledge/knowledge/architecture/01-project-structure.md`
- `peer-knowledge/knowledge/decisions/15-plugin-skill-mcp-system.md`
- `peer-knowledge/knowledge/decisions/16-skill-call-lifecycle.md`
- `peer-knowledge/knowledge/architecture/19-system-prompt-context-architecture.md`

These documents are part of the engineering contract, not background reading. Treat them as read-only reference material by default.

Companion knowledge identity (not a machine path):

- Remote: `https://github.com/ly-ccx/Peer-Knowledge`
- Local directory aliases: `Peer-Knowledge`, `peer-knowledge`

Resolve the knowledge root in this order. Never commit a user-specific absolute path.

1. Environment variable `PEER_KNOWLEDGE_ROOT` if it points at an existing directory.
2. Optional gitignored file `.peer-workspace.local.json` in this repo root (`peerKnowledgeRoot`).
3. A sibling directory of this repo that matches a knowledge alias.
4. An already-open multi-root workspace folder that is the knowledge repo.

If none resolve, ask. Do not invent a home-directory path or write one into git.

## Workspace Entry

This code repo is the default daily entry. Open it as the primary workspace root. Mount `peer-knowledge` beside it when review or durable knowledge writes are needed.

- C-level: edit this repo directly.
- B-level: edit this repo and state the architecture impact. Write back to `peer-knowledge` if a boundary actually moved.
- A-level: review and update the architecture document or ADR in `peer-knowledge` first, then implement here.

Starting a session in the knowledge repo does not make the knowledge repo the delivery target. Commits follow the change: code here, decisions there. The short intake playbook is `peer-knowledge/playbook/change-intake.md`.

## User Repository Preference: Architecture Docs Live in peer-knowledge

Do not recreate, stage, or submit architecture / design knowledge under `docs/architecture/*` or other knowledge trees in this code repo unless the user explicitly asks. If a code change would normally require architecture documentation, explain the architecture impact in the response and write durable knowledge into `peer-knowledge` instead.

## Non-Negotiable Runtime Chain

All local capabilities must flow through:

```text
Capability Provider
  -> Manifest
    -> Runtime Projection
      -> Tool Call
        -> PermissionGrant
          -> Evidence
```

Do not create separate execution paths for Bash, file operations, MCP, Plugin, Skill, attachments, app automation, or future multimodal capabilities.

## Layer Rules

- The desktop client owns local discovery, local authorization, local execution, and Evidence return.
- Renderer owns presentation and user interaction only.
- Protocol owns cross-layer contracts.
- Evidence owns factual accountability.
- System Context owns context admission, not tool execution.

## System Context Rules

- System prompt construction must be treated as System Context assembly, not ad hoc string concatenation.
- New project instructions, mode reminders, tool prompts, compact summaries, Skill hints, Plugin hints, and MCP capability hints must enter through an explicit Context Source or documented temporary adapter.
- Tool output, file content, webpage content, and attachment content are factual/user context. Do not promote them to system instructions.
- Provider-specific request formatting belongs behind a provider encoder seam. Do not scatter OpenAI / Anthropic message-shape decisions across unrelated modules.
- Compact summaries are continuity context only. They do not replace Tool Result, Evidence, artifact refs, or rerunnable retrieval hints.

## Hard Bans

- Do not let renderer directly use `fs`, `child_process`, MCP lifecycle, or secret storage.
- Do not store permission truth only in renderer state.
- Do not render textual `[Tool call]` or `[Tool result]` claims as real tool execution.
- Do not let assistant text stand in for structured Tool Result or Evidence.
- Do not bypass Runtime Projection when exposing model-visible tools.
- Do not rely on prompt instructions as the only enforcement for permissions or capability limits.
- Do not add ad hoc cross-process payloads when a protocol object should exist.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ly-ccx/Peer-Agent](https://github.com/ly-ccx/Peer-Agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
