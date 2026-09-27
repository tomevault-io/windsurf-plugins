---
trigger: always_on
description: Guidance for AI coding agents working in this repository.
---

# AGENTS.md

Guidance for AI coding agents working in this repository.

## Project overview

Kody, a GitHub maintainer agent built on the [eve](https://eve.dev) agent framework. A weekly schedule composes a digest email of the configured repo's open issues and sends it through the **Resend** MCP connection; recipients reply to act on it (for example "create Linear issues for #1 and #2"), and replies come back in through the **resend** channel. Kody reads and triages GitHub issues through the **github** tools (GitHub Tools SDK via Vercel Connect), creates and cross-references issues through the **Linear** MCP connection, and answers when @mentioned on GitHub issues/PRs or in Linear Agent Sessions. Per-user preferences live in **Vercel Blob**. Its workflow lives in `agent/instructions.ts`.

The whole agent is defined under `agent/`. eve discovers capabilities from the filesystem. See [`ARCHITECTURE.md`](./ARCHITECTURE.md) for the component map, data flow, and boundaries.

## Setup & commands

```bash
pnpm install        # install dependencies (Node 24.x)
pnpm dev            # eve dev — local TUI; run /model once to link a model provider
pnpm typecheck      # tsc (TypeScript, no emit)
pnpm check          # ultracite (Biome) lint + format check
pnpm fix            # ultracite (Biome) auto-fix
pnpm build          # eve build
eve deploy          # deploy to Vercel production (use this, not raw `vercel deploy`)
npx eve info        # print the discovered surface + discovery diagnostics
pnpm validate       # check + typecheck + eve info in one command
```

There is no unit-test suite. **Verify changes with `pnpm validate` (lint, typecheck, and discovery diagnostics must all report 0 errors / 0 warnings), then exercise the agent in the `pnpm dev` TUI.**

## eve conventions

- **Read the relevant guide in `node_modules/eve/docs/` before writing code.** Don't invent framework APIs; confirm them against the docs.
- **Identity comes from the filesystem, never a `name` field.** A tool at `agent/tools/github.ts` is the tool `github`; a connection at `agent/connections/linear.ts` registers as `linear`.
- Authored slots: `agent/agent.ts` (model), `agent/instructions.ts` (`defineInstructions`, the system prompt; resolved at build time, injecting `RESEND_FROM_ADDRESS` into the sending-email rule), `agent/tools/*.ts` (`defineTool`), `agent/connections/*.ts`, `agent/channels/*.ts`, `agent/schedules/*.ts` (cron-triggered sessions), `agent/skills/<name>/SKILL.md`, `agent/subagents/<id>/agent.ts` (`defineAgent`), `agent/sandbox.ts`.
- **Channels:** `github` (eve GitHub channel via Vercel Connect, botName "Kody"; a custom `onComment` hook keeps the built-in mention and ignore rules but dispatches only when the commenter's `author_association` is OWNER, MEMBER, or COLLABORATOR, so untrusted accounts can't drive the agent; an `onPullRequest` hook posts a summary comment with a changed-files table on every newly opened PR, skipping bot authors but deliberately not association-gated since summarizing outside PRs is the point), `linear` (eve Linear channel via Connect; Linear Agent Sessions, where users delegate or mention the agent; a custom `onAgentSession` hook injects the requester's email as session context), `resend` (Chat SDK channel using `@resend/chat-sdk-adapter` with Redis state and streaming off; email in and out), plus the `eve` route-auth channel.
- **Tools** run in the app runtime (full `process.env`), one default export per file. Gate destructive tools with `approval` from `eve/tools/approval` (here: `clear_user_preferences`). The `github` tool exposes the GitHub Tools SDK's `maintainer` preset (`connectGithubTools`) with credentials via Vercel Connect; issue-conversation writes (`addIssueComment`, `addLabels`, `closeIssue`, `createIssue`, `removeLabel`) set `requireApproval: "never"` because the email surface cannot render an approval prompt, while higher-impact writes keep the SDK's approval-by-default.
- **Connections** are MCP servers: `linear` (`https://mcp.linear.app/mcp`, app-scoped auth shared through `linearAuth` in `agent/lib/constants.ts` with scopes `read`, `write`, `issues:create`, `comments:create`) and `resend` (`https://mcp.resend.com/mcp`, static bearer token from `RESEND_API_KEY` via `auth.getToken` — a static token works in every session including the cron run, which has no signed-in user, and the Resend connector cannot issue app-scoped tokens; list/send emails, templates, contacts). Connections accept the same `approval` field as tools if a write needs gating; neither passes one today.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vercel-labs/kody-eve-template](https://github.com/vercel-labs/kody-eve-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
