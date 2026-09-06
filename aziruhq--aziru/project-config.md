---
trigger: always_on
description: When working on Aziru, prioritize readability, safety, and a focused feature set. Avoid clever abstractions and do not add out-of-scope features without explicit approval.
---

# Project Context

When working on Aziru, prioritize readability, safety, and a focused feature set. Avoid clever abstractions and do not add out-of-scope features without explicit approval.

**Do not edit any code without the user's explicit consent.** Propose changes first and wait for approval before modifying files.

## About

Aziru is an open-source, self-hostable AI email triage assistant. It is Gmail-first, but not a full email client.

**Provider support (product decision):** Aziru implements Outlook support to mirror Gmail. Outlook must offer the same feature set as Gmail, with parity across triage, sorting, drafts (approval-only), and all downstream functionality. Mail-mutation policy: when the label-writeback feature flag is enabled, the write scope (gmail.modify / Mail.ReadWrite) is requested **upfront in bulk** at Google sign-in and inbox connect — writeback is **on by default** per workspace (users can switch it off), and upcoming in-provider features (thread summaries and draft replies surfaced inside the Gmail/Outlook UI) share that same grant, so authorization is gathered once rather than via per-feature incremental prompts. Aziru mirrors sorted folders into the mailbox as Gmail labels / Outlook categories under the `Aziru` namespace and keeps them in sync as threads are sorted. If the user declines the write permission (or the flag is off), the connection proceeds read-only and writeback is inert. Aziru still **never sends or deletes mail**, and applies no mailbox writes beyond that label/category writeback. New provider work should reuse the existing provider abstraction rather than forking Gmail-specific logic.

Aziru will also be offered as a hosted SaaS product. The codebase must support both deployment models equally. Design, architecture, storage layout, and API cost structure must never assume a single-tenant or fully self-managed environment. Specifically:

- Multi-tenancy must be a first-class concern: data isolation, per-user resource accounting, and tenant-scoped configuration should be built in, not retrofitted.
- AI and third-party API costs must be attributable per user so the hosted offering can track and control spend.
- Infrastructure choices (database, queues, storage) should have clear self-host-friendly defaults (e.g. Postgres, Redis, local file storage) while remaining swappable for managed cloud equivalents in production.
- Features that would be prohibitively expensive or operationally complex to run at scale in the hosted offering should be flagged before implementation.

Aziru sorts email threads, not individual messages. New messages in existing threads trigger re-sorting of the full thread.

A key use case is bulk triage of an existing inbox: users may want to sort and classify thousands of emails already accumulated, not just handle incoming ones. Features and jobs should be designed to handle both ongoing (real-time) and historical (backfill) triage at scale.

## Monorepo

- `apps/web/` - Next.js frontend
- `apps/api/` - TypeScript API server
- `apps/worker/` - background jobs
- `packages/db/` - Prisma schema, migrations, client
- `packages/shared/` - shared types and Zod schemas
- `packages/ai/` - AI providers, prompts, output validation
- `packages/config/` - shared env/config

## AI & Policy

- Treat LLM output as untrusted.
- Validate structured AI output with Zod.
- Reject unknown node IDs, invalid paths, and invalid final destinations.
- Policy code decides final actions, not prompts.
- Keep mock sorting available for deterministic testing.
- Support local Ollama for dev and frontier LLMs for production through provider abstraction.
- Tune and commit routing/AI constants for the production model configuration (frontier Gemini). Self-hosted deployments on other models set their own constants; do not tune shipped defaults against the offline CI model.
- **Thread summaries** are generated lazily on thread open, cached in `ThreadSummary`, and metered as `THREAD_SUMMARY`. Two invariants are non-obvious and load-bearing: single-message and automated threads return the stored snippet with no LLM call, no row, and no meter; and the summary text is derived email content, so it is never logged (not even a preview on a parse failure, unlike the draft module). A FAILED row never records a meter unit, which is what makes retry free.

## Safety & Privacy

- Never auto-send email.
- Never send from Aziru GUI.
- Drafts require user approval.
- Store minimal email data.
- Never log full email bodies.
- Encrypt OAuth tokens and API keys at rest.
- Audit important actions.

## Workflow

- At the end of large tasks (multi-file changes, feature additions, refactors), provide a brief summary: what was changed, which files were affected, and any caveats or follow-up work.

## UX

- Minimize the number of clicks required to complete any action. Prefer inline controls, smart defaults, and progressive disclosure over multi-step flows.
- Both the marketing site and the web app must be fully responsive. All layouts, components, and interactions must work correctly on mobile, tablet, and desktop screen sizes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aziruhq/aziru](https://github.com/aziruhq/aziru) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
