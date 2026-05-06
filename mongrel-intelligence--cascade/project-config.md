---
trigger: always_on
description: cd web && npm install && cd ..
---

# CASCADE — PM-to-Code Automation Platform

## Quick start

```bash
npm install
cd web && npm install && cd ..
# Redis required (router/BullMQ). `.cascade/setup.sh` installs + starts it.
npm run dev          # Router (webhook receiver, :3000)
npm run dev:web      # Dashboard frontend (:5173, separate terminal)
node dist/dashboard.js   # Dashboard API (:3001, third terminal, after `npm run build`)
```

> `npm start` runs the **router** (`dist/router/index.js`), **not** the dashboard.

## Architecture

Three separate services, **no monolithic server mode**:

1. **Router** (`src/router/index.ts`) — receives webhooks, enqueues to Redis/BullMQ.
2. **Worker** (`src/worker-entry.ts`) — processes one job per container, exits.
3. **Dashboard** (`src/dashboard.ts`) — tRPC API + static frontend for web UI and CLI.

Flow: `PM/SCM/alerting webhook → Router → Redis → Worker → TriggerRegistry → Agent → Code → PR`.

**Capacity-gate invariant.** Every PM router adapter (`src/router/adapters/{linear,trello,jira}.ts`) must wrap `triggerRegistry.dispatch(ctx)` in PM-provider `AsyncLocalStorage` scope via the shared `withPMScopeForDispatch(fullProject, dispatch)` helper at `src/router/adapters/_shared.ts` — in addition to the per-PM-type credential scope (`withLinearCredentials` / `withTrelloCredentials` / `withJiraCredentials`). Without the PM-provider wrapping, the pipeline-capacity gate at `src/triggers/shared/pipeline-capacity-gate.ts` cannot resolve `getPMProvider()`, **fails closed** under the spec-017 fail-closed policy (blocks the run + ERROR + Sentry capture under tag `pipeline_capacity_gate_no_pm_provider`), and `maxInFlightItems` is silently disabled for the PM-source path. Mirror the GitHub adapter's existing correct shape at `src/router/adapters/github.ts:dispatchWithCredentials`. The static guard at `tests/unit/integrations/pm-router-adapter-pm-scope.test.ts` enforces this at CI time — adding a new PM router adapter without the wrapping fails CI with a precise file path.

Integration abstraction lives in `src/integrations/`. For **adding a new PM provider**, see @src/integrations/README.md — PM providers (Trello, JIRA, Linear) use the `PMProviderManifest` registry with a **behavioral conformance harness** (spec 009 — config round-trip, discovery shape, full lifecycle scenario, auth-header provenance, single-entrypoint invariant). Each provider owns its Zod config schema (`src/integrations/pm/<provider>/config-schema.ts`) as the single source of truth — the central `src/config/schema.ts` imports it. PM adapter method signatures use branded `StateId` / `LabelId` / `ContainerId` from `src/pm/ids.ts` to make state-name-vs-ID confusion a compile error at direct-adapter call sites. All runtime surfaces (router, worker, CLI, dashboard) register integrations through a single entrypoint at `src/integrations/entrypoint.ts`. **Spec 010 follow-ups** added generic `pm.discovery.createLabel` / `createCustomField` mutation endpoints + `currentUser` discovery capability + real shared React components for every `StandardStepKind` under `web/src/components/projects/pm-providers/steps/`. **Spec 011** migrated all three production providers (Trello, JIRA, Linear) onto those shared components, added a 7th `StandardStepKind: custom-field-mapping`, widened `container-pick` / `project-scope` / `webhook-url-display` with optional props, and deleted the three legacy `pm-wizard-{trello,jira,linear}-steps.tsx` files. **Spec 012** migrated each provider's webhook UX (programmatic create for Trello/JIRA, signing-secret + instructions for Linear) into per-provider manifest webhook adapters (Fragment compositions around the shared `WebhookUrlDisplayStep`); deleted the legacy `WebhookStep` + `LinearWebhookInfoPanel` + `useWebhookManagement` + `useLinearWebhookInfo`. Every PM wizard step now renders via the manifest path without exception. A new PM provider writes zero edits to shared orchestration (`pm-wizard.tsx`, `pm-wizard-common-steps.tsx`, `pm-wizard-hooks.ts`); provider-specific UI ships either as `kind: 'custom'` steps or as Fragment compositions inside the provider folder's wizard adapters. SCM (GitHub) and alerting (Sentry) still use the legacy `IntegrationModule` pattern via self-registration in `src/github/register.ts` + `src/sentry/register.ts`. Don't improvise; the README covers both patterns.

## PR checkout (worker) — gotcha

Worker checks out PRs via `refs/pull/N/head` (works for same-repo **and** external-fork branches). When `prNumber` is set on `AgentInput`, `setupRepository`:

1. Fetches `+refs/pull/<N>/head:refs/remotes/pr/<N>` from `origin`.
2. Detached-checks out `pr/<N>`.
3. If `headSha` is also set, verifies `git rev-parse HEAD` matches.

Any non-zero git exit code **throws** — no warn-and-continue. The legacy `prBranch` field is retained for log readability but **not** used to drive checkout (fork branches don't exist on `origin` and the by-name path silently 404s).

## Testing

```bash
npm test                 # Unit tests (all 4 unit projects)
npm run test:integration # Integration tests (requires Postgres — see below)
npm run test:all         # Unit + integration
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mongrel-intelligence/cascade](https://github.com/mongrel-intelligence/cascade) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
