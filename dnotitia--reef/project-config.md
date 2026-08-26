---
trigger: always_on
description: > Root-level, cross-cutting rules for reef. Package-local rules live in
---

# Project Context for AI Agents

> Root-level, cross-cutting rules for reef. Package-local rules live in
> `packages/core/AGENTS.md`, `packages/web/AGENTS.md`,
> `packages/jira-migrator/AGENTS.md`, `packages/orchestration/runtime/AGENTS.md`,
> `packages/orchestration/controller/AGENTS.md`,
> `packages/orchestration/providers/reef/AGENTS.md`,
> `packages/orchestration/providers/codex/AGENTS.md`, and
> `packages/orchestration/providers/github/AGENTS.md`,
> nested `AGENTS.md` files under those package trees; the `CLAUDE.md` files only
> point back to these `AGENTS.md` files.

## Rule Placement

- Keep this root file for repo-wide contracts that cross package boundaries:
  security, persistence, issue data model, schema ownership, release gates, and
  workflows that must stay consistent across packages.
- Put package defaults in the matching workspace package's `AGENTS.md`.
- Put implementation rules in the nearest subtree `AGENTS.md` so agents editing
  that code see the rule without carrying unrelated context. Examples:
  `packages/core/src/adapters/AGENTS.md`, `packages/web/src/app/AGENTS.md`,
  `packages/web/src/server/AGENTS.md`, and `packages/web/tests/e2e/AGENTS.md`.
- When a rule outgrows a short contract or becomes a runbook, move the runbook
  to `docs/` and leave a one-line pointer here or in the nearest package file.

## Repo Shape

- Exact dependency and runtime versions live in `package.json`, package manifests,
  and `tsconfig*.json`; do not rely on version guesses from memory.
- This is a pnpm workspace with private packages. Product runtime behavior
  starts in `core` for schemas, models, AKB access, and shared contracts, then
  the server-only application and provider adapters in `web` own GitHub, LLM,
  and agent execution before the result surfaces through the UI. Operator-run
  migration behavior for
  Jira lives in `packages/jira-migrator`; the provider-neutral one-run
  execution core and process signal seam live in `packages/orchestration/runtime`, while
  callers own scheduling and delivery orchestration. Concrete orchestration
  providers, including the private GitHub SCM adapter, own only the backend I/O
  granted by their provider contract.
- `core` is framework-agnostic: no Next.js imports, no DOM APIs, and no Node-only
  globals where avoidable.

## Issue Tracking

reef's own development is tracked in an akb vault (`project_prefix=REEF`) that is
internal to Dnotitia; access to it is not a prerequisite for contributing. When
you do edit issues directly in akb, read the target vault's vault-skill first.
Issue lifecycle state is the `reef_issues.status` row value, not document
metadata.

## Core Invariants

- reef-web persists nothing that belongs to a specific user: no database,
  server-side session store, Redis, KMS, or per-user cache.
- The akb session is the `__reef_session` httpOnly cookie; decode it read-only
  per request and forward the AKB-issued JWT to akb as
  `Authorization: Bearer <akb-jwt>`.
- GitHub access for monitored-repo grounding is deployment
  managed through `REEF_GITHUB_APP_ID`, `REEF_GITHUB_APP_INSTALLATION_ID`, and
  `REEF_GITHUB_APP_PRIVATE_KEY`, with `REEF_GITHUB_PAT` allowed only as a
  deployment-managed dev/CI fallback; reef-web must not collect, store, or
  forward a browser GitHub PAT.
- LLM configuration is deployment-managed server state through the single
  provider-neutral `REEF_LLM_API_KEY`, `REEF_LLM_BASE_URL`, and
  `REEF_LLM_MODEL` contract. Set all three to enable AI or none to disable it;
  partial configuration fails closed. The URL may target OpenRouter or an
  akb-platform gateway, but Reef does not infer a provider or deployment mode.
  Never store per-user LLM keys.
- AKB is the user-account authority. Preserve the stable account-denial codes
  `membership_required`, `account_suspended`, and `identity_conflict`; an AKB
  account denial or invalid-session 401 must clear every established Reef auth
  cookie before returning. A resource-level permission denial must not sign the
  user out.
- `core` is the only place where Reef product AKB I/O and AKB auth/session calls
  (`login`, `getMe`, `getCurrentActor`) originate. The server-only `web`
  application owns monitored-repository GitHub I/O, deployment-managed LLM I/O,
  and agent execution; it consumes core's public schemas, errors, models, and
  AKB adapter. Separate orchestration provider packages own only provider I/O
  explicitly granted by their provider-neutral contracts, such as SCM Git
  transport and GitHub pull-request delivery. Route Handlers remain thin: they
  own session/cookie lifecycle (set/clear the `__reef_session` cookie, decode
  it, translate `ReefError` to PM-facing language), never an inline `fetch` or
  an inline AKB wire schema.

## TypeScript And Boundaries

- TypeScript strict mode is mandatory. Avoid `any`; justify any `@ts-ignore`.
- Zod schemas in `packages/core/src/schemas/` are the single source of truth for data that
  crosses boundaries. Import inferred types instead of redefining them in `web`.
- Wire fields from akb documents, `reef_issues` rows, and GitHub payloads stay
  `snake_case`; TypeScript variables, function names, and React props stay
  camelCase.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dnotitia/reef](https://github.com/dnotitia/reef) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
