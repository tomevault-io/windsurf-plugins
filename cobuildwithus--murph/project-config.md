---
trigger: always_on
description: Prefer deletion and the smallest maintainable architecture that satisfies the
---

# AGENTS.md

## Default To Deletion And Simplicity

Prefer deletion and the smallest maintainable architecture that satisfies the
current requirement. Add a dependency, abstraction, service, state owner, or
process only when concrete product, security, test, or measured performance
evidence proves the simpler design insufficient.

## Purpose

This file is the compact routing map for agent work in this repository.
Durable guidance lives in `agent-docs/`; keep detailed policy there instead of expanding this file.

## Precedence

1. Explicit user instruction in the current chat turn.
2. `Hard Rules (Non-Negotiable)` in this file.
3. `agent-docs/operations/agent-workflow-routing.md`.
4. Other detailed docs under `agent-docs/**`.

If instructions still conflict after applying this order, ask the user before acting.

## Read First

Always read these before repo code/docs/test/config work:

1. `agent-docs/index.md`
2. `ARCHITECTURE.md`
3. `docs/contracts/00-invariants.md`
4. `agent-docs/ARCHITECTURE_GUIDANCE.md`
5. `agent-docs/references/repo-scope.md`
6. `agent-docs/operations/agent-workflow-routing.md`
7. `agent-docs/PRODUCT_SENSE.md`
8. `agent-docs/PRODUCT_CONSTITUTION.md`

## Task Router

| If the task is about... | Also read | Notes |
| --- | --- | --- |
| Review-only inspection with no planned file edits | `agent-docs/operations/verification-and-runtime.md` | No repo-wide checks by default. Add runtime proof only when requested or when static inspection leaves a material gap. |
| Docs or process only | `agent-docs/operations/verification-and-runtime.md` | Follow the docs/process task class in the workflow router. |
| Repo code, tests, or config | `agent-docs/operations/completion-workflow.md`, `agent-docs/operations/verification-and-runtime.md` | Use the workflow router for task class, plan needs, audits, verification, and commit path. |
| User-facing frontend/UI work in `apps/web` | `agent-docs/FRONTEND.md` | Follow the normal task-class implementation route; the completion workflow still controls browser proof and required frontend review. |
| Auth, secrets, trust boundaries, or external runtime surfaces | `agent-docs/SECURITY.md` | Treat as higher risk by default. |
| Retries, queues, cron, concurrency, or failure handling | `agent-docs/RELIABILITY.md` | Capture direct proof for operational changes. |
| Cloudflare infrastructure, Workers, Durable Objects, R2, or deploy/runtime platform APIs | `agent-docs/SECURITY.md`, `agent-docs/RELIABILITY.md`, relevant official Cloudflare docs | Read Cloudflare docs thoroughly before designing; prefer the simplest canonical Cloudflare API or feature, and assume the platform likely already provides the needed primitive before rolling bespoke infrastructure. |
| Test selection or verification changes | `agent-docs/references/testing-ci-map.md` | Keep test coverage and doc claims aligned. |
| Product behavior or UX tradeoffs | `agent-docs/PRODUCT_SENSE.md`, `agent-docs/PRODUCT_CONSTITUTION.md` | Prefer repo-local durable specs over chat memory. |
| iMessage/SMS replies, outbound message copy, reminders, notifications, line health, or phone-number messaging behavior | `agent-docs/operations/imessage-deliverability.md` | Design for reciprocal conversations, safe pacing, link hygiene, cold-contact protection, and fail-closed line health. |
| Marketing, positioning, copy, or experiment library work | `agent-docs/product-marketing-context.md` | Use the repo marketing context for positioning, differentiation, customer language, and brand voice. |
| Health Commons content or experiment library structure | `agent-docs/product-specs/health-commons.md` | Generated catalog artifacts are ignored build outputs; commit authored content and intentional generator/schema/test changes only. |
| Dependency changes | `agent-docs/SECURITY.md` | Follow the dependency supply-chain rules before handoff. |

## Hard Rules (Non-Negotiable)

- Never expose secrets, raw credentials, private keys, tokens, full `Authorization` headers, or downloaded secret values in commits, code, docs, generated files, comments, logs, examples, quoted output, or external artifacts. Keep legal names, local account usernames, and home-directory paths out of committed or published artifacts; for local debugging, prefer repo-relative paths and do not let identifier redaction block root-cause proof.
- Treat screenshots, chat transcripts, and user feedback as confidential evidence, not repository-ready source material. Never copy, closely paraphrase, or hardcode them, including names, handles, images, identifying details, distinctive wording, or exact scenarios, into system prompts, tests, fixtures, snapshots, evals, docs, comments, PR descriptions, or any source that may become public.
- Treat `.env` and `.env*` as sensitive. Never print, commit, or otherwise expose their contents.
- Do not pull remote environment variables into local files for inspection. Use provider CLI list/status commands that show names/scopes only, and ask before any operation that would download secret values.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cobuildwithus/murph](https://github.com/cobuildwithus/murph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
