---
trigger: always_on
description: This file defines how coding agents should work in the official `reorder` repository.
---

# Reorder agent guidelines

This file defines how coding agents should work in the official `reorder` repository.

## Always

- Write all code, comments, specs, markdown files, lessons, and commit messages in English only, regardless of the language used in the chat.
- **When the user approves changes after code review, propose a commit message in the Conventional Commits format `type(scope): description` (e.g., `feat(ai): add create-spec skill`, `fix(dunning): resolve retry loop`). Wait for explicit user approval of the commit message before committing and pushing changes to the repository.**
- Identify which Reorder area you are changing and check the Task Router below before starting.
- Read the relevant runtime documentation in `docs/` before reading implementation files.
- Refer to `docs/README.md` for plugin overview, current scope, and implemented domains.
- Enter plan mode for non-trivial tasks (3+ steps or architectural decisions) and use the `create-spec` skill to draft a specification in `.agents/specs/` before writing code.
- Check `.agents/lessons.md` at the start of the session to avoid repeating past mistakes.
- After fixing any bug or resolving a complex issue, update `.agents/lessons.md` with the lesson learned and a rule to prevent it in the future.
- Use Medusa agentic skills whenever they fit the task.
- After successfully pushing code to GitHub, **always** ask the user if the public Mintlify documentation needs to be updated. If yes, execute the `sync-docs` skill.
- Keep changes minimal and local to the affected area.
- Follow Medusa conventions (file-based routing with `route.ts`, Awilix resolve, custom modules under `src/modules/<domain>/`, models in `models/`, migrations in `migrations/`, workflows in `src/workflows/`, jobs in `src/jobs/`).
- Write integration tests for new features (preferring HTTP integration tests in `integration-tests/http/`). Keep them self-contained.
- If behavior changes, update the matching runtime documentation in `docs/`.

## Ask First

- Ask and obtain explicit confirmation before executing destructive data wipe scripts (e.g., `wipe-test-data`).
- Ask before changing branch/PR automation, pipeline labels, QA flows, or release behaviors.
- Ask before making changes that span multiple domains or modules without an existing spec.
- Ask before adding new external dependencies to `package.json`.
- Ask before modifying database models or introducing complex schema migrations.

## Never

- Never use `any` in TypeScript code. Prefer descriptive, strict domain types.
- Never write business rules directly in route handlers or React components; keep them in workflows or service layers.
- Never bypass mutation guards or domain boundaries (do not introduce unnecessary cross-domain coupling).
- Never modify generated files by hand.
- Never refactor unrelated files while fixing a local issue.
- Never document intended future behavior; only document stable, implemented behavior.

## Validation Commands

Run the smallest relevant validation command for your changes:

```bash
yarn build
yarn test:integration:http
yarn test:integration:modules
yarn test:e2e                                      # Requires running Medusa backend (ADMIN_BASE_URL)
```

## Task router

Match the task to all relevant rows before researching or coding.

| Task | Read first / Action |
|------|------------|
| Plugin overview, current scope, implemented domains | `docs/README.md` |
| Subscription domain changes | `docs/architecture/subscriptions.md`, `docs/api/admin-subscriptions.md`, `docs/testing/subscriptions.md` |
| Plan and offer changes | `docs/architecture/plan-offers.md`, `docs/api/admin-plan-offers.md`, `docs/testing/plan-offers.md` |
| Renewal changes | `docs/architecture/renewals.md`, `docs/api/admin-renewals.md`, `docs/testing/renewals.md` |
| Dunning changes | `docs/architecture/dunning.md`, `docs/api/admin-dunning.md`, `docs/testing/dunning.md` |
| Payment context, payment methods, and off-session charging | `docs/architecture/payments.md`, `docs/api/store-subscription-payment-methods.md`, `docs/api/admin-subscriptions.md` |
| Cancellation and retention changes | `docs/architecture/cancellation.md`, `docs/api/admin-cancellations.md`, `docs/testing/cancellations.md` |
| Activity log changes | `docs/architecture/activity-log.md`, `docs/api/admin-activity-log.md`, `docs/testing/activity-log.md` |
| Analytics changes | `docs/architecture/analytics.md`, `docs/api/admin-analytics.md`, `docs/testing/analytics.md` |
| Subscription settings changes | `docs/architecture/settings.md`, `docs/api/admin-subscription-settings.md`, `docs/testing/subscription-settings.md` |
| Storefront and customer account subscription APIs | `docs/api/store-subscription-checkout.md`, `docs/api/store-subscription-offers.md`, `docs/api/store-customer-cancellations.md`, `docs/architecture/subscriptions.md` |
| Admin UI routes and widgets | matching files in `docs/admin/`, then `src/admin/README.md` |
| Admin UI E2E browser tests | `playwright.config.ts`, `e2e/`, matching `docs/testing/*.md` |
| Admin or store API route implementation | `src/api/README.md`, then matching `docs/api/*.md` |
| Workflow-backed mutations | `src/workflows/README.md`, then matching architecture and API docs |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reorder-js/reorder](https://github.com/reorder-js/reorder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
