---
trigger: always_on
description: - Answer in the user's language
---

# AGENTS.md

## Operating Standard

- Answer in the user's language
- Read the relevant chat history before acting.
- Be autonomous by default: inspect, decide, implement, validate, and report without unnecessary confirmation loops.
- Ask only when ambiguity blocks a safe decision, the product choice is genuinely open, or the action is risky/destructive enough that the user should explicitly choose.
- Do not hallucinate. Verify uncertain claims through code, scripts, docs, tests, runtime output, or repository evidence.
- Preserve unrelated user changes. Do not revert, overwrite, reformat, or clean up work you did not create unless explicitly asked.
- Prefer evidence over ceremony. Keep process proportional to the task.
- The job is not to sound smart. The job is to leave the system clearer, more correct, and easier to trust.

## Instruction Priority

- If instructions conflict, follow higher-priority system, developer, and user instructions first, then the nearest repository instructions.
- Safety, privacy, and preservation of user work take priority over speed or convenience.
- When editing this file, keep equivalent agent files such as `CLAUDE.md` aligned unless the difference is intentional and documented.

## User Interaction

- Assume repository users are Vibe coders without programming experience. Do not ask them to evaluate technical implementations or choose which of two good engineering solutions is better.
- When user input is genuinely needed, frame the question as a product decision: describe the user experience, behavior, constraint, or tradeoff each option creates, then recommend one.
- Keep technical rationale concise and tied to product impact. Ask for confirmation only when the product outcome is truly open or the action is risky.

## Repository Grounding

- Start from the repository itself, not assumptions.
- For non-trivial work, read `README.md` and relevant `docs/` early for setup, architecture, runbooks, product constraints, and caveats.
- Trust current code, scripts, schemas, tests, and runtime output over stale docs. Call out doc drift and align it when practical.
- When relevant files or repository shape are unclear, get a fresh snapshot with `tree -L 2`, `tree -L 3`, or `rg --files`.
- Do not treat `README.md` as a file inventory. Discover structure dynamically.
- Use the repository's existing package manager, scripts, test runner, formatter, linter, build tools, and generators.
- Use `docs/LOCAL_DATABASE.md` and `docker-compose.yml` as the local PostgreSQL source of truth. Default to Docker Compose across Windows, macOS, and Linux; do not ask for native PostgreSQL setup unless the user explicitly chooses it.
- In Codex shell sessions, do not assume JS tooling is already on `PATH`. For `node`, `npm`, and `bun`, prefer `PATH="/opt/homebrew/bin:$HOME/.bun/bin:$PATH"`.
- Do not add new production dependencies without explicit user approval unless the user directly requested that dependency by name. Prefer existing utilities, framework APIs, and the standard library.
- Before implementing with a new library, inspect the relevant `package.json` first. Prefer established libraries already installed in this template, especially Zod, TanStack Query, TanStack Form, Hono, Prisma, Expo, and the shared `@web-app-demo/contracts` package.
- If a missing dependency would clearly improve the product outcome, explain the user-visible reason and ask before installing it.
- Before using framework-specific APIs, check the current official documentation or local installed package types/examples, then write code to match the current API rather than memory.
- For E2E, use Playwright for web and Maestro for mobile. Read `docs/TESTING.md` before adding flows. Prefer valuable user-visible coverage over narrow happy-path-only smoke tests: cover critical journeys, high-risk regressions, auth/session behavior, persistence, navigation, and important empty/error/edge states when the test can stay stable and maintainable. Keep exhaustive validation matrices, combinatorial edge cases, concurrency, and pure business rules in integration/contract/unit tests.
- For mobile E2E selectors, prefer stable React Native `testID` constants from `mobile/src/constants/testIds.ts`; do not rely on coordinates or fragile text when an action selector can have an id.
- For Expo dev client + Maestro, run against an installed development build, not Expo Go. Open the app bundle with the dev-client `openLink` URL from `MAESTRO_DEV_SERVER_URL`, and preflight backend/Metro reachability before UI steps.
- For mobile E2E input stability, use `EXPO_PUBLIC_E2E=1` only in E2E bundles, keep production password fields secure, avoid `hideKeyboard`, center important CTA targets before taps, and keep custom touch targets around `44-48pt` or larger.
- After changing mobile Maestro flows, runner inputs, or E2E-only app behavior, run `bun run --cwd mobile e2e:maestro:audit` alongside the relevant typecheck/test/lint validation.

## Project Context

- Use `README.md` as the source of truth for first-run repository download, bootstrap, and product intake instructions.
- Keep durable project choices in README files and docs, not in this agent file.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [di-sukharev/vibe](https://github.com/di-sukharev/vibe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
