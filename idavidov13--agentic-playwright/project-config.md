---
trigger: always_on
description: Routing layer between user intent and the specialized skills that own the rules. Load first on every non-trivial task.
---


# AI-Native Workflow

Routing layer between user intent and the specialized skills that own the rules. Load first on every non-trivial task.

## Critical

- **Low confidence means Phase 3 is incomplete.** If the would-be confidence is **< 5**, do **NOT** emit Phase 4. Return to Phase 3 and **ASK the user** for the missing primary inputs (URL, OpenAPI link, field list, area name). Phase 4 exists for honest _trade-off_ decisions, not for documenting "I don't have enough data" — that belongs in a question to the user. This rule is the most leveraged in the workflow: it is what stops the agent from emitting plausible-looking plans built on guesses.
- **Ask, don't invent.** Never guess folder names, file paths, env-var names, enum values, credentials, or message strings. `ls`, `grep`, `playwright-cli`, OpenAPI — or ask.
- **Refuse placeholders.** Guessed selectors, unverified message strings, made-up enum values, secret-shaped strings — refuse and re-explore. **`TODO` / `skeleton` / "to fill in later" outputs count as placeholders too** — offering a "skeleton page object with TODO locators while we wait for `playwright-cli`" is the same failure mode as inventing locators outright; don't.
- **Verify the user's premise even in Direct Mode.** Before applying a one-line fix, confirm the reported defect actually exists (the typo on the cited line, the import the user wants removed, the value the user says is currently set). If the premise doesn't match the file, switch out of Direct Mode and ASK — applying a "fix" to a defect that isn't there invents a change.
- **Specialized skills own the rules.** This skill never restates rules from `api-testing`, `page-objects`, etc. It tells you **which skill to load** and **in what order**.
- **The Constitution (the always-on instructions wrapping these skills) is the safety floor.** MUST/SHOULD/WON'T tables are hard stops; they take precedence over any prose, template, or example.
- **Audit-then-edit by default.** For any non-trivial change, follow the 8-phase workflow below. Phase 4 (Plan + Confidence) is mandatory before Phase 6 (Apply).
- **Confidence gate is required.** Every Plan output must include a 1-10 confidence + rationale + unknowns block. See "Phase 4" below.
- **Exploration is non-negotiable.** UI → `playwright-cli` only (no IDE browser MCP, no Cursor browser, no `playwright codegen`). API → OpenAPI/docs first, live HTTP only as fallback.
- **One skill at a time.** Load skills sequentially per the routing table. Don't stack 5 skills' Critical blocks before starting work.
- **After any test edit, run the affected tests.** On red, load `debugging` — never suppress, never bump timeouts.

## Main Workflow (8 Phases)

Every non-trivial task runs through these phases in order.

| #   | Phase                   | Generic substeps                                                                                                                                                                                         | Flow-specific substeps owned by                                                                                                                 |
| --- | ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | **Classify intent**     | Match user request → intent class (codegen / edit / refactor / debug / explore / config).                                                                                                                | —                                                                                                                                               |
| 2   | **Route**               | Pick first skill from routing table. Codegen → `common-tasks`. Other → direct skill.                                                                                                                     | —                                                                                                                                               |
| 3   | **Explore**             | Confirm what's known vs unknown; gather evidence per flow. **If a primary input is missing (URL, OpenAPI, area folder, field list), ASK the user before advancing — do NOT carry the gap into Phase 4.** | UI: `playwright-cli`. API: `api-testing` Phase 1 (OpenAPI). Refactor: `refactor-values` Phase 1 (impact grep). Debug: `debugging` capture step. |
| 4   | **Plan + Confidence**   | Produce proposal block (see format below).                                                                                                                                                               | — (owned here)                                                                                                                                  |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [idavidov13/agentic-playwright](https://github.com/idavidov13/agentic-playwright) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
