---
trigger: always_on
description: Bun + Turborepo monorepo. Backend **Elysia**, frontend **Next.js (SSR)** + **shadcn/ui**,
---

# AGENTS.md

Bun + Turborepo monorepo. Backend **Elysia**, frontend **Next.js (SSR)** + **shadcn/ui**,
auth **better-auth**, ORM **Drizzle** + PostgreSQL. Self-hosted with Docker Compose;
Coolify is the deploy target of the reference instance.

Prefer the most specific `AGENTS.md` for the area being changed: every app and some packages
have their own, and it wins over this file. The `CLAUDE.md` next to it is a pointer, not a
second source.

## Core Coding Principles

**IMPORTANT — always follow, no exceptions:** Follow KISS and YAGNI. Write the simplest
thing that works, avoid premature abstractions, and don't add them until multiple concrete
implementations actually need them. No overengineering.

Before writing new code, study the existing codebase and follow its established patterns and
conventions when they are sound. Reuse existing shared modules, components, and utilities
instead of duplicating or reinventing them; extend what's there rather than adding parallel
solutions.

## Writing style (docs, comments, chat)

Write plainly and literally. This applies to all prose: documentation, code comments, commit
messages, and chat replies.

- No metaphors, no personification, no figurative language. Do not write "the parser is the
  heart of the step", "data lives in Postgres", "self-healing queue", "the ticket flows through
  the pipeline". State the fact directly: "the parser is the main task of the step", "data is
  stored in Postgres", "a failed donor is retried on the next run".
- No rhetorical flourishes or dramatization. Describe what something does, not how important or
  elegant it is.
- Technical terms that happen to be metaphors in origin (watermark, backlog, queue, pipeline)
  are fine — they are standard terminology, not stylistic choices.
- Prefer short, direct sentences over expressive ones.

**Write the final state, not the edit history.** When the user changes a decision, rewrite
docs/comments to describe only the current design. Do not record the transformation of their
thinking.

- Do not write "not X, but Y", "now in the DB instead of in code", "decided", "moved from a
  separate step", "previously we planned". Just state Y.
- Do not annotate a choice with why it replaced an earlier idea from this conversation. The
  reader wants the current requirement, not how we got here.
- Forward-looking design rationale ("column `project_key` is added now so steps 3–4 need no
  migration") and cross-references between sections are fine — they explain the current design,
  not a discarded one.
- A discarded idea is only worth mentioning if it is a lasting "considered and rejected
  because…" that a future reader would otherwise re-propose — and then state it once, plainly,
  not as a correction to a prior draft.

## Golden rules

- **Runtime is Bun**, never npm/yarn/pnpm. Install with `bun install`, run scripts with `bun run`.
- **All code, comments, and strings are in English.**
- Cross-package imports use the workspace protocol: `"@repo/db": "workspace:*"`. Packages
  export raw `.ts` (see each package's `exports`) — Bun and Next transpile them, no build step.
- Run tasks from the **repo root** via Turborepo: `bun run dev` / `build` / `typecheck`.

## Layout

```
apps/api        Elysia (Bun) — mounts better-auth at /api/auth/*        :3000
apps/web        Next.js App Router, SSR (not SPA) + shadcn + TanStack Q :3001
apps/worker     webhook and notification delivery, agent runs, schedules
apps/bot        Telegram bot, long polling
packages/db     @repo/db     — Drizzle client, schema, migrations
packages/auth   @repo/auth   — better-auth server instance + instance auth settings
packages/crypto @repo/crypto — AES-256-GCM encryption for secrets at rest
packages/mailer @repo/mailer — SMTP/Resend transport for outbound email
packages/agent-tools @repo/agent-tools — tool definitions for the AI agent runtime
packages/eslint-config @repo/eslint-config — shared ESLint config
```

Dependency graph: `api → @repo/auth → @repo/db`. **The web app never imports packages
directly** — it talks to the API over HTTP (better-auth client + fetch).

## Commands (from root)

| Command                   | Purpose                                                          |
| ------------------------- | ---------------------------------------------------------------- |
| `bun run dev`             | api + web in watch mode (turbo)                                  |
| `bun run build`           | build all apps                                                   |
| `bun run typecheck`       | tsc across the workspace                                         |
| `bun run lint`            | ESLint across the workspace                                      |
| `bun run format:check`    | Prettier check (CI runs the same)                                |
| `bun run test`            | run each app's test suite (turbo)                                |
| `bun run db:generate`     | generate SQL migrations from Drizzle schema                      |
| `bun run db:migrate`      | apply migrations                                                 |
| `bun run db:migrate:test` | apply migrations to the test DB (`.env.test`)                    |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [croffasia/itsaplan](https://github.com/croffasia/itsaplan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
