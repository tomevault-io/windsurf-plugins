---
trigger: always_on
description: > **READ THIS FIRST.** Whenever a code change in this repo would make `SETUP.md` or `FEATURES.md` even slightly inaccurate, you **must** update those files in the same response — without being asked, without confirmation, without waiting for a follow-up. Treat the doc update as part of the task itself, not as a separate optional step. A task is **not complete** until the docs match reality.
---

# Claude Code — Project Instructions

## Mandatory doc maintenance — AUTOMATIC, NO PROMPTING NEEDED

> **READ THIS FIRST.** Whenever a code change in this repo would make `SETUP.md` or `FEATURES.md` even slightly inaccurate, you **must** update those files in the same response — without being asked, without confirmation, without waiting for a follow-up. Treat the doc update as part of the task itself, not as a separate optional step. A task is **not complete** until the docs match reality.
>
> The user should never have to remind you to "also update the docs" or "make sure FEATURES.md is current". If they have to say it, you've already failed the rule.

### SETUP.md
Update whenever:
- A new environment variable is added, renamed, or removed.
- A new provider/integration changes how it authenticates (OAuth vs API key).
- A prerequisite changes (Node version, system dependency, etc.).
- The startup command or DB path changes.
- A "how to connect" step changes.

### FEATURES.md
Update whenever:
- A new feature, module, KPI, card, or sidebar section is added — add it to the relevant section.
- A feature, control, button, toggle, or setting is removed or renamed (e.g. "Toggle density" being deleted means the row in the shortcut table goes away in the same commit).
- A keyboard shortcut is added, changed, or removed.
- An integration is added or dropped from a module.
- Header / Settings / Preferences gain or lose a control, tab, or pref-card.
- Something moves from "Not yet implemented" to done — remove it from that list.
- Something is deferred that was previously working — add it to "Not yet implemented".

### Self-check before ending the turn
Before you say "done", scan: did this change touch any user-facing control, env var, route, shortcut, integration, setting, or section? If yes, open SETUP.md and FEATURES.md and confirm both still describe the project accurately. If not, fix them now — in the same response.

## MANDATORY PRE-PUSH CHECKLIST — no exceptions

Before any `git push` or PR, ALL of the following must pass locally:

1. **`npx tsc --noEmit`** — zero type errors.
2. **`npm run test:coverage`** — NOT `npm test`. This is the command CI runs and it enforces coverage thresholds. `npm test` alone is insufficient.
3. **New code = new tests.** Every new function, module, or non-trivial branch added in a PR must have a corresponding unit test. "It's frontend DOM code" is not an exemption — if the code cannot be reached by the existing Node test environment, either (a) write a test using jsdom/happy-dom, or (b) get explicit sign-off from the user before accepting a threshold adjustment. Silently lowering thresholds without explaining the reason and getting approval is not acceptable.
4. **Patch coverage must not be 0% on new code.** Before pushing, confirm that every new function and every new branch you added is exercised by a test. If `npm run test:coverage` shows new lines as uncovered, write the tests first — do not push. A `codecov/patch` check flagging 0% on your diff is not a config issue to paper over with `informational: true`; it means tests are missing. This rule applies to every PR without exception.
5. **Coverage thresholds must not decrease** unless the user explicitly approves. If new code genuinely cannot be unit-tested (pure browser APIs with no jsdom path), document why in the commit message and ask the user before touching `vitest.config.ts`.
6. **Sync with `prod` before every push.** Run `git fetch origin prod && git rebase origin/prod` before pushing any branch. Resolve all conflicts locally, re-run steps 1–4, then push. A PR with a merge conflict is not ready for review.

A push that causes CI to fail or has unresolved conflicts is a broken workflow. The checklist above exists so that never happens.

---

## Project overview

A personal productivity dashboard running locally on Mac. Node + TypeScript backend, Vite + TypeScript frontend, SQLite for all persistence. No cloud, no external hosting. The project name shown in the header / browser tab is configurable via Settings → Preferences (`brand.name` / `brand.subtitle` in the `settings` table). Do not hardcode "Daily Command Center" or any owner name in new user-facing strings — route them through `applyBrand` (frontend) or `getAppSetting("brand.name", …)` (server).

- Server entry: `src/server/index.ts`
- Frontend entry: `src/frontend/main.ts`
- Styles: `src/frontend/styles.css`
- DB: `src/server/db.ts` — all migrations inline in `migrate()`, seed from `.env` in `seedFromEnv()`
- Config: `src/server/config.ts` — reads `.env` via dotenv

## Key architecture decisions

- **Multi-workspace model**: workspaces → connector_instances → identities in SQLite. `effective()` in each integration reads DB first, falls back to `.env` config.
- **Request context**: active workspace propagated via AsyncLocalStorage (`src/server/lib/request-context.ts`), set from `?workspace=` query param.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ewaily/DailyCommandCenter](https://github.com/Ewaily/DailyCommandCenter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
