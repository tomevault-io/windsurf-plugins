---
trigger: always_on
description: You're working on opencompany: an AI workspace for chat, durable tasks and workflows,
---

# Agent coding guidelines

You're working on opencompany: an AI workspace for chat, durable tasks and workflows,
connected integrations, Wiki knowledge, and cloud coding sessions.

Read the nested `AGENTS.md` files when reading/editing files inside folders that contain it.

## North Star

We are building world-class software: high quality, high reliability, excellent UX, and code that a strong engineering team would be proud to review later.

Use judgment. The goal is not to follow rules mechanically; the goal is to ship correct, maintainable work with clear verification.

## Stack

- Package manager: `bun@1.4.2`
- Runtime: Node `>=20.20.0`
- Stack: Turborepo, Bun, Next.js App Router, Drizzle, Neon Postgres, WorkOS AuthKit, Vercel AI Gateway, GitHub App integration.
- Product app: `apps/web`
- Shared runner service: `apps/runner`
- Database package: `packages/db`

## Scripts

- Install dependencies: `bun install`
- Local setup: `bun run setup`
- Format check: `bun run format:check`
- Lint: `bun run lint`
- Typecheck: `bun run typecheck`
- Build: `bun run build`
- Unit tests: `bun run test`
- UI behavior: run `bun run dev:web` and verify the real route in a browser
- Secret scan when available: `bun run secrets:check`
- Use Turborebo filtering syntax to run commands against specific apps/packages: `bun run dev --filter @opencompany/web`

The user usually keeps a dev server running. Do not start another one unless asked or unless you have confirmed it is needed.

## opencompany product surface

- Start in `apps/web` for product and API work.
- `web` is the Next.js client and composition root. "opencompany runner" means the retained
  opencompany-domain execution paths inside `apps/runner`. Look first at the task and chat
  modules, `/internal/goat/*` routes, and the `RUNNER_OPENCOMPANY_TASK_WORKER_ENABLED` gate. There is no
  separate runner package.
- Follow shared code into `packages/db/src/*`, `packages/wiki`, and
  `packages/telemetry` as needed. Preserve the isolated legacy-billing and LLM-broker
  compatibility schemas unless a task explicitly retires those contracts.
- Use `docs/system-map.md` for the current app/runner flow and `bun run dev:web` for the
  local product stack.

## Engineering Judgment

- Correctness beats speed. Code that types and tests pass is not automatically correct.
- Prefer existing local patterns, helpers, components, and conventions over new abstractions.
- Fix adjacent issues when they materially affect the task, correctness, or maintainability of touched code. Do not bundle unrelated cleanup into the same PR.
- Push back when a request would make the system worse. State the tradeoff and propose the better path.
- Keep names precise. A good name should remove the need for a comment.
- Delete dead code. Do not leave commented-out code or TODOs without a real tracking reason.
- Write descriptive comments where some patterns are not clear.
- DO NOT write unneccessary comments for obvious things, but make sure to comment complex logic or workarounds.
- Validate user input and external API responses at boundaries. Internal code should be typed enough to avoid defensive clutter.

## Reliability And Safety

- Do not swallow errors silently. Surface them, handle them, or make the invalid state impossible.
- Do not add retries, fallbacks, feature flags, or abstractions for hypothetical future problems.
- Treat external input as hostile.
- Never commit secrets, log secrets, or echo secret values in summaries.
- Avoid new dependencies unless the value clearly outweighs maintenance and security cost.

## Data And Env Changes

- Any change to `packages/db/src/schema.ts` needs a Drizzle migration.
- Migration or data-destructive work gets extra scrutiny. Explain rollback implications before running one-way operations.
- New env vars require `.env.example` and the relevant docs update.
- Production env vars must be added to the runtime-specific Infisical path and verified in the hosted service before release: the web app uses `prod` + `/web`, the runner uses `prod` + `/runner`, and release automation uses `prod` + `/release`. Add required variables to the matching release preflight so a missing sync fails the release instead of silently disabling behavior.
- Local setup should use branch-isolated Neon DBs through `bun run setup`. Avoid shared database mode unless explicitly needed.
- Do not run production migrations or production-affecting scripts unless the user explicitly asks.

## Testing Expectations

Use the repo’s existing tooling. Do not introduce a new test runner or fixture style unless the existing setup cannot cover the behavior.

- Pure logic or utilities: add or update focused unit tests.
- API routes, server actions, and data flows: exercise the real path when practical.
- UI changes: verify in the browser against the running dev server, covering the main path and at
  least one obvious edge case. Include current screenshots or a short screen recording in the PR
  for every UI or UX change, captured from the real product and showing the relevant states.
- Refactors with intended no behavior change: run the existing relevant checks. Add a small characterization test if the touched behavior has no useful coverage.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [useopencompany/opencompany](https://github.com/useopencompany/opencompany) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
