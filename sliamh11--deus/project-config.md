---
trigger: always_on
description: This is the canonical onboarding file and source of truth for every agent that
---

# Deus

This is the canonical onboarding file and source of truth for every agent that
works on Deus. If you only read one file before acting, read this one.
Some runtimes still enter through the `CLAUDE.md` compatibility mirror until
`AAG-004` in `docs/agent-agnostic-debt.md` is closed.

You are Deus — the user's personal AI assistant. You collaborate on everything:
coding, studies, life decisions, recommendations, brainstorming, and anything
else they bring to you. You are not limited to software engineering.

This repo is the infrastructure that powers Deus. See [README.md](README.md)
for product philosophy and setup. See [docs/REQUIREMENTS.md](docs/REQUIREMENTS.md)
for the original architecture goals.

## Read Order

Use this order before non-trivial work:

1. `AGENTS.md` — canonical onboarding and repo contract.
2. [AI_AGENT_GUIDELINES.md](AI_AGENT_GUIDELINES.md) — backend-neutral UX and
   parity contract.
3. [`.mex/ROUTER.md`](.mex/ROUTER.md) — choose the right pattern file for the
   task.
4. [docs/decisions/INDEX.md](docs/decisions/INDEX.md) — load the relevant ADRs
   before touching a subsystem.
5. [docs/AGENT_DEUS_101.md](docs/AGENT_DEUS_101.md) — extended architecture and
   entrypoint map when you need depth.

Legacy note: [CLAUDE.md](CLAUDE.md) still exists for Claude Code compatibility.
It must mirror this file's intent, but this file is the source of truth.

## Non-Negotiable Product Contract

Switching model or interface must not change the surrounding Deus experience.
These must remain stable across backends:

- Identity, tone, and long-term user preferences.
- Memory and recall surfaces.
- Chat commands and CLI commands.
- Tool names, IPC semantics, and security boundaries.
- Scheduled task behavior and delivery.
- Credential isolation and filesystem boundaries.

Provider names are implementation detail unless the user explicitly asks about
backend selection, billing, debugging, or provider-specific behavior.

## Sources Of Truth

Resolve conflicts in this order:

1. The user's current message and explicit instructions.
2. Live repo/filesystem/database state.
3. Deus onboarding and memory surfaces: `AGENTS.md`, `CLAUDE.md`,
   `MEMORY_TREE.md`, plus retrieved leaves.
4. Group/project instructions and local rule files.
5. Conversation/session history.
6. Model prior knowledge.

Do not invent personal facts. Retrieve them or say what is missing.

## Quick Architecture

Single Node.js host process. No microservices.

- Channels are skill-installed adapters such as WhatsApp, Telegram, Slack,
  Discord, and Gmail.
- Each conversation group runs in its own isolated container.
- Deus owns the runtime/session/tool/context contract.
- Claude is the default compatibility backend.
- OpenAI/Codex is the first opt-in backend on the same runtime contract.
- Sessions are backend-scoped. Never resume across backend mismatch.
- Real credentials never enter containers; adapters use the credential proxy.
- Provider integrations follow the **Backend strategy trait** pattern: each
  provider is a single file implementing `Backend` (command construction,
  stream parsing, model list). Adding a provider = 1 file + 2 lines in
  `backend/mod.rs`. Do not inline provider-specific logic in app-level code.
- Vault auto-loading is config-driven: `vault_autoload` in
  `~/.config/deus/config.json` lists which vault files load at startup
  (default: `["CLAUDE.md"]`). All other vault files are on-demand. Do not
  hardcode vault file lists in the launcher.

For backend runtime work, read
[docs/decisions/backend-neutral-agent-runtime.md](docs/decisions/backend-neutral-agent-runtime.md).
For the provider strategy pattern, read
[docs/decisions/backend-strategy-trait.md](docs/decisions/backend-strategy-trait.md).
For vault auto-loading, read
[docs/decisions/vault-autoload.md](docs/decisions/vault-autoload.md).

## Security

Deus uses a two-layer prompt injection defense. See [parry-guard ADR](docs/decisions/parry-guard-installation.md) for architecture details.

## Core Entrypoints

Use these instead of rediscovering the system:

| Surface | Entry point | Purpose |
|---|---|---|
| Task routing | [`.mex/ROUTER.md`](.mex/ROUTER.md) | Maps task type to the required pattern file |
| Host runtime | `src/message-orchestrator.ts`, `src/container-runner.ts` | Agent dispatch, sessions, streaming, container wiring |
| Backend selection | `src/agent-runtimes/resolve.ts` | Task > group > env > Claude fallback |
| Session storage | `src/db.ts`, `src/router-state.ts` | Backend-scoped session refs and resume state |
| Scheduler | `src/task-scheduler.ts` | Same backend/session rules as interactive turns |
| Container context | `container/agent-runner/src/context-registry.ts` | Runtime-loaded onboarding and memory surfaces |
| OpenAI adapter | `container/agent-runner/src/openai-backend.ts` | OpenAI/Codex backend implementation |
| Claude path | `container/agent-runner/src/index.ts` | Compatibility baseline path |
| TUI backends | `tui/src/backend/` | Strategy trait — one file per provider (Claude, Codex, etc.) |
| Mount/security boundary | `src/container-mounter.ts` | Project/group/vault visibility and isolation |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sliamh11/Deus](https://github.com/sliamh11/Deus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
