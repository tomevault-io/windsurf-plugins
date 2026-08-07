---
trigger: always_on
description: > Natural-language-driven Agent orchestration. See [PRODUCT.md](./docs/PRODUCT.md) for vision & roadmap.
---

# a2wave — Project Guide

> Natural-language-driven Agent orchestration. See [PRODUCT.md](./docs/PRODUCT.md) for vision & roadmap.

> **Primary language: English.** This is an OSS-facing repository — all code, comments, commit messages, documentation, and identifiers must be written in English.

> **Trust model — an internal enterprise team.** a2wave assumes **Agent authors
> and Agent users are all trusted colleagues acting in good faith**. Agents run
> the underlying CLIs with real capabilities (filesystem, shell, injected
> credentials) *by design*; the platform does not sandbox trusted authors from
> each other, nor defend against a malicious insider deliberately building a
> hostile Agent.
>
> Security controls — authentication, per-Agent owner/editor/viewer permissions,
> audit logging, rate limiting, per-run credential injection — enforce
> **accountability and least privilege among cooperating teammates**, not
> containment of an adversary inside the trust boundary.
>
> **What this means when working in this repo:** do not file, "fix", or harden
> against threats that only arise from a hostile authenticated author — that is
> outside the design, and such changes add complexity while protecting nobody.
> This does *not* relax the controls that do apply: never skip authentication,
> never allow anonymous invocation, never drop an audit entry, and never put
> credentials in `details` (Iron Rule 5). Deployments exposing a2wave to
> untrusted users must add their own isolation layer.
>
> Full statement: [SECURITY.md](./SECURITY.md) · [docs/PRODUCT.md](./docs/PRODUCT.md)

## Architecture

```
a2wave (pnpm monorepo)
├── apps/api/         # Hono + SQLite (Drizzle ORM) + Local Agent Execution
├── apps/web/         # React 19 + Vite + TailwindCSS v4 + Ant Design
├── apps/cli/         # CLI tool (a2wave command)
└── packages/shared/  # Zod schemas & types
```

Stack: TypeScript, Biome (lint), TanStack Query, React Router v7

## Product Identity & Iron Rules

a2wave is a general-purpose Agent building and orchestration platform for enterprises. It builds on mature agent CLIs such as Cursor Agent / Claude Code / OpenAI Codex, extends capabilities via Skills and MCP Servers, and publishes Agents over API / Feishu / Slack / Discord / A2A / scheduled / chat page / GitLab repository trigger / GitHub repository trigger.

The following Iron Rules define the product boundary; every new feature must be checked against them first:

| # | Iron Rule | Description |
|---|------|------|
| 1 | **Orchestrate, don't execute** | a2wave is the orchestration layer; execution capability comes from the underlying agent CLIs (Cursor/Claude Code/Codex). Do not build our own LLM inference, code execution, or sandbox runtime. |
| 2 | **Extend through composition** | New capabilities are delivered by combining Skills + MCP Servers, not by hardcoding business logic into the platform core. If a feature can be solved with a Skill or MCP, it should not become a built-in platform feature. |
| 3 | **Natural-language-driven, not flow-driven** | Agents are configured and orchestrated in natural language — prompts, intents, and A2A messages. No drag-and-drop DAG editor, no traditional workflow primitives like variable mapping or conditional branches. |
| 4 | **Agent autonomy — the platform does not intervene in execution details** | The platform creates, configures, triggers, and monitors Agents; it does not interfere with an Agent's runtime reasoning or tool-call decisions. No "step approval", "manual checkpoints", or other flow controls that break Agent autonomy. |
| 5 | **Enterprise-grade constraints, scoped by the trust model** | Security (AUTH_SECRET, rate limiting), auditability (Run records; for background work that deliberately writes none, an equivalent audit-log entry — see Evaluation), and operability (health checks, logs) are hard requirements. Never sacrifice infrastructure for "quick trial" experiences. No anonymous invocation; never skip authentication. But the goal is **accountability and least privilege among trusted colleagues** (see the trust model above), *not* containment of a hostile insider — do not harden against threats that only a malicious authenticated author could pose. |

> **For feature requests that violate the Iron Rules, contact the maintainers for confirmation before proceeding.**

## Core Concepts

Agent, Provider, MCP Server (stdio/sse/http/group), Skill, SCM Source, Run, ChatMessage, Settings, Evaluation Set / Case / Task.
A Skill is creator-private by default (`visibility = private`); only an administrator may publish it as `all-users`, after which every signed-in user can discover and bind it while mutations remain owner/admin-only. Platform-seeded built-in Skills are system-owned and persist as `all-users` so every signed-in user can discover, bind, clone, and authenticated-export them; public share exports still omit all Skill content.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LilithGames/a2wave](https://github.com/LilithGames/a2wave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
