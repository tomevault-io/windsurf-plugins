---
trigger: always_on
description: Open-source Discord-like voice + text chat (pqp.gg). See [`CLAUDE.md`](./CLAUDE.md) for the stack and how to run.
---

# Agent notes

Open-source Discord-like voice + text chat (pqp.gg). See [`CLAUDE.md`](./CLAUDE.md) for the stack and how to run.

## Agent delegation

Act as the lead engineering agent. Prefer Astra for orchestration, architecture decisions, ambiguous implementation decisions, cross-cutting changes, the final implementation plan, integration, and final review.

Delegate aggressively when work can proceed independently: repository exploration, finding files, tracing call paths, investigating patterns, straightforward tests, mechanical refactors, documentation lookup, and reviewing a proposed implementation.

- Give each agent a narrow objective, explicit file ownership where it edits, and a clear completion criterion.
- Run independent tasks in parallel. Give agents only the context needed for their task.
- Ask for concise findings with evidence, verification results, and anything they could not verify.
- When model selection is available, use cheaper/faster models for routine work: Luna for narrow lookups and mechanical tasks, Terra for bounded coding, tests, and reviews. Escalate difficult findings to the lead agent.
- State whether an agent may merge or deploy and which gate applies. One agent owns a shared deployment or staging environment at a time.
- The lead agent owns integration and final review. Model preferences here guide delegation; the main model is selected in the app or CLI.

## Subagent orchestration

Delegation does not transfer responsibility for completion. The lead owns every dispatched task through verification and handoff.

1. Track every agent: objective, model, owned files/environment, current state, dependencies, and expected output. Check live status before reporting how many agents are running.
2. Work on independent tasks while an agent runs. Do not duplicate its assigned work or mutate its owned files/environment. Reassign ownership explicitly before taking over.
3. Read every completed result promptly. Integrate its findings, verify its evidence, and report meaningful results or blockers to the user without waiting to be asked. Send a concise progress update at least every 60 seconds during active work.
4. Distinguish preparation, execution, and verification. Report an action as started only after its tool/process has started; report success only with checked evidence. A stopped agent waiting for a dependency is not running.
5. When a dependency clears, explicitly resume the waiting agent and verify it is active. A message to a stopped agent is not a restart. If delegation fails or hits a limit, take ownership of the remaining work or report the genuine blocker.
6. Do not finish the work turn while any required subagent is still running or its result remains uncollected. Use the wait mechanism, integrate every required terminal result, and verify the combined outcome before the completion reply. Answer interim user questions promptly without abandoning the active task.
7. An agent finishing is not the task finishing. Collect artifacts, perform integration/final review, and account for tests, delivery, cleanup, and any remaining work. Only abandon an agent after explicit failure or when its work is no longer needed; record why and who owns any remainder.
8. Reuse authority already granted for the same scope and budget. Ask again only for a materially new action, increased risk/cost, or a genuinely missing decision.
9. For long-running tests and paid resources, keep an explicit owner, budget/deadline, result location, and cleanup plan. Before a session handoff, verify what survives the session and transfer ownership; never imply live agents migrate with conversation history.

## i18n

Read [`docs/I18N.md`](./docs/I18N.md) before adding copy.

1. Import only `@/lib/i18n`. Never import `i18next` or `react-i18next` outside `client/src/lib/i18n/` and `electron/`.
2. Add both `en` and `pt-BR` JSON keys. Interpolation is `{name}`, not `{{name}}`.
3. Plurals: `_one` / `_other` / `_zero`. Pass a numeric `count`. Use `_zero` when 0 must not be Portuguese singular.
4. Desktop permission copy: pass `{ context: "desktop" }` at the call site. Do not inject it in the wrapper.
5. Electron menus live in `electron/locales/`. Do not put them in the client JSON.
6. Leave Worker/OG meta, slash command **names**, `error-boundary.tsx`, and legal route files alone unless the task is those files.

### PT-BR voice (QG, What's New, in-app)

Write the Portuguese **first**. Do not write English and translate. Translated PT-BR is what sounds fake.

It has to sound like a person in the QG. `você`. Short. What to click. The bar is `dados-discord-e-cargos.pt-BR.md`, not a press release.

Do:

- Read it out loud. If you would not send it in the QG, rewrite.
- Concrete: the button, the menu, the slash command.
- Gerund (`está fazendo`), not European `a fazer`. `você`, not `tu` / `a gente` as a system voice.

Loanwords: Brazilian gaming Portuguese keeps a lot of English. Do not "fix" that.

- Keep the word the QG already says: DM, chat, call, mute / mutar, kick / kickar, ban, staff, post, feed, login, app, APK, VIP, card, online, offline, Watch party, Chrome, Play Store, TestFlight. Full list in `docs/I18N.md`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rafaelcg/pqp](https://github.com/rafaelcg/pqp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
