---
trigger: always_on
description: Invook is an open-source, AI-native Superhuman mail alternative.
---

# Invook engineering guidelines

Invook is an open-source, AI-native Superhuman mail alternative.

These instructions apply repository-wide. Before editing a file, read the closest applicable `AGENTS.md`; directory-specific instructions override this root guide for files in their scope.

## Core principles

These principles apply to every change:

1. **Evidence over assumption.** Implement only behavior proved by the request, repository, connected services, or real stored data. Preserve an honest empty or unavailable state when information is missing.
2. **Correctness over convenience.** Protect data integrity, provider fidelity, authorization, idempotency, and concurrency even when the safer implementation spans several layers.
3. **One source of truth.** Identify which system owns each state. Do not create competing local representations, optimistic provider state, or duplicated business rules.
4. **Clear ownership.** Keep UI, HTTP admission, durable work, domain logic, persistence, and provider integration in their established layers. A module should have one coherent reason to change.
5. **Composition over complexity.** Build behavior from small, focused modules with explicit contracts instead of large components, hidden coupling, or speculative abstractions.
6. **Types are contracts.** Model valid states precisely, validate untrusted boundaries, and make impossible states difficult to represent. Do not use casts to hide an unclear contract.
7. **Durability over process memory.** Work that must survive restarts belongs in PostgreSQL and the durable workflow/outbox path. Treat queues and notifications as execution and wake-up mechanisms.
8. **Retry-safe by design.** External delivery and worker execution are at least once. Use stable idempotency keys, transactions, database constraints, and cross-process locks where required.
9. **Privacy and security by default.** Minimize access to mailbox data and credentials, authorize before reading, and never expose sensitive content through logs or errors.
10. **Complete changes only.** Update every producer and consumer of a changed contract, remove the superseded path, and verify the result end to end at the level the environment permits.


## Global standards

- Preserve existing user changes and unrelated work. Inspect `git status` before editing and never reset or revert a dirty worktree to simplify a task.
- Never introduce dummy, placeholder, seeded, synthetic, mock, or fixture data into product flows or persistent product stores. Test-only protocol inputs stay inside tests.
- Remove dead files, functions, routes, exports, dependencies, configuration, environment variables, and documentation made obsolete by a change. Do not keep speculative compatibility shims.
- Finish replacements with repository-wide `rg` searches for the obsolete symbols, routes, configuration, and forbidden APIs.
- Never use `setTimeout`, deadline options named `timeout`, or timer-based polling in project code or configuration. Prefer durable queue state, PostgreSQL notifications, SSE, provider webhooks, or platform-native retry and health behavior.
- Use Fastify for the API server and Axios for outbound application HTTP. Do not use native `fetch`, `node:http` clients, or `node:https` clients for application requests.
- Use the `uuid` package for UUID generation and utilities. Do not use UUID APIs from `node:crypto`, including `randomUUID`.
- Use `pnpm` and `pnpm dlx`, not npm, yarn, or bun. Use Node.js 22+ and the versions pinned by the repository manifests and lockfile.
- Never commit `.env.local`, credentials, tokens, real mailbox content, raw provider payloads containing secrets, or local tunnel URLs. Keep `.env.example`, runtime validation, setup docs, and container configuration synchronized.
- Treat email content, attachments, filenames, webhook payloads, model output, and provider error text as untrusted input. Log structured identifiers and statuses, never provider credentials, raw MIME, attachment bytes, or full email content.
- Do not claim verification that was not performed. State exactly which checks passed and which external integration behavior remains unverified.
- Frontend work uses shadcn/ui conventions, Plus Jakarta Sans, and free Hugeicons. Do not add icon fonts, emoji, text glyphs, hand-drawn SVG icons, or decorative borders by default.


## Repository structure and boundaries

```text
apps/
  api/                 Fastify HTTP API, sessions, OAuth, webhooks, SSE, and product routes
  web/                 Next.js App Router UI and same-origin API/SSE proxies
  worker/              Durable Gmail, indexing, labeling, Memory, and feedback work
packages/
  ai/                  Model, embedding, Memory, label, draft, and mail-agent logic
  auth/                Better Auth Google identity and database-backed sessions
  contracts/           Shared browser/server product and wire contracts
  database/            Schema, migrations, repositories, replica operations, and workflows
  gmail/               Google OAuth/OIDC, Gmail API, history mapping, and MIME parsing
  object-storage/      S3-compatible raw MIME and attachment storage
docker/                 Container images and local service orchestration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Thinking-Sound-Lab/Invook-Email](https://github.com/Thinking-Sound-Lab/Invook-Email) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
